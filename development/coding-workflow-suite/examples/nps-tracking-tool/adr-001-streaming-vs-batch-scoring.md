# ADR-001: Micro-batch NPS scoring on a 60-second window

**Stage:** ADR-001
**Project:** Internal NPS-Tracking Tool
**Date:** 2026-05-21
**Upstream artifacts:** tech-spec-nps-tracking-tool-draft.md
**Originating question-id:** Q-001

## Assumptions and inferred inputs

| Input | Source | If inferred or user-confirmed: notes |
|---|---|---|
| Architectural context | supplied (Tech Spec Draft §Open architectural questions, Q-001) | — |
| Originating question_id | supplied | — |
| Candidate options | supplied (Tech Spec Draft §Open architectural questions, Q-001) | — |
| Reversibility classification | user-confirmed | Type 2 — Snowflake Task schedule is the only knob; no downstream artifact changes required to retune. |

## Status

accepted (signed off by Priya Patel for data architecture and David Kim for cost posture)

## Context

Q-001 from the Tech Spec Draft asks how survey responses should flow from ingest to the `nps_score` value the CSM-facing UI displays. FR-1.1 sets the acceptance criterion: the dashboard score must reflect a newly arrived response within two minutes (P95). The Tech Spec Draft frames three forces that pull against each other.

First, the team's operational surface is Snowflake plus a Python ingest worker — no streaming infrastructure (Kafka or similar) is currently operated, and the 1 FTE-quarter budget cannot absorb a new managed-streaming vendor relationship without breaking the existing-stack constraint. Second, the FR-1.1 latency target rules out a naive hourly batch refresh; the staleness window alone exceeds the requirement. Third, SOC 2 review of the data path favors a single, auditable computation surface over a fan-out across multiple services.

This decision needs recording because the choice between continuous streaming, periodic batch, and micro-batch on Snowflake-native primitives shapes the ingest worker's contract, the Read API's freshness assumptions, and the cost line item David Kim signs off on. A future reader will want to know why the team did not adopt streaming — the FR-1.1 target alone would have justified it — and why the cadence settled at 60 seconds rather than tighter.

## Decision

We will use a micro-batch architecture: the Python ingest worker writes survey responses to Snowflake on arrival, and a Snowflake Task runs every 60 seconds to recompute `nps_score` for any account whose `survey_response.received_at` is newer than the task's last run.

## Consequences

**Reversibility:** Type 2 — reversible. The 60-second cadence can be tightened toward streaming or relaxed toward hourly batch by changing the Snowflake Task schedule; no downstream artifact changes required.

### Positive

- Meets FR-1.1 (2-minute refresh) with comfortable margin — ~75-second P95 worst-case staleness against a 2-minute requirement.
- Uses tools the Data team already operates (Snowflake Tasks); no new vendor relationship and no streaming runtime to learn.
- Single auditable execution path for SOC 2 — every score computation is visible in Snowflake's query history.

### Negative

- 60-second worst-case staleness in the UI; CS Managers may occasionally see a response one cycle late relative to its arrival.
- Snowflake Tasks costs scale with frequency — flagged for monitoring in the Implementation Plan rollout phase.

### Neutral

- Review the 60-second cadence at the 6-month mark; tighten if the latency budget allows it within the FR-1.1 margin.

## Options considered

### Option 1 — Streaming ingest with continuous scoring

- **Description:** Introduce a streaming processor (Kafka or managed equivalent) that scores responses continuously as they arrive.
- **Why rejected:** Would meet FR-1.1 with margin, but introduces a streaming processor the team does not currently operate and a new vendor relationship for managed streaming. Against the existing-stack constraint; the 1 FTE-quarter budget cannot absorb both new infrastructure and the application work.

### Option 2 — Pure batch with hourly refresh

- **Description:** Run a single Snowflake Task on an hourly cron to recompute scores for all accounts with recent activity.
- **Why rejected:** Fails FR-1.1's 2-minute acceptance criterion outright (max staleness ~1 hour). Even with a tightened cron, variance plus Snowflake Task scheduling lag pushes the P95 above the 2-minute requirement.

### Option 3 — Micro-batch with a 60-second window (CHOSEN)

- **Description:** Python ingest worker writes responses to Snowflake on arrival; a Snowflake Task runs every 60 seconds to recompute `nps_score` for any account with a newer `received_at`.
- **Why accepted:** Uses existing Snowflake Tasks (familiar to the Data team), satisfies FR-1.1 with margin (~75-second P95 worst-case staleness vs. the 2-minute requirement), and offers a single auditable execution path via Snowflake query history.

## Related ADRs

- **Influences:** ADR-002 (Auth model for the frontend) — the Read API the frontend authenticates against serves the `nps_score` values computed by this task; the auth-model choice is downstream of the scoring contract.
- **Influences:** ADR-003 (Storage and computation of detractor driver summaries) — both this ADR and ADR-003 keep the data path on Snowflake-native primitives; the SOC 2 and existing-stack constraints argued here apply there as well.
