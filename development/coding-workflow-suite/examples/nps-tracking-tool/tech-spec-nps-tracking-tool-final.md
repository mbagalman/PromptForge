# Tech Spec: Internal NPS-Tracking Tool

**Stage:** Tech Spec (Final)
**Project:** Internal NPS-Tracking Tool
**Date:** 2026-05-21
**Upstream artifacts:** tech-spec-nps-tracking-tool-draft.md, adr-001-streaming-vs-batch-scoring.md, adr-002-auth-model.md, adr-003-driver-storage.md

## Assumptions and inferred inputs

| Input | Source | If inferred or user-confirmed: notes |
|---|---|---|
| Product behavior (CSM at-risk list, VP trend tiles, intervention marking, 2-min refresh) | supplied (Draft) | — |
| Non-functional targets (score freshness, zero data loss across 24h Delighted outage) | supplied (Draft) | — |
| Out-of-scope items (no customer-facing surface, no write-back to Delighted) | supplied (Draft) | — |
| Existing stack: Snowflake as canonical store, Auth0 SSO, Datadog, K8s, GitHub Actions, sqitch | supplied (Draft) | — |
| Language/framework versions: Python 3.12 services, React 19 + TypeScript 5.4 SPA, FastAPI for read API | supplied (Draft) | — |
| Upstream source: Delighted webhook (HMAC-signed JSON, response-level granularity) | supplied (Draft) | — |
| Compliance regime: SOC 2 Type II, EU data residency for EU customers | supplied (Draft) | — |
| Project shape: greenfield within existing platform | supplied (Draft) | — |
| Scoring cadence: micro-batch via Snowflake Task on a 60-second window | supplied (ADR-001) | Resolves Q-001; recomputes `nps_score` for any account with newer `survey_response.received_at`. |
| Auth model: Auth0-issued JWT, offline verification against cached JWKS, 1-hour token lifetime | supplied (ADR-002) | Resolves Q-002; refresh via Auth0's refresh-token flow. |
| Driver summarization: raw text in Snowflake, read-time Cortex summarization, cached into `survey_response.comment_summary` | supplied (ADR-003) | Resolves Q-003; circuit-breaker at ~1000 Cortex calls/hour soft cap. |

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
|  (pre-existing)|   |  (canonical     |   |  (Snowflake Task,    |
+----------------+   |   store; Tasks; |   |   60s window;        |
                     |   Cortex)       |   |   ADR-001)           |
                     +--------+--------+   +----------------------+
                              ^                       ^
                              |                       |
                     +--------+--------+   +----------+-----------+
                     |   Read API      |<--|   Ingest worker      |
                     |   (FastAPI;     |   |  /webhook/delighted  |
                     |   JWT verify    |   +----------------------+
                     |   per ADR-002;  |
                     |   Cortex on     |
                     |   first read    |
                     |   per ADR-003)  |
                     +--------+--------+
                              ^
                              |  HTTPS, Auth0-issued JWT (ADR-002)
                              |
                     +--------+--------+
                     |  Frontend SPA   |---> Auth0 (SSO, JWT issuance, refresh)
                     |  (React)        |
                     +--------+--------+
                              |
                              v
                     +-----------------+
                     |    Datadog      |  (logs, metrics, traces)
                     +-----------------+

Deployment substrate: company-standard Kubernetes platform.
Trust boundaries: Delighted->Ingest (HMAC); Frontend->Read API (Auth0 JWT, offline verify);
                  Read API->Snowflake (service account, row-level security by region);
                  Read API->Cortex (read-time summarization, ADR-003).
```

## Component breakdown

### Ingest worker

- **Owns:** the Delighted webhook contract; signature verification; idempotent writes into `survey_response`.
- **Responsibility:** validate HMAC, deduplicate by `response_id`, persist the row, emit metrics. Nothing else — no scoring, no summarization.
- **Dependencies (in):** Delighted webhook POSTs. **(out):** Snowflake (`survey_response` table), Datadog, Vault (HMAC secret).
- **Most-feared failure mode:** signature-verification regression that admits forged payloads, or duplicate-row admission that breaks downstream score idempotency. Engineering rationale for the split: webhook ingestion has its own availability profile (must absorb Delighted retries during back-end outages) and benefits from being deployable independently of the scoring path.

### Scoring service

- **Owns:** the score-freshness invariant; population of `nps_score`.
- **Responsibility (per ADR-001):** runs as a **Snowflake Task with a 60-second window**, recomputing `nps_score` for any account with a newer `survey_response.received_at` than its existing `nps_score.computed_at`. Native SQL only; no per-event triggering.
- **Driver summarization (per ADR-003):** the scoring service **does not precompute summaries**. The `primary_driver` field on `nps_score` is populated from the most recent `survey_response.comment_summary` for the account, which the Read API fills in on first read.
- **Dependencies (in):** Snowflake Tasks scheduler. **(out):** Snowflake (`survey_response` read, `nps_score` write), Datadog.
- **Most-feared failure mode:** silent staleness — the Task fails or skews behind without alerting. Engineering rationale for the split from ingest: the 60-second scoring cadence is operationally distinct from webhook absorption and is operated entirely inside Snowflake, isolating both deployment and failure surface.

### Frontend

- **Owns:** the CSM-facing UX (at-risk list, drill-down, intervention marking) and the VP-facing weekly trend tiles.
- **Responsibility (per ADR-002):** read-only against the Read API; **authenticates via JWT issued by the existing Auth0 tenant**, attached as a bearer token on every Read API request. Token lifetime is **1 hour**; the SPA refreshes via Auth0's refresh-token flow before expiry. Renders stale-data banners when freshness drops.
- **Dependencies (in):** end-user browsers (internal employees only). **(out):** Read API, Auth0 (login and refresh).
- **Most-feared failure mode:** silently serving stale data without surfacing the staleness to the CSM. Engineering rationale for separating from the Read API: the SPA's release cadence and asset-delivery profile (CDN-fronted bundle) differ from the API's, and the two have different rollback semantics.

### Read API

- **Owns:** the REST surface the frontend consumes; intervention writes; **read-time summarization of detractor comments (per ADR-003)**.
- **Responsibility:** translate REST requests into Snowflake view queries; write intervention marks; enforce per-request authorization against the requester's JWT.
- **JWT verification (per ADR-002):** **verifies inbound JWTs offline against the Auth0 JWKS endpoint**, cached locally for 1 hour. No per-request Auth0 round-trip. Region claim from the JWT is bound into every Snowflake query for row-level-security alignment.
- **Read-time summarization (per ADR-003):** on the first read of a `survey_response` whose `comment_summary` is null and `comment_raw` is non-null, the Read API **calls Snowflake Cortex's summarization function and writes the result back to `survey_response.comment_summary`** as a cache. Subsequent reads serve the cached summary directly. A **circuit-breaker pauses summarization calls when hourly Cortex call count exceeds the budgeted threshold (~1000 calls/hour soft cap)**; reads under breaker-open conditions return the raw comment with a `summary_unavailable` flag.
- **Dependencies (in):** Frontend. **(out):** Snowflake (views + `intervention` writes + `survey_response.comment_summary` writes), Snowflake Cortex (summarization on first read), Auth0 JWKS endpoint (cached), Datadog, Vault.
- **Most-feared failure mode:** authorization bypass (region claim mishandled, JWT verification skipped, JWKS cache poisoned) that lets a CSM see another region's data in violation of row-level-security expectations. Engineering rationale for keeping it thin and stateless: the JWT-offline-verify model (ADR-002) means no session state to operate, and read-time Cortex summarization (ADR-003) is naturally co-located with the read path that needs it.

## Data model

Storage is Snowflake throughout. Referential integrity for FKs is enforced at the application boundary (Snowflake does not enforce FKs); the Read API validates account existence on intervention writes.

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
| comment_summary | string nullable | **Populated read-time by Read API via Cortex (per ADR-003)**; null until first read, then cached. |
| received_at | timestamp | From Delighted payload. |
| source | string | Constant `'delighted'` for now; reserves space for future channels. |

### `nps_score`

| Field | Type | Notes |
|---|---|---|
| account_id | string PK | One row per account. |
| latest_score | integer | Most recent NPS score. |
| latest_response_at | timestamp | Freshness reference. |
| primary_driver | string nullable | Reflects the latest `survey_response.comment_summary`; null when no detractor summary has been cached yet (per ADR-003). |
| computed_at | timestamp | **Written by Snowflake Task on a 60-second window (per ADR-001)**; drives the freshness SLI. |

### `intervention`

| Field | Type | Notes |
|---|---|---|
| intervention_id | string PK | UUID generated by Read API. |
| account_id | string FK -> account.account_id | — |
| csm_email | string | **Pulled from the requester's JWT claims (per ADR-002), not the request body.** |
| note | string | CSM-supplied free text. |
| marked_at | timestamp | Server-assigned. |

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
- `GET /accounts/{id}/responses?days=` — drill-down to recent responses for one account. **First read of a response with a null `comment_summary` triggers a Cortex summarization call and caches the result (per ADR-003)**; subsequent reads are cache hits.
- `POST /accounts/{id}/interventions` — writes an `intervention` row; body is `{note}`; `csm_email` is taken from the JWT claims (per ADR-002).
- `GET /trends/weekly` — VP-facing trend tiles, week-bucketed.
- **Authentication (per ADR-002):** **Auth0-issued JWT** on every request, attached as `Authorization: Bearer <token>`. Read API **verifies the JWT offline against the Auth0 JWKS** (cached 1 hour). 1-hour token lifetime; refresh handled by the SPA.
- **Authorization:** every request is filtered by the JWT's region claim against `account.region` row-level security.
- **Failure semantics:** Snowflake unavailability -> 503 with `Retry-After`; the frontend renders the stale-data banner. JWT verification failure -> 401. Cortex circuit-breaker open -> 200 with raw comment and `summary_unavailable: true`.
- **Versioning:** URL-path versioning (`/v1/`) reserved for future use; current endpoints are unversioned-implicit-v1.

### Snowflake interaction model

- **Scoring service (per ADR-001):** **Snowflake Tasks on a 60-second schedule**, native SQL only. No per-event triggering and no streaming substrate.
- **Read API (per ADR-003):** standard SQL through the existing Snowflake connector against a set of views; **Snowflake Cortex summarization invoked from the read path on first read of a detractor comment**, with the cached summary written back to `survey_response.comment_summary`.
- **Migrations:** `sqitch` per the company's existing workflow.

## Failure modes and reliability targets

- **Webhook delivery failure (Delighted outage or transient ingest failure).** Delighted retries with exponential backoff; ingest idempotency by `response_id` guards against duplicates. Target: zero data loss over a 24-hour Delighted outage window.
- **Snowflake unavailability.** Frontend renders a stale-data banner with the last-known `nps_score.computed_at`. Scoring service backs off with jitter; the Snowflake Task resumes from where it left off on recovery. Target: no manual intervention required to recover for outages under 2 hours.
- **Score-freshness lag.** Measured as the gap between webhook arrival at ingest and the corresponding `nps_score.computed_at`. **Target (per ADR-001): P95 under 90 seconds** (the PRD's 2-minute refresh requirement minus a 30-second buffer for downstream display).
- **Cortex cost overrun.** Alerting on Snowflake Cortex hourly call count; **circuit-breaker (per ADR-003) pauses summarization calls when the hourly count exceeds the budgeted threshold (~1000 calls/hour soft cap)**. Target: cost ceiling not exceeded in any rolling hour without paging.
- **JWT compromise / revocation lag (per ADR-002).** Revocation latency is bounded by the **1-hour token lifetime**. A CSM whose access is revoked may continue to view the at-risk list until their current JWT expires (maximum 1 hour). Mitigation: short token lifetime; revocation enforced on the next refresh. Target: revocation reflected at the Read API within 1 hour of the Auth0-side revocation event; documented and accepted as a known limitation.

## Security considerations

- **Webhook authenticity:** HMAC verification on every inbound POST; the secret is in Vault and rotated per company policy. Forged-payload risk is the primary inbound threat.
- **Data residency:** Snowflake row-level security enforced by `account.region`; EU customer responses are readable only from the EU-region Snowflake context. Auditable per SOC 2.
- **PII handling:** `account_email`, `comment_raw`, and `comment_summary` are never written to application logs or shipped to external observability tools. Datadog receives only `response_id`, `account_id` hash, and `nps_score`. Redaction is enforced at the logging SDK layer.
- **Request authorization (per ADR-002):** **Read API verifies every request's JWT offline against the cached Auth0 JWKS**; the region claim is bound into the downstream Snowflake query. **Revocation latency is up to 1 hour** (bounded by token lifetime) and is the named compliance trade-off accepted in ADR-002.
- **Read-time Cortex summarization (per ADR-003):** `comment_raw` is sent to Snowflake Cortex within the Snowflake trust boundary only; no external LLM vendor receives PII. The cached `comment_summary` inherits the same row-level-security regime as `comment_raw`.

## Observability

- **Ingest worker:** webhook arrival rate; ingest latency P50/P95; deduplication hit rate; signature-verification failures (alerting threshold: any sustained nonzero rate).
- **Scoring service (per ADR-001 and ADR-003):** **scoring lag P50/P95 (SLI for the freshness target, P95 < 90s)**; **Snowflake Task success rate**; **Cortex call count and dollar cost (alerting threshold tied to circuit-breaker soft cap)**; **primary-driver summarization failure rate**.
- **Read API:** P95 endpoint latency per route; 4xx/5xx rates; intervention-write success rate; **JWT verification failure rate (signal for JWKS-cache or Auth0-issuer drift)**; **read-time summarization latency P50/P95 (target: P95 first-read < 3s for the FR-2.1 drill-down responsiveness budget, per ADR-003)**; **circuit-breaker open-state duration**.
- **Frontend:** page-load timing via RUM; FE error tracking; stale-data-banner display rate (as a proxy for upstream-freshness SLI breaches visible to users); token-refresh failure rate.

All signals go to Datadog with structured JSON logs and standard tracing context; retention follows the company's default tier. Cardinality is bounded by route + region + status; per-account labels are not emitted.

## Cross-cutting concerns

- **Logging:** structured JSON to Datadog; PII redaction at the SDK layer (account email replaced with a deterministic hash; comment text and summary text never emitted).
- **Configuration:** environment variables for non-secret config; secrets in the company's Vault instance.
- **Deployment:** Docker images built by the company's existing GitHub Actions CI; deployed to the existing Kubernetes platform via the standard manifests.
- **Migrations:** Snowflake schema migrations via the company's existing `sqitch` workflow; every schema change ships with a forward and reverse change.

## Open architectural questions

- **Q-001 — Real-time vs. batch scoring.** Resolved (see ADR-001). Chosen approach: micro-batch with a 60-second Snowflake Task window, recomputing `nps_score` for any account with newer `survey_response.received_at`.
- **Q-002 — Auth model for the frontend.** Resolved (see ADR-002). Chosen approach: JWT issued by Auth0 with offline verification at the Read API against cached JWKS; 1-hour token lifetime; refresh via Auth0's refresh-token flow.
- **Q-003 — Storage and computation of detractor driver summaries.** Resolved (see ADR-003). Chosen approach: raw text in Snowflake plus read-time summarization via Snowflake Cortex, cached into `survey_response.comment_summary` on first read; circuit-breaker at ~1000 Cortex calls/hour soft cap.
