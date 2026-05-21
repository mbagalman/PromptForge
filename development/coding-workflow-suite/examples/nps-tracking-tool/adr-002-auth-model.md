# ADR-002: Auth0-issued JWT with offline verification for the Read API

**Stage:** ADR-002
**Project:** Internal NPS-Tracking Tool
**Date:** 2026-05-21
**Upstream artifacts:** tech-spec-nps-tracking-tool-draft.md
**Originating question-id:** Q-002

## Assumptions and inferred inputs

| Input | Source | If inferred or user-confirmed: notes |
|---|---|---|
| Architectural context | supplied (Tech Spec Draft §Open architectural questions, Q-002) | — |
| Originating question_id | supplied | — |
| Candidate options | supplied (Tech Spec Draft §Open architectural questions, Q-002) | — |
| Reversibility classification | user-confirmed | Type 1 — switching auth models post-launch requires coordinated changes across frontend, Read API, Auth0 tenant config, and the CSM login flow. |

## Status

accepted (signed off by Sarah Chen as primary stakeholder via Priya Patel's security review)

## Context

Q-002 from the Tech Spec Draft asks how the CSM-facing frontend should authenticate to the Read API. Three forces shape the answer.

First, the company's identity direction is Auth0-first SSO; any auth model that bypasses Auth0 is fighting the org-wide pattern and the security team's review template. Second, the team's operational surface is constrained to the existing stack — standing up a new service (a session store, a token-introspection proxy, or anything else with its own backup and monitoring needs) is outside both budget and the existing-stack constraint named in the Tech Spec Draft. Third, the Read API's per-request latency budget matters because the CSM dashboard is interactive; any auth scheme that adds a synchronous network hop per request will be visible to users and to the SOC 2 review of operational dependencies.

This decision needs recording with particular care because the choice is hard to back out of. The frontend's auth library, the Read API's verification middleware, the Auth0 tenant's application configuration, and the CSM-facing login flow all change together if the team later wants a different model. A future reader — especially one considering adding a second client, such as the deferred mobile experience or a future expansion-scoring tool — needs to know which forces dominated and which were traded away, so they can revisit the choice deliberately rather than by accident.

## Decision

We will use a JWT issued by the existing Auth0 tenant via the standard SPA flow, verified offline by the Read API against Auth0's JWKS endpoint (cached for 1 hour), with a 1-hour token lifetime and refresh via Auth0's refresh-token flow.

## Consequences

**Reversibility:** Type 1 — hard to reverse. Switching auth models post-launch is a multi-team coordination cost: the frontend, the Read API, Auth0 tenant configuration, and the CSM-facing login flow all change together. The team should re-validate this choice before adding any second client (e.g., the deferred mobile experience or a future expansion-scoring tool).

### Positive

- Matches the company's Auth0-first SSO direction and the existing-stack constraint.
- No new service dependencies — no session store, no introspection proxy — to operate, monitor, or back up.
- Standard SPA + JWT pattern with broad library support on both the frontend and the Read API side.
- Offline verification means the Read API does not call Auth0 on every request; per-request latency is unaffected by Auth0 availability or rate limits.

### Negative

- Revocation latency up to 1 hour: a CSM whose access is revoked may continue to view the at-risk list until their JWT expires. Mitigated by short-lived tokens and documented in the security considerations section of the Tech Spec.
- Reversal cost is real and multi-team. Backing out of JWT-with-offline-verification means coordinated changes across the frontend, Read API verification middleware, Auth0 tenant configuration, and the CSM-facing login flow — not a single-service swap. This is the canonical Type 1 cost.

### Neutral

- The 1-hour token lifetime balances revocation latency against user friction (no mid-session re-authentication for typical CS workflows).
- JWKS cache TTL (1 hour) means Auth0 key-rotation events propagate to the Read API within an hour; documented for the on-call runbook.

## Options considered

### Option 1 — Session-based auth with a backend session store

- **Description:** Issue an opaque session cookie on login; the Read API validates against a session store (Redis or similar) on each request.
- **Why rejected:** Offers fast revocation but requires a session-store dependency the team does not currently operate. New service to deploy, monitor, and back up. Outside budget and outside the existing-stack constraint.

### Option 2 — JWT issued by Auth0 with offline verification (CHOSEN)

- **Description:** Frontend obtains a JWT from the existing Auth0 tenant via the standard SPA flow; the Read API verifies the JWT signature offline against Auth0's JWKS endpoint (cached for 1 hour). Token lifetime: 1 hour; refresh via Auth0's refresh-token flow.
- **Why accepted:** Matches the company's Auth0-first SSO direction. No new service dependencies. Offline verification means the Read API does not call Auth0 on every request. Revocation latency is bounded by token lifetime (1 hour) — acceptable given short token lifetime and the documented mitigation.

### Option 3 — Opaque-token issued by Auth0 with introspection

- **Description:** Auth0 issues an opaque token; the Read API calls Auth0's introspection endpoint on every request to validate it.
- **Why rejected:** Solves revocation latency but adds an Auth0 introspection call to every Read API request, with both latency and cost implications. Auth0's introspection endpoint is rate-limited.

## Related ADRs

- **Depends on:** ADR-001 (Micro-batch NPS scoring on a 60-second window) — the Read API authenticated by this scheme serves the `nps_score` values computed by ADR-001's task; auth sits on top of the scoring contract.
- **Influences:** ADR-003 (Storage and computation of detractor driver summaries) — the Read API governed by this auth scheme is the same surface that triggers Cortex summarization on first read in ADR-003; token-lifetime assumptions carry into the drill-down flow.
