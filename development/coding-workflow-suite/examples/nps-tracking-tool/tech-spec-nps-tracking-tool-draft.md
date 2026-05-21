# Tech Spec: Internal NPS-Tracking Tool

**Stage:** Tech Spec (Draft)
**Project:** Internal NPS-Tracking Tool
**Date:** 2026-05-21
**Upstream artifacts:** prd-nps-tracking-tool.md

## Assumptions and inferred inputs

| Input | Source | If inferred or user-confirmed: notes |
|---|---|---|
| Product behavior (CSM at-risk list, VP trend tiles, intervention marking, 2-min refresh) | supplied (PRD) | — |
| Non-functional targets (score freshness, zero data loss across 24h Delighted outage) | supplied (PRD) | — |
| Out-of-scope items (no customer-facing surface, no write-back to Delighted) | supplied (PRD) | — |
| Existing stack: Snowflake as canonical store, Auth0 SSO, Datadog, K8s, GitHub Actions, sqitch | user-confirmed | Standing engineering-team stack; PRD does not enumerate but team operates these in production. |
| Language/framework versions: Python 3.12 services, React 19 + TypeScript 5.4 SPA, FastAPI for read API | user-confirmed | Engineering team standard; not specified in PRD. |
| Upstream source: Delighted webhook (HMAC-signed JSON, response-level granularity) | user-confirmed | Vendor of record for survey distribution; PRD names the survey program but not the integration shape. |
| Compliance regime: SOC 2 Type II, EU data residency for EU customers | supplied (PRD §Non-functional requirements) | — |
| Project shape: greenfield within existing platform | user-confirmed | No prior NPS surface exists; integrates with already-running CRM sync, Auth0, Snowflake. |

## Problem statement

Build an internal tool that turns Delighted NPS survey responses into a CSM-facing at-risk-account list and a VP-facing weekly trend view, with detractor responses summarized into a primary-driver string per account. The system ingests Delighted webhooks, scores accounts on a rolling basis against Snowflake-resident CRM data, and serves a read-only React SPA backed by a thin FastAPI read layer. This is greenfield work within the existing Snowflake/Auth0/K8s platform; no externally-facing surface and no write-back to Delighted are in scope.

## System context diagram

```
                  +-----------------------+
                  |   Delighted (SaaS)    |
                  |  survey distribution  |
                  +-----------+-----------+
                              |  webhook POST (HMAC-signed JSON)
                              v
+----------------+   +-----------------+   +----------------------+
|  CRM sync      |-->|   Snowflake     |<--|   Scoring service    |
|  (pre-existing)|   |  (canonical     |   |  (scheduled jobs +   |
+----------------+   |   store; Tasks; |   |   Cortex calls)      |
                     |   Cortex)       |   +----------------------+
                     +--------+--------+              ^
                              ^                       |
                              |                       |
                     +--------+--------+   +----------+-----------+
                     |   Read API      |<--|   Ingest worker      |
                     |   (FastAPI)     |   |  /webhook/delighted  |
                     +--------+--------+   +----------------------+
                              ^
                              |  HTTPS, JWT-bearing
                              |
                     +--------+--------+
                     |  Frontend SPA   |---> Auth0 (SSO, token issuance)
                     |  (React)        |
                     +--------+--------+
                              |
                              v
                     +-----------------+
                     |    Datadog      |  (logs, metrics, traces)
                     +-----------------+

Deployment substrate: company-standard Kubernetes platform.
Trust boundaries: Delighted->Ingest (HMAC); Frontend->Read API (Auth0 JWT);
                  Read API->Snowflake (service account, row-level security by region).
```

## Component breakdown

### Ingest worker

- **Owns:** the Delighted webhook contract; signature verification; idempotent writes into `survey_response`.
- **Responsibility:** validate HMAC, deduplicate by `response_id`, persist the row, emit metrics. Nothing else — no scoring, no summarization.
- **Dependencies (in):** Delighted webhook POSTs. **(out):** Snowflake (`survey_response` table), Datadog, Vault (HMAC secret).
- **Most-feared failure mode:** signature-verification regression that admits forged payloads, or duplicate-row admission that breaks downstream score idempotency. Engineering rationale for the split: webhook ingestion has its own availability profile (must absorb Delighted retries during back-end outages) and benefits from being deployable independently of the scoring path.

### Scoring service

- **Owns:** the score-freshness invariant; population of `nps_score`; detractor driver summarization (mechanism deferred to Q-003).
- **Responsibility:** compute per-account NPS scores from `survey_response`, produce a primary-driver summary string, write one row per account into `nps_score`. Scheduled inside Snowflake via Tasks; calls Cortex for summarization.
- **Dependencies (in):** Snowflake Tasks scheduler. **(out):** Snowflake (`survey_response` read, `nps_score` write), Snowflake Cortex (summarization), Datadog, Vault.
- **Most-feared failure mode:** silent staleness — the job runs but produces incorrect or stale `nps_score` rows without alerting. Engineering rationale for the split from ingest: scoring cadence (Q-001) is the architecturally hot question, and isolating it lets the resolution change without touching ingest.

### Frontend

- **Owns:** the CSM-facing UX (at-risk list, drill-down, intervention marking) and the VP-facing weekly trend tiles.
- **Responsibility:** read-only against the Read API; presents stale-data banners when freshness drops; authenticates via Auth0 (issuance pattern deferred to Q-002).
- **Dependencies (in):** end-user browsers (internal employees only). **(out):** Read API, Auth0.
- **Most-feared failure mode:** silently serving stale data without surfacing the staleness to the CSM. Engineering rationale for separating from the Read API: the SPA's release cadence and asset-delivery profile (CDN-fronted bundle) differ from the API's, and the two have different rollback semantics.

### Read API

- **Owns:** the REST surface the frontend consumes; intervention writes.
- **Responsibility:** translate REST requests into Snowflake view queries; write intervention marks; enforce per-request authorization against the requester's JWT.
- **Dependencies (in):** Frontend. **(out):** Snowflake (views + `intervention` writes), Auth0 (token validation, mechanism per Q-002), Datadog, Vault.
- **Most-feared failure mode:** authorization bypass that lets a CSM see another region's data in violation of row-level-security expectations. Engineering rationale for keeping it thin and stateless: deployable behind the existing load balancer with no new state to operate, and the auth model resolution (Q-002) is self-contained here.

## Data model

### `account`

| Field | Type | Notes |
|---|---|---|
| account_id | string PK | Sourced from CRM sync; already present in Snowflake. |
| name | string | Display name. |
| arr_usd | number | Used for at-risk prioritization. |
| region | string | Drives row-level security (EU vs. non-EU). |
| csm_email | string | Joins to intervention ownership. |
| created_at | timestamp | — |

### `survey_response`

| Field | Type | Notes |
|---|---|---|
| response_id | string PK | Delighted's identifier; idempotency key for ingest. |
| account_id | string FK -> account.account_id | Resolved at ingest time from Delighted's `account_email`. |
| nps_score | integer 0–10 | — |
| comment_raw | string nullable | PII; never logged. |
| comment_summary | string nullable | Populated by scoring service; mechanism per Q-003. |
| received_at | timestamp | From Delighted payload. |
| source | string | Constant `'delighted'` for now; reserves space for future channels. |

### `nps_score`

| Field | Type | Notes |
|---|---|---|
| account_id | string PK | One row per account. |
| latest_score | integer | Most recent NPS score. |
| latest_response_at | timestamp | Freshness reference. |
| primary_driver | string nullable | Detractor driver summary; mechanism per Q-003. |
| computed_at | timestamp | Drives the freshness SLI. |

### `intervention`

| Field | Type | Notes |
|---|---|---|
| intervention_id | string PK | UUID generated by Read API. |
| account_id | string FK -> account.account_id | — |
| csm_email | string | Pulled from the requester's JWT, not the request body. |
| note | string | CSM-supplied free text. |
| marked_at | timestamp | Server-assigned. |

Storage is Snowflake throughout. Referential integrity for FKs is enforced at the application boundary (Snowflake does not enforce FKs); the Read API validates account existence on intervention writes.

## Interfaces

### Webhook in (Delighted -> Ingest worker)

- **Method/path:** `POST /webhook/delighted`.
- **Payload:** Delighted's documented response schema (`response_id`, `account_email`, `score`, `comment`, `timestamp`).
- **Authentication:** HMAC signature header; secret in Vault.
- **Idempotency:** `response_id` is the dedup key; a repeated POST returns 200 without rewriting.
- **Failure semantics:** signature failure -> 401, no body persistence, alert. Snowflake-write failure -> 5xx so Delighted's retry kicks in.
- **Versioning:** Delighted's contract is external; breaking changes are handled by deploying a new path (`/webhook/delighted/v2`) rather than mutating the existing one.

### Read API (Frontend -> Read API)

- `GET /at-risk-accounts?since=&limit=` — returns the CSM at-risk list, sorted by composite risk score. JSON.
- `GET /accounts/{id}/responses?days=` — drill-down to recent responses for one account.
- `POST /accounts/{id}/interventions` — writes an `intervention` row; body is `{note}`; `csm_email` is taken from JWT.
- `GET /trends/weekly` — VP-facing trend tiles, week-bucketed.
- **Authentication:** Auth0-issued credential on every request; specific issuance pattern (session vs. JWT vs. opaque-token + introspection) deferred to Q-002.
- **Authorization:** every request is filtered by the requester's region claim against `account.region` row-level security.
- **Failure semantics:** Snowflake unavailability -> 503 with `Retry-After`; the frontend renders the stale-data banner.
- **Versioning:** URL-path versioning (`/v1/`) reserved for future use; current endpoints are unversioned-implicit-v1.

### Snowflake interaction model

- **Scoring service:** Snowflake Tasks for scheduling, Cortex calls for summarization, native SQL for reads/writes.
- **Read API:** standard SQL through the company's existing Snowflake connector against a set of views (not raw tables) so that schema evolution does not break the API.
- **Migrations:** `sqitch` per the company's existing workflow.

## Failure modes and reliability targets

- **Webhook delivery failure (Delighted outage or transient ingest failure).** Delighted retries with exponential backoff; ingest idempotency guards against duplicates. Target: zero data loss over a 24-hour Delighted outage window.
- **Snowflake unavailability.** Frontend renders a stale-data banner with the last-known `nps_score.computed_at`. Scoring service backs off with jitter; the Snowflake Task picks up where it left off on recovery. Target: no manual intervention required to recover for outages under 2 hours.
- **Score-freshness lag.** Measured as the gap between webhook arrival at ingest and the corresponding `nps_score.computed_at`. Target: P95 under 90 seconds (PRD's 2-minute refresh requirement minus a buffer for downstream display).
- **Cortex cost overrun.** Alerting on Snowflake Cortex hourly call count; a circuit-breaker pauses summarization calls when the count exceeds the budgeted threshold. Target: cost ceiling not exceeded in any rolling hour without paging.

## Security considerations

- **Webhook authenticity:** HMAC verification on every inbound POST; the secret is in Vault and rotated per company policy. Forged-payload risk is the primary inbound threat.
- **Data residency:** Snowflake row-level security enforced by `account.region`; EU customer responses are readable only from the EU-region Snowflake context. Auditable per SOC 2.
- **PII handling:** `account_email` and `comment_raw` are never written to application logs or shipped to external observability tools. Datadog receives only `response_id`, `account_id` hash, and `nps_score`. Redaction enforced at the logging SDK layer.
- **Request authorization:** Read API authorizes every request against the requester's Auth0 credential; the precise mechanism (session-store lookup, JWT verification, or opaque-token introspection) is deferred to Q-002.

## Observability

- **Ingest worker:** webhook arrival rate, ingest latency P50/P95, deduplication hit rate, signature-verification failures (alerting threshold: any sustained nonzero rate).
- **Scoring service:** scoring lag P50/P95 (SLI for the freshness target), Cortex call count and dollar cost, primary-driver summarization failure rate.
- **Read API:** P95 endpoint latency per route, 4xx/5xx rates, intervention-write success rate.
- **Frontend:** page-load timing via RUM, FE error tracking, stale-data-banner display rate (as a proxy for upstream-freshness SLI breaches visible to users).

All signals go to Datadog with structured JSON logs and standard tracing context; retention follows the company's default tier. Cardinality is bounded by route + region + status; per-account labels are not emitted.

## Cross-cutting concerns

- **Logging:** structured JSON to Datadog; PII redaction at the SDK layer (account email replaced with a deterministic hash; comment text never emitted).
- **Configuration:** environment variables for non-secret config; secrets in the company's Vault instance.
- **Deployment:** Docker images built by the company's existing GitHub Actions CI; deployed to the existing Kubernetes platform via the standard manifests.
- **Migrations:** Snowflake schema migrations via the company's existing `sqitch` workflow; every schema change ships with a forward and reverse change.

## Open architectural questions

### Q-001 — Real-time vs. batch scoring

**The question:** How should the scoring service satisfy the 2-minute refresh acceptance criterion (PRD FR-1.1) given SOC 2 auditability constraints and the existing Snowflake-centric stack?

**Candidate approaches:**
1. Streaming ingest with continuous scoring (each webhook triggers a scoring pass for the affected account).
2. Pure batch on Snowflake Tasks with an hourly refresh.
3. Micro-batch with a 60-second window via Snowflake Tasks.

**Forces in tension:**
- Latency requirement (PRD's 2-minute refresh) favors streaming or short micro-batch.
- Existing-stack alignment favors batch on Snowflake — the team already operates Tasks and has no streaming infra.
- SOC 2 auditability favors a single, auditable scoring path over a streaming path where each event takes its own route.
- Budget favors lower infra complexity; standing up a streaming substrate is the most expensive option.

**Resolution:** Resolve before Implementation Plan via an ADR (`adr.md`).

### Q-002 — Auth model for the frontend

**The question:** Which Auth0-backed credential pattern should the Read API and Frontend use for the CSM-facing surface?

**Candidate approaches:**
1. Session-based: Auth0 login, server-side session store consulted on every Read API request.
2. JWT issued by Auth0 with offline (signature-based) verification at the Read API.
3. Opaque token issued by Auth0 with introspection (`/userinfo` or equivalent) on every Read API request.

**Forces in tension:**
- Existing-stack consistency: Auth0 is the company's SSO, so all three are viable; the team does not currently operate a session store.
- Revocation latency: sessions and opaque-tokens revoke immediately; JWT revocation depends on lifetime-based eviction or a denylist.
- Operational complexity: sessions require new infrastructure; JWT minimizes per-request dependencies; opaque-token introspection adds an Auth0 round-trip per request.
- Compliance-adjacent posture (SOC 2): faster revocation is generally preferred for employee-internal tools handling PII.

**Resolution:** Resolve before Implementation Plan via an ADR (`adr.md`).

### Q-003 — Storage and computation of detractor driver summaries

**The question:** Where and when should the `primary_driver` summary be computed, and where should the inputs to that summary live?

**Candidate approaches:**
1. Raw text in Snowflake (`comment_raw`) with read-time summarization via Cortex, summary cached into `nps_score.primary_driver` on first read.
2. Precomputed summary at ingest time, written immediately to `survey_response.comment_summary` and rolled up by the scoring service.
3. Vector store (Pinecone or pgvector) for semantic search across responses, with summary computed against retrieved neighbors.

**Forces in tension:**
- SOC 2 constraint on new vendors effectively rules out external vector stores (Pinecone); pgvector would require a new datastore the team does not currently operate.
- Ingest-time cost: precomputing summaries on responses that no CSM ever reads is wasted Cortex spend.
- Read latency: read-time summarization adds a Cortex-call delay on first read for an account.
- Cortex cost predictability: ingest-time computation has predictable cost; read-time computation is bursty.

**Resolution:** Resolve before Implementation Plan via an ADR (`adr.md`).
