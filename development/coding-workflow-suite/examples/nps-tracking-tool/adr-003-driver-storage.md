# ADR-003: Raw text in Snowflake with read-time Cortex summarization, cached

**Stage:** ADR-003
**Project:** Internal NPS-Tracking Tool
**Date:** 2026-05-21
**Upstream artifacts:** tech-spec-nps-tracking-tool-draft.md
**Originating question-id:** Q-003

## Assumptions and inferred inputs

| Input | Source | If inferred or user-confirmed: notes |
|---|---|---|
| Architectural context | supplied (Tech Spec Draft §Open architectural questions, Q-003) | — |
| Originating question_id | supplied | — |
| Candidate options | supplied (Tech Spec Draft §Open architectural questions, Q-003) | — |
| Reversibility classification | user-confirmed | Type 2 — a vector store can be added later for semantic search without re-ingesting historical data. |

## Status

accepted (signed off by Priya Patel for SOC 2 posture and by David Kim for cost posture)

## Context

Q-003 from the Tech Spec Draft asks how the detractor "driver" summaries surfaced in the CSM drill-down view should be stored and computed. Three forces are in tension.

First, SOC 2 posture: survey content includes free-text comments that may contain PII, and the Tech Spec Draft's constraints rule out moving that content to any new data processor without a procurement review the team cannot absorb in the 2-quarter timeline. Second, the existing-stack constraint: standing up a new service (Postgres-with-pgvector, an external vector index) is outside scope. Third, the cost posture David Kim signs off on: Snowflake Cortex calls are not free, and any architecture that pays for summarization on responses no one reads is wasteful — the FY2025 dataset shows roughly 60% of detractor responses are never viewed.

This decision needs recording because the trade-space is not obvious from the outside. A future reader will notice that the system does not have a vector store and may assume the team simply did not consider semantic search; this ADR records that the option was on the table and was deferred for SOC 2 and existing-stack reasons rather than dismissed. The cache-on-first-read mechanism is also subtle enough that it deserves the showing-your-work treatment — the first-read latency is the visible cost of the cache choice, and the FR-2.1 acceptance criterion is the budget it spends.

## Decision

We will store raw response text in Snowflake's `survey_response.comment_raw` field with `comment_summary` initially NULL, then on first read call Snowflake Cortex's summarization function, populate `comment_summary`, and serve the result from cache on all subsequent reads.

## Consequences

**Reversibility:** Type 2 — reversible. Adding a vector store later (for semantic search across responses) is a v2 addition that does not require re-ingesting historical data.

### Positive

- No new vendor; survey content stays inside the Snowflake environment, preserving SOC 2 posture.
- Uses existing Snowflake Cortex — no new service to operate.
- Cache keeps repeat reads cheap; Cortex usage is bounded by actual read volume rather than ingest volume.

### Negative

- First-read latency on a new response is ~3 seconds (Cortex summarization call). CS Managers will see a brief loading state on first drill-down to a response.
- Mitigated by FR-2.1's drill-down acceptance criterion (≤ 3s for ≤50 responses); the budget is aligned but tight, and the Implementation Plan should track first-read latency in the rollout phase.

### Neutral

- Vector-store option preserved for v2 (semantic search across detractor responses) if and when search becomes a requirement; no v2 migration is required because raw text remains in Snowflake.

## Options considered

### Option 1 — External vector store (Pinecone or similar)

- **Description:** Embed each response and store the vector in a managed vector database; query semantically at read time.
- **Why rejected:** Provides best-in-class semantic search but introduces a new vendor data processor. Fails the SOC 2 constraint without a procurement review the team cannot absorb in the 2-quarter timeline. Survey content (including PII) would leave the Snowflake environment.

### Option 2 — pgvector via Postgres

- **Description:** Stand up a Postgres instance with the pgvector extension and store embeddings alongside response metadata.
- **Why rejected:** Avoids the new-vendor concern but requires standing up Postgres as a new service; outside the existing-stack constraint.

### Option 3 — Precomputed summary at ingest time, stored in `comment_summary`

- **Description:** Call Cortex on every response as it lands, write the summary alongside the raw text, and serve directly from the column at read time.
- **Why rejected:** Wastes Cortex compute on responses no one reads (estimated 60% of detractor responses are not viewed in the FY2025 dataset). Higher Cortex cost than the chosen approach for no functional gain.

### Option 4 — Raw text in Snowflake + read-time summarization, cached (CHOSEN)

- **Description:** Survey responses land in `survey_response` with `comment_raw` populated and `comment_summary` NULL. On first read, the Read API calls Cortex, populates `comment_summary`, and serves the result. Subsequent reads hit the cached value.
- **Why accepted:** Survey content stays in Snowflake (SOC 2-safe). Cortex usage is bounded by actual read volume rather than ingest volume. Cache makes repeat reads cheap.

## Related ADRs

- **Depends on:** ADR-001 (Micro-batch NPS scoring on a 60-second window) — both decisions keep the data path on Snowflake-native primitives; the SOC 2 and existing-stack constraints that argued for micro-batch scoring also argued against an external vector store here.
- **Depends on:** ADR-002 (Auth0-issued JWT with offline verification) — the Read API that triggers Cortex summarization on first read is the same surface governed by ADR-002's auth scheme.
