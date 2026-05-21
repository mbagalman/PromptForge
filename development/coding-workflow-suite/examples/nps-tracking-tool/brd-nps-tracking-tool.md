# BRD: Internal NPS-Tracking Tool

**Stage:** BRD
**Project:** Internal NPS-Tracking Tool
**Date:** 2026-05-21
**Upstream artifacts:** none

## Assumptions and inferred inputs

| Input | Source | Notes |
|---|---|---|
| $1.4M FY2025 ARR-loss baseline from detractor accounts that churned 8+ days after filing a response | user-confirmed | Sarah Chen's team produced this figure from FY2025 churn analysis; reused verbatim as the cost-of-inaction anchor. |
| 14-day average time-to-intervention (TTI) for detractor responses in Delighted | user-confirmed | Measured against current Delighted queue handling; treated as the pre-launch baseline for outcome O1. |
| ~12 CS Managers as the daily-user population | inferred | User named "CS Managers" as the affected group; the headcount is inferred from current CS org size and confirmed during scoping. |
| Detractor defined as NPS score ≤ 6 (the standard Delighted boundary) | inferred | User referenced "detractor responses" without restating the score boundary; inferred from Delighted's default convention and Acme's existing usage. Flagged here so the PRD can confirm before instrumentation. |
| "Time-to-intervention" measured from webhook receipt to first logged CS-Manager action | inferred | The 14-day baseline figure exists, but the precise start/end events of the measurement were not stated; inferred to match the only definition that supports the post-launch attestation in Success criteria. |

## Executive summary

Acme Corp's Customer Success team currently sees detractor NPS responses (scores ≤ 6)
an average of **14 days** after they land in the company's Delighted instance.
FY2025 churn analysis attributes **$1.4M of lost ARR** to accounts whose detractor responses
sat unread for **8+ days before final cancellation** — windows in which CS intervention
was demonstrably still possible based on the team's own post-mortem of those accounts.

This BRD commits the business to two measurable outcomes within **2 quarters of production launch**:
cut detractor TTI from **14 days to 5 days** (O1), and recover at least **30% ($420K)**
of the historically-unintervened-churn dollar value (O2).
The primary decision-maker is **Sarah Chen, VP of Customer Success**;
the headline constraint is a budget cap of **1 FTE-quarter of engineering and $0 incremental SaaS spend**,
with the existing **SOC 2 Type II audit posture preserved**.

## Business context

- **Situation:** Detractor survey responses arrive in Delighted but are not surfaced to CS Managers
  on any predictable cadence. The queue is reviewed reactively, and high-risk responses are routinely
  discovered only after a renewal conversation has already gone sideways — or after the account has cancelled.
- **Affected groups:** ~12 CS Managers (daily users of the surfacing tool);
  Survey Ops (tool-adjacent — no operational change, but data flows through their existing Delighted instance).
- **Cost of inaction:** $1.4M ARR lost in FY2025 from detractor accounts that churned 8+ days after filing a response —
  i.e., losses the team had a documented signal for and missed.
  Continuing the current pattern is reasonably modeled at a comparable annual run-rate against FY2026 retention targets.
- **Why now:** FY2026 retention targets are set against the FY2025 baseline,
  and the Delighted webhook contract (the data source this work depends on) runs through 2027 —
  both the business pressure and the technical window favor acting this cycle rather than deferring to FY2027.

## Stakeholders

| Role | Name or named role | Notes |
|---|---|---|
| Primary decision-maker | Sarah Chen — VP of Customer Success | Funds, kills, or rescopes the work. Owns the FY2026 retention number this BRD is built around. |
| Sign-off owner | David Kim — CFO | Required for any new tooling cost or new vendor relationship; constrains the $0-incremental-spend boundary. |
| Sign-off owner | Priya Patel — Data team lead | Approves Snowflake access patterns and is accountable for preserving the SOC 2 Type II audit posture. |
| Affected group | ~12 CS Managers | Daily users of the surfaced at-risk view; success depends on their adoption over current email-triage habits. |
| Affected group | Survey Ops | Tool-adjacent; no workflow change, but the Delighted webhook they steward is the upstream data dependency. |

## Business outcomes

1. **O1 — Reduce detractor time-to-intervention.**
   Decrease average TTI for detractor responses from **14 days to 5 days**
   within **2 quarters** of production launch.
2. **O2 — Recover unintervened-churn dollar value.**
   Prevent at least **$420K** of the **$1.4M** FY2025 unintervened-churn baseline
   (≥30% recovery) within the same 2-quarter window post-launch.

## Success criteria

The business will declare this work a success if, **2 quarters after production launch**,
both of the following hold and are jointly attestable by Sarah Chen and Priya Patel:

- **TTI threshold met.** Mean TTI on detractor responses — measured from Delighted webhook receipt
  to the first logged CS-Manager action against the account — is **≤ 5 days**,
  sustained across the trailing 60 days at the 2-quarter mark.
- **ARR recovery threshold met.** Cumulative ARR retained on accounts that filed a detractor response
  and were touched within the new TTI window is **≥ $420K**,
  measured against the FY2025 unintervened-churn cohort methodology Priya Patel's team already uses.

Either criterion missing its threshold at the 2-quarter mark is a "no" — not a "well, sort of."
A partial result (one of two met) triggers a documented review with the primary decision-maker
before any v2 commitment is made.

Both criteria are deliberately tied to actions the CS team controls
and to ARR figures the Data team already produces —
no new measurement infrastructure is required to attest to either at the 2-quarter mark,
which keeps the success test within the budget and SOC 2 constraints stated below.

## Constraints

- **Budget:** **1 FTE-quarter of engineering** and **$0 incremental SaaS spend**.
  No new paid tooling, no new vendor data processors without procurement review.
- **Timeline:** Production launch within **2 quarters** of work start;
  the 2-quarter outcome measurement window begins at launch.
- **Regulatory / compliance:** **SOC 2 Type II posture must be preserved.**
  Any access pattern that would require re-attestation is out of bounds without Priya Patel's explicit sign-off.
- **Organizational:** Must use the **existing Snowflake warehouse** as the analytics substrate;
  survey data must stay in the **existing Delighted instance** and be read via the existing webhook —
  no data egress to a new processor.
- **Other:** Internal-only tool; no customer-facing surface, no external API.

## Non-goals

- **Not a replacement for the Delighted survey tool itself** —
  Delighted remains the system of record for survey collection.
- **Not customer-facing** —
  no customer-visible UI, email, or notification produced by this tool in v1.
- **Expansion-opportunity scoring is out of v1** —
  promoters and upsell signals are explicitly deferred; revisit in v2.
- **Automated intervention actions are out of v1** —
  the tool surfaces at-risk accounts; humans act.
  No auto-emails, auto-tickets, or auto-tasks created by the tool itself.

## Risks and dependencies

| Item | Type | Owner or upstream | Mitigation or watch |
|---|---|---|---|
| Delighted webhook availability | Dependency | Survey Ops; Delighted vendor contract through 2027 | Contract horizon covers the outcome window; monitor any 2027 renewal renegotiation that could change webhook terms. |
| CS Manager adoption — tool may add friction over existing email-triage habit | Risk | Sarah Chen | Rollout phase of the Implementation Plan owns adoption tactics (in-context training, default landing surface, weekly TTI readout to CS leadership). |
| Snowflake Cortex (LLM functions) cost trajectory if usage scales beyond v1 scope | Risk | Priya Patel; David Kim | Hard-cap Cortex usage to the v1 detractor-classification path; review cost weekly during ramp; expansion-scoring (a likely next consumer) is explicitly out of v1. |

## Open questions

- Should at-risk surfacing **email/Slack CS Managers proactively**, or remain **pull-only** in a dashboard?
  Defer to the PRD — this is an FR-level decision that depends on adoption modeling the BRD does not own.
- Should **v2 expansion-opportunity scoring** reuse this pipeline or live as a separate system?
  Out of scope for v1; revisit in **Q1 2027** with Sarah Chen and Priya Patel before any v2 commitment.
