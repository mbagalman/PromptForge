# Implementation Plan: Internal NPS-Tracking Tool

**Stage:** Implementation Plan
**Project:** Internal NPS-Tracking Tool
**Date:** 2026-05-21
**Upstream artifacts:** tech-spec-nps-tracking-tool-final.md, adr-001-streaming-vs-batch-scoring.md, adr-002-auth-model.md, adr-003-driver-storage.md

## Assumptions and inferred inputs

Inputs to this plan are tagged `supplied` (from the named upstream artifact), `user-confirmed` (inline content the user provided in the working session), or `inferred` (a placeholder reasoned from the upstream artifacts that should be re-verified before execution begins).

| Input | Source | Notes |
|---|---|---|
| Tech Spec (Final) | supplied (`tech-spec-nps-tracking-tool-final.md`) | Authoritative for component breakdown, data model, observability targets, and non-functional requirements (FR-1.1 / FR-1.2 / FR-2.1 / FR-2.2 / FR-3.1). |
| ADR-001 (streaming vs. batch scoring) | supplied | Constrains Phase 2 to a 60-second Snowflake Task cadence; sources the R-1 risk. |
| ADR-002 (auth model) | supplied | Constrains Phase 3 to JWT-via-Auth0 with a 1-hour token lifetime; sources the R-2 risk. |
| ADR-003 (driver storage) | supplied | Constrains Phase 2 to Cortex summarization with the read-time-cache pattern; sets the $0-incremental-SaaS budget; sources the R-3 risk. |
| Team capacity | user-confirmed | 1 FTE-quarter (~60 engineering-days available; 50 days conservative target, 10 days reserve). |
| Definition of done (project-level) | user-confirmed | v1 launched to ~12 CS Managers with feature flag ON in production; all four phases' gates met; on-call runbook produced; 5 business days of clean production observation. |
| User population | supplied (BRD) | ~12 CS Managers plus the VP of CS. Internal-only tool, no external exposure. |
| Named approvers and owners | supplied (BRD + ADRs) | Priya Patel as Data team lead and security reviewer; Sarah Chen as VP of CS sign-off; engineering lead role assumed staffed for the project quarter. |
| CI/CD substrate | inferred | Plan assumes the existing GitHub Actions pipeline plus the `sqitch` migration tooling already used by adjacent services; if either is unavailable, Phase 1 effort widens by ~2 engineering-days. |

## Plan overview

- **Scope:** Build and launch the Internal NPS-Tracking Tool end-to-end across four sequential phases: ingest pipeline, scoring service, frontend MVP with Auth0 integration, and a coordinated rollout to all CS Managers. The plan covers code, schema migrations, observability, runbook, and the rollout walkthrough — but does not re-litigate decisions resolved in the named ADRs.
- **Target completion shape:** Sequential phases (Phase 1 → Phase 2 → Phase 3 → Phase 4); no canary because the user population is ~12 internal users. Phase 4 is a coordinated walkthrough with the feature flag flipped ON once gates clear, followed by a 5-business-day observation window that closes the project-level definition of done.
- **Exclusions:** Token-introspection-based JWT revocation (deferred per ADR-002 residual risk R-2); ingest-time precomputation of `comment_summary` (only triggered if R-3 materializes per ADR-003); BRD non-goals (external customer-facing surface, mobile app, multi-tenant rollout to non-CSM roles). The plan also explicitly does not cover the Cortex-cadence-tightening work flagged in R-1 — that is a follow-on initiative if and when the per-account update frequency requirement changes.

## Work breakdown

Phases are ordered. Each phase has Goal, Deliverable, Approval-gate criteria, Estimated effort, and Dependencies. Approval-gate criteria are deterministic — a reviewer can evaluate them without taste or judgment. Estimates carry confidence bands. Work-items inside each phase are listed for orientation only; they are not separately gated and may be reshuffled by the implementing engineer as long as the phase-level approval criteria are still met at the gate.

### Phase 1 — Ingest pipeline + Snowflake schema

- **Goal:** Detractor responses land in Snowflake within 60 seconds of arriving at the Delighted webhook.
- **Deliverable:** Python ingest worker deployed to staging; Snowflake schema migrated (`account`, `survey_response` tables); webhook signature verification working; all unit and integration tests pass on the phase branch.
- **Work-items (indicative, not separately gated):**
  - Provision the `sqitch` migration for `account` and `survey_response` against the dev Snowflake instance, then promote to staging.
  - Implement the Python ingest worker with HMAC signature verification against the Delighted webhook secret stored in Vault.
  - Implement idempotent insert semantics keyed on the Delighted response ID to absorb retry traffic.
  - Wire a staging deployment via the existing CI pipeline; emit ingest-lag and ingest-error counters into the standard observability stack.
  - Add unit tests for signature verification (valid, expired, tampered) and integration tests for end-to-end payload landing.
  - Document the schema and the webhook contract in the project's data catalog entry.
  - Add the ingest-lag dashboard tile to the team's existing service-health dashboard so it surfaces in routine review.
  - Verify the Vault read path for the webhook secret works from the staging compute environment; capture the read latency.
  - Add a smoke test that exercises a full Delighted-to-Snowflake round-trip against the staging deployment.
- **Approval-gate criteria:**
  - (a) All unit and integration tests pass on PR; required reviewers include the Data team lead.
  - (b) Security review for the survey-data access path recorded by Priya Patel (named approver) — recorded means a written approval in the project's standard review tool, not a verbal sign-off.
  - (c) Snowflake schema review recorded by a member of the Data team.
- **Estimated effort:** 5–8 engineering-days (P50 = 6; bands reflect uncertainty in Delighted webhook idempotency edge cases — specifically whether retried-with-different-payload behavior requires defensive merging logic beyond the response-ID key).
- **Dependencies:** Existing Delighted webhook access (already provisioned); Snowflake schema migration tooling (existing `sqitch` workflow); Vault access for webhook secret.

### Phase 2 — Scoring service + 60-second batch window

- **Goal:** Per-account NPS score and `primary_driver` field populate within 60 seconds of a new detractor response landing.
- **Deliverable:** Snowflake Task running every 60 seconds (per ADR-001); summarization wired to Cortex (per ADR-003) with the read-time-cache pattern in the `comment_summary` field; observability dashboards for scoring lag and Cortex cost.
- **Work-items (indicative, not separately gated):**
  - Author the Snowflake Task SQL for the 60-second batch window, including the rolling-window NPS calculation and the `primary_driver` enumeration logic.
  - Wire the Cortex summarization call behind the read-time-cache pattern: on read, return cached `comment_summary` if present; on miss, call Cortex, persist, return.
  - Implement the Cortex cost circuit-breaker with the ADR-003 default threshold (~1000 calls/hour); fail reads to a static degraded-mode payload when tripped.
  - Stand up observability dashboards: scoring lag P95, Cortex spend per hour, circuit-breaker state.
  - Run the 48-hour synthetic detractor traffic generator against staging to populate the gate (a) and gate (b) measurements.
  - Document the Cortex-cost projection methodology so the gate (b) measurement is reproducible by a Data team reviewer.
  - Add unit tests for the rolling-window NPS calculation and the `primary_driver` enumeration logic; verify against fixture data covering all five driver categories.
  - Add the scoring-lag-P95 panel as a permanent dashboard tile, not just a Phase-2 measurement artifact.
- **Approval-gate criteria:**
  - (a) P95 score-write-to-read latency under 90 seconds on staging measured over 48 hours of synthetic detractor traffic.
  - (b) Snowflake Cortex usage cost measured over the same 48-hour window; total projected monthly cost within the $0-incremental-SaaS budget per ADR-003.
  - (c) Cortex circuit-breaker tested end-to-end (triggered manually; verified that the alarm fires and reads degrade gracefully).
- **Estimated effort:** 4–7 engineering-days (P50 = 5; band widens if the Cortex per-call latency distribution forces a redesign of the cache-fill path).
- **Dependencies:** Phase 1 complete (Snowflake schema and `survey_response` data flowing).

### Phase 3 — Frontend MVP + Auth0 integration

- **Goal:** CS Managers can view the at-risk list and drill into accounts; the VP of CS can view the weekly trend tiles.
- **Deliverable:** React 19 SPA deployed to staging; JWT-via-Auth0 (per ADR-002) integrated end-to-end; FR-1.1 / FR-1.2 / FR-2.1 / FR-2.2 / FR-3.1 satisfied; observability for page-load timing and FE error tracking active.
- **Work-items (indicative, not separately gated):**
  - Build the Read API in FastAPI fronting Snowflake, with JWT verification middleware against the Auth0 tenant.
  - Build the React 19 SPA: at-risk list view (FR-1.1, FR-1.2), drill-down view (FR-2.1, FR-2.2), trend-tile view (FR-3.1).
  - Wire the intervention-marking write path through the Read API to a `intervention` table; ensure concurrent CSM reads observe consistent state.
  - Integrate axe-core into the frontend CI step; baseline the rule set against WCAG 2.1 AA.
  - Seed the staging Snowflake instance with 5,000 representative accounts to drive gate (c) and gate (d) measurements.
  - Wire Sentry (or equivalent) for frontend error tracking; configure the dashboard for page-load timing and unhandled-rejection rate.
  - Author the Playwright end-to-end suite for the three E2E scenarios listed under the test strategy.
  - Run the manual screen-reader pass against the at-risk list and the drill-down view before exiting Phase 3; capture the result as evidence for the Phase 4 sign-off.
  - Confirm the FR-3.1 trend tile renders against the seeded staging data and matches the VP-of-CS visualization expectations agreed during the BRD review.
- **Approval-gate criteria:**
  - (a) All WCAG 2.1 AA checks pass via axe-core in CI (zero violations of WCAG 2.1 AA rules; warnings allowed but reviewed).
  - (b) The auth flow integration test suite passes against the staging Auth0 tenant (test cases cover happy-path login, refresh-token flow, and expired-token rejection).
  - (c) P95 page load < 2 seconds for the at-risk list on staging with seeded data for 5,000 accounts (FR-1.1 acceptance criterion).
  - (d) P95 drill-down read for accounts with ≤50 responses < 3 seconds on staging, including cold-read Cortex summarization (FR-2.1 acceptance criterion).
- **Estimated effort:** 8–12 engineering-days (P50 = 10; the spread reflects whether the cold-read Cortex path comfortably meets gate (d) or forces a precomputation workaround mid-phase).
- **Dependencies:** Phase 2 complete (scoring service producing `nps_score` rows for staging accounts).

### Phase 4 — Rollout

- **Goal:** Production launch to all ~12 CS Managers; first 5 business days of production usage observed.
- **Deliverable:** Feature flag toggled ON in production; on-call runbook produced and reviewed; rollback procedure documented and tested in staging; observability checkpoints active in production.
- **Work-items (indicative, not separately gated):**
  - Run the 5-business-day staging soak with the feature flag ON; capture the error-budget baseline measurement that gate (a) compares against.
  - Schedule and run the VP-of-CS staging walkthrough; capture the recorded sign-off in the standard review tool.
  - Draft the on-call runbook (paging policy, common failure modes, escalation contacts, dashboard links); route to the on-call rotation lead for review.
  - Execute the rollback drill in staging: flip the feature flag OFF, confirm the user-facing surface disappears, confirm data state is consistent, capture the run log as the gate (d) artifact.
  - Schedule the production-flag-on window with all ~12 CSMs notified; begin the 5-business-day production observation window.
  - Hand off the weekly usage-telemetry review to the VP of CS per the R-4 mitigation cadence.
  - File the project closeout note: definition-of-done checklist completed, residual risks tracked, follow-on items routed.
  - Capture the post-launch metrics baseline so future enhancements have a reference for comparison.
  - Archive the gate-evidence artifacts (signed approvals, latency reports, accessibility scans, runbook revisions) in the project's standard review tool.
- **Approval-gate criteria:**
  - (a) Feature flag enabled in staging for 5 business days with no observability regressions (zero new error-budget burn against the baseline).
  - (b) VP of CS (Sarah Chen) sign-off recorded — written confirmation that the staging walkthrough met functional expectations.
  - (c) On-call runbook produced; reviewed and signed off by the on-call rotation lead.
  - (d) Rollback procedure (feature flag OFF + manual data-state reconciliation) documented and tested end-to-end in staging.
- **Estimated effort:** 3–5 engineering-days (P50 = 4; the band tightens once Phase 3 closes because the remaining work is mostly process, not engineering).
- **Dependencies:** Phase 3 complete; on-call rotation member available for the rollout window.

**Cumulative effort:** P50 sum = 25 engineering-days; band = 20–32 engineering-days. Sits comfortably inside the 50-day conservative target with 10 days reserve, leaving headroom for the residual risks below.

**Dependency graph:** Phase 1 → Phase 2 → Phase 3 → Phase 4 is strictly sequential. There is no parallelizable work between phases because each phase's gate produces the data state or runtime surface that the next phase consumes. The exception is the on-call runbook drafting in Phase 4, which can begin in parallel with the Phase 3 final week if the engineering lead has spare capacity — but the runbook cannot be signed off until the staging soak in gate (a) completes.

**Approval-gate summary (deterministic checks across phases):**

| Phase | Gates count | Key recorded approvers |
|---|---|---|
| Phase 1 | 3 | Data team lead (Priya Patel) for security + schema |
| Phase 2 | 3 | Engineering lead; Data team lead for cost projection |
| Phase 3 | 4 | Engineering lead; CI for accessibility and performance |
| Phase 4 | 4 | VP of CS (Sarah Chen); on-call rotation lead |

## Agent execution boundaries

This table defines what an autonomous coding agent is and is not allowed to do on this project. `agents-md-generator.md` consumes it verbatim to set permission boundaries in the generated AGENTS.md / CLAUDE.md / SKILL.md file. Entries name specific verbs against specific surfaces — not categories. The three action classes are: `allowed without approval` (the agent acts autonomously and reports after), `requires approval` (the agent pauses for human sign-off before acting), and `prohibited` (the agent refuses and escalates).

| Action class | Examples | Rationale |
|---|---|---|
| `allowed without approval` | Edit files under `src/lib/`, `src/components/`, `src/api/`, `tests/`, `fixtures/`. Run the test suite via `pytest` (Python) or `npm test` (frontend). Run linters and formatters (`ruff`, `black`, `npm run lint`, `npm run format`). Update test fixtures and snapshots. Read from staging or development Snowflake instances. Open draft pull requests for review. | These actions affect only code, tests, and fixtures in non-protected paths; they are recoverable via standard PR review. Existing CI verifies correctness before merge. |
| `requires approval` | Schema migrations against any environment (`sqitch deploy` to dev / staging / prod). Dependency upgrades that change major version (e.g., FastAPI 0.x → 1.x, React 19 → 20). Calls to Snowflake Cortex above ~1000 calls/hour (cost guardrail per ADR-003). Touching any file under `infra/` or `.github/workflows/`. Modifying the `nps-tracking-tool.enabled` feature flag configuration in any environment. Changes to the Auth0 application configuration in any environment. | These actions touch data state, system configuration, or cost surfaces that benefit from explicit human sign-off. The approval gate is a recorded approval from a named role (Data team lead for Snowflake; engineering lead for major upgrades; VP of CS or her delegate for feature flag changes in production). |
| `prohibited` | Force-pushing to `main` or any release branch. Modifying production Snowflake data via direct SQL. Modifying Auth0 application configuration in production. Writing survey-response PII (account email, `comment_raw`, `comment_summary`) to application logs or to any external service (observability, ticketing, error trackers). Generating or rotating credentials of any kind. Disabling the Cortex cost circuit-breaker without explicit Data-team-lead approval. | These actions are either destructive, security-sensitive, or compliance-sensitive (SOC 2, GDPR). They should never be performed by an autonomous agent; if a situation seems to require one, the agent should escalate to a human with the relevant decision authority. |

If any row in this table is empty or stated vaguely, the plan is incomplete — `agents-md-generator.md` will not produce a useful agent file from it. The rows above name specific verbs against specific surfaces precisely so the downstream generator can compile them into runtime permission boundaries without re-interpretation.

## Risk register

Risks are drawn from the ADRs' residual-risk sections and from the BRD's adoption hypothesis. Each carries a named owner and a concrete mitigation, not a generic "monitor." Owners are responsible for raising the risk's status during the phase review where the risk is most likely to materialize — R-1 and R-3 in Phase 2 retrospectives, R-2 during the Phase 4 launch-window review, R-4 during the weekly post-launch usage-telemetry reviews.

| Risk | Source | Mitigation | Owner |
|---|---|---|---|
| R-1: Snowflake Tasks cost growth if hourly cadence is later tightened. | ADR-001 | Cost-monitoring dashboard with alarm at 1.5× baseline; cadence change requires recorded Data-team-lead approval. | Data team lead (Priya Patel) |
| R-2: JWT revocation latency up to 1 hour. | ADR-002 | Documented policy that off-boarded CSMs lose access within 1 hour (acceptable per HR policy); future enhancement (token introspection) tracked separately. | VP of CS for policy; engineering lead for technical mitigation |
| R-3: Cortex first-read latency may exceed FR-2.1's 3-second drill-down acceptance criterion in worst case (large response with complex comment text). | ADR-003 | Latency telemetry per Phase 2 gate (c); if violations exceed 1% in production, switch to ingest-time precomputation. | Engineering lead |
| R-4: CS Managers may not adopt the tool if it adds friction over their existing email-triage habit. | BRD | Live walkthrough with each CSM in the first week post-launch; usage telemetry reviewed weekly for 4 weeks. | VP of CS |

## Test strategy

Each test category below is owned by the team member implementing the corresponding phase work-item. Failing checks at any level block merge into the phase branch; the phase cannot advance past its approval gate until the relevant suites are green.

- **Unit tests**
  - Required for all new code.
  - Coverage target ≥80% line coverage on the ingest worker, scoring service, and Read API.
  - Frontend unit-test coverage target ≥70%.
  - Tooling: pytest for Python; vitest for TypeScript.
  - Unit tests run on every push to a phase branch; coverage reports are surfaced in PR review and failing-coverage PRs are blocked from merge.
- **Integration tests** — required for:
  - Webhook payload → Snowflake (`survey_response` table).
  - Scoring task → `nps_score` table.
  - Read API → frontend (mocked Auth0).
  - Intervention-write round-trip.
  - Cortex circuit-breaker.
  - Idempotent-replay behavior on duplicate Delighted response IDs.
  - JWT verification failure paths (expired, malformed, wrong issuer).
  - Cortex cost-circuit-breaker trip and reset cycle, including the degraded-mode response payload.
  - Tooling: pytest for the Python integration tests; vitest with mocked services for the frontend integration tests.
- **End-to-end tests** — required for:
  - Full Delighted webhook → CSM-visible at-risk list update within 2 minutes.
  - Drill-down round-trip with Cortex summarization.
  - Intervention marking with concurrent CSM read.
  - Auth0 happy-path login from a clean browser session through to the at-risk list view.
  - Tooling: Playwright.
- **Performance / load tests** — required at Phase 2 gate (a) and Phase 3 gate (c). Tooling: k6.
  - Phase 2 load profile: synthetic detractor traffic at 10× peak observed Delighted volume for 48 hours.
  - Phase 3 load profile: simulated CSM read traffic against the seeded 5,000-account staging dataset.
- **Accessibility tests** — axe-core in CI per Phase 3 gate (a); manual screen-reader pass before Phase 4 sign-off.
  - Screen-reader pass covers the at-risk list, the drill-down view, and the trend tile.
- **Tooling summary:** pytest for Python; vitest for TypeScript; Playwright for E2E; axe-core for accessibility; k6 for load. All suites run in CI and gate phase advancement at the PR level; failed checks block merge into the phase branch.

A failing performance or accessibility check is treated identically to a failing unit test: the PR cannot merge until the failure is resolved or an explicit exception is recorded by the relevant named approver (engineering lead for performance, accessibility champion for axe-core). No category may be "skipped for this PR" silently.

## Rollout plan

The rollout is shaped around the user population (~12 CSMs, internal only) and the project-level definition of done (5 business days of clean production observation). Because the population is small and known by name, the team prefers a coordinated walkthrough plus a single flag-on event over an incremental ramp; the savings in process overhead are higher-value than the diminished granularity of a percentage canary.

- **Feature flag:** `nps-tracking-tool.enabled`. ON in staging from Phase 2 onward; OFF in production until Phase 4 gate (a) clears. Toggle authority for production sits with the engineering lead and is logged per the `requires approval` row of the *Agent execution boundaries* table.
- **Canary:** not applicable; the user population is ~12 CSMs and the launch is internal-only. Phase 4 is a coordinated walkthrough rather than a canary rollout. The walkthrough doubles as the R-4 mitigation: each CSM receives a guided session, and usage telemetry is reviewed weekly for 4 weeks against the BRD adoption hypothesis.
- **Observability checkpoints (production):** scoring lag P95 per ADR-001; Cortex cost per ADR-003; JWT verification failure rate per ADR-002; FR-1.1 / FR-2.1 latency dashboards; error budget burn per service. Each dashboard's regression definition is captured in the on-call runbook. A regression on any of these dashboards during the 5-business-day observation window blocks the project-level definition of done from closing until the regression is resolved or accepted as a non-issue by the engineering lead.
- **Rollback procedure:** flip the feature flag to OFF (frontend hides the new pages); ingest worker continues but the user-facing surface is gone. Data state remains consistent (no destructive rollback needed). Cortex circuit-breaker stays armed. Success criterion for the rollback itself: feature surface is gone within 5 minutes of flag-flip and no errors propagate to the rest of the platform's dashboards.
- **Communication:** the engineering lead notifies the VP of CS and the on-call channel before each production change (flag-on, flag-off). If the rollback path fires, the engineering lead pages the on-call rotation lead and posts to the incident channel with the dashboard link that triggered the rollback.

## Definition of done

Each criterion is a yes/no check evaluable without judgment. The project is complete when all five are true simultaneously and have been so for the full 5-business-day observation window. If any criterion regresses inside the window, the window resets — partial credit is not awarded for criteria that were briefly true.

- All four phases' approval gates have been met and recorded.
- Production feature flag is ON for all ~12 CS Managers.
- On-call runbook is reviewed and active.
- The first 5 business days of production usage are observed without observability regressions or audit-relevant findings.
- TTI dashboard shows the FR-3.1 trend tile populated and visible to the VP of CS.

## Open execution questions

Each item below is a candidate for follow-up before the corresponding phase begins; neither blocks plan ratification, but both should be resolved before the relevant phase enters its gate window. Both questions are deliberately scoped to be answerable from measurements taken during the phase that precedes the decision — they are not architectural reopens.

- Whether the 1-hour JWT lifetime is the right value for production or should be tightened for the launch window (e.g., 30 minutes for the first 2 weeks); defer to a Phase 4 review with Priya Patel.
- Whether to keep the Cortex circuit-breaker's threshold at 1000 calls/hour (the ADR-003 default) or set it lower for v1 to keep cost certainty tighter; defer to a Phase 2 measurement-based decision.
- Whether the synthetic detractor traffic generator used for the Phase 2 gate (a) measurement should be retained as a permanent staging tool for future regression testing; defer to a post-Phase-2 tooling review.

---

Next step: the *Agent execution boundaries* table above is the input to `agents-md-generator.md`. Run that prompt next to produce the project's AGENTS.md / CLAUDE.md / SKILL.md file, or close the chain here if no autonomous-agent execution is planned. If the table changes after the agent file is generated, regenerate the agent file rather than hand-editing it.
