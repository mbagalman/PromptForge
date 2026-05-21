# PRD: Internal NPS-Tracking Tool

**Stage:** PRD
**Project:** Internal NPS-Tracking Tool
**Date:** 2026-05-21
**Upstream artifacts:** brd-nps-tracking-tool.md

## Assumptions and inferred inputs

| Input | Source | If inferred or user-confirmed: notes |
|---|---|---|
| Business outcomes (TTI 14d to 5d in 2Q; recover $420K+ of unintervened-churn dollars) | supplied (BRD Business outcomes) | — |
| Stakeholders and sign-off owners (VP CS Sarah Chen; CFO David Kim; Data lead Priya Patel) | supplied (BRD Stakeholders) | — |
| Non-goals (mobile, predictive expansion scoring, CRM integration, etc.) | supplied (BRD Non-goals) | — |
| Regulatory constraints (SOC 2 Type II, GDPR EU residency) | supplied (BRD Constraints) | — |
| Personas (CS Manager primary; VP of Customer Success secondary) | user-confirmed | BRD named stakeholders but did not name product personas; CS leadership confirmed daily user is the CSM and the VP is a weekly-cadence consumer. |
| Performance target (P95 page load < 2s for <=5,000 accounts in scope) | user-confirmed | Threshold chosen in session against the observed account-base size; not a contractual SLA. |
| "Primary detractor driver" presentation as plain language (not raw features) | inferred | UX requirement inferred from CSM workflow context; not explicitly called out in BRD. Confirm during design review. |

## Product overview

The Internal NPS-Tracking Tool is a desktop web application for Acme Corp's Customer Success team that surfaces detractor responses from the existing Delighted survey instrument and routes attention to the accounts most at risk of churn. The BRD established the business case: in FY2025, $1.4M of ARR churned from accounts whose detractor responses sat unread for an average of 14 days; the target is to compress that time-to-intervention (TTI) to 5 days in the second quarter of operation and recover at least 30% of that historic loss.

The tool is internal-only and supports — but does not replace — the existing human intervention workflow (email and call). Two surfaces are in scope: a daily working surface for the CS Manager (at-risk list, account drill-down, intervention marking) and a weekly review surface for the VP of Customer Success (aggregate trend tiles). Both run behind the existing Auth0 SSO and read from the existing Delighted feed; this PRD does not introduce a new survey instrument, a new identity provider, or a new system of record.

Scope boundary: this PRD covers the at-risk list, account drill-down, intervention marking, and the VP-level trend view. It does not cover predictive expansion scoring, mobile, automated outreach, or CRM integration; those are explicitly recorded in *Out-of-scope* below.

## Target users and personas

### CS Manager (primary)

- **Role / title:** Customer Success Manager, Acme Corp Customer Success team.
- **What they do with the product:** Reviews the at-risk-accounts list at the start of each workday, sorted by ARR descending. Drills into top accounts to read recent detractor responses in plain language, understand the primary driver, and decide whether to reach out. Marks accounts as "intervened" after sending an email or scheduling a call so peers don't double-touch the same account.
- **Context of use:** Daily, desktop browser, single-tenant internal app behind SSO. Expertise: domain-fluent in the customer book but not in modeling or analytics tooling. Holds 30–80 accounts in active rotation; sessions are short (5–15 minutes) and frequent.

### VP of Customer Success (secondary)

- **Role / title:** Vice President of Customer Success.
- **What they do with the product:** Reviews aggregate trend tiles weekly — total at-risk ARR, weekly count of new detractor responses, average TTI — to track whether the team is closing the response-to-intervention gap. Does not drill into individual accounts day-to-day; that's the CSM's surface.
- **Context of use:** Weekly, desktop browser, often in a leadership review. Expertise: high product fluency, not a daily user of the tool itself.

## User stories

- As a CS Manager, I want to see all at-risk accounts in one list sorted by ARR descending, so that I can spend my limited triage time on the accounts where churn would hurt most.
- As a CS Manager, I want each at-risk row to show the latest NPS score, days since the detractor response, and the primary driver in plain language, so that I can decide whether to dig in without opening the account.
- As a CS Manager, I want to drill into an account and read its detractor responses from the last 90 days in plain language, so that I can compose a relevant outreach without learning the underlying model.
- As a CS Manager, I want to mark an account "intervened" with a timestamp and a short note, so that my peers know the account is being handled and don't duplicate the outreach.
- As the VP of Customer Success, I want a weekly view of total at-risk ARR, new detractor volume, and average TTI, so that I can tell at a glance whether the team is closing the response-to-intervention gap.

## Functional requirements

### At-risk account list (CSM)

- **FR-1.1** CS Manager views a list of at-risk accounts (NPS <= 6 in the last 30 days) sorted by ARR descending.
  - **Acceptance criterion:** the list refreshes within 2 minutes of a new detractor response landing in Delighted.
  - **Dependencies:** Delighted survey feed; account-to-ARR mapping from the finance system of record.
- **FR-1.2** Each row in the at-risk list shows: account name, ARR (USD), latest NPS score, days since detractor response, and primary detractor driver in plain language.
  - **Acceptance criterion:** every list row populates all five columns; missing data renders as an em dash (—), not as an empty cell.
  - **Dependencies:** detractor-driver labelling pipeline (definition deferred to `tech-spec.md`).

### Account drill-down and intervention marking (CSM)

- **FR-2.1** CS Manager drills into an at-risk account and sees all detractor responses from that account in the last 90 days, presented in plain language rather than raw model features.
  - **Acceptance criterion:** drill-down view loads in under 3 seconds for accounts with 50 or fewer responses in the window.
  - **Dependencies:** survey-response storage in Snowflake; plain-language presentation layer.
- **FR-2.2** CS Manager marks an account "intervened" with a timestamp and a free-text note of approximately 500 characters; the mark is visible to other CS Managers viewing the same account.
  - **Acceptance criterion:** the intervention mark and note persist and are visible to all CSMs within 60 seconds of marking.
  - **Dependencies:** authenticated CSM identity from SSO; shared persistence visible across CSM sessions.

### VP trend view (VP of CS)

- **FR-3.1** VP of Customer Success views aggregate weekly trend tiles: total at-risk ARR, weekly count of new detractor responses, and average TTI.
  - **Acceptance criterion:** trend tiles update at least weekly and display a 13-week trailing window.
  - **Dependencies:** the same detractor-response feed as FR-1.1; the intervention-marking data from FR-2.2 to compute TTI.

## Non-functional requirements

### Performance

- P95 page load < 2 seconds for the at-risk list at the expected account-base size (<= 5,000 accounts in scope). Drill-down latency target is captured per-requirement in FR-2.1.

### Reliability

- 99.5% uptime during business hours (8am–7pm ET, Monday–Friday). Outside business hours: best-effort, no formal target.
- Graceful degradation when the Delighted feed lags: the at-risk list continues to render the last known good state with a visible staleness indicator. Specific UI for the staleness indicator is deferred to `tech-spec.md`.

### Security

- SOC 2 Type II posture preserved end-to-end. Survey-response PII — including account email addresses and free-text comments — remains in Snowflake; the application surfaces it on demand via the existing Auth0 SSO and never persists survey content in browser localStorage or any client-side store.

### Accessibility

- WCAG 2.1 AA conformance. The at-risk list and drill-down view are keyboard-navigable; screen-reader compatibility is verified before launch with at least one screen-reader pass over the primary CSM flow.

### Compliance

- GDPR data residency: detractor responses originating from EU customers are stored in the EU Snowflake instance only and are not replicated outside the EU region by this tool.

## Out-of-scope

- Predictive expansion-opportunity scoring (deferred to Q1 2027 per the BRD; outside the churn-prevention case this tool is justified against).
- Mobile experience. v1 is desktop only; phone-sized layouts are not a requirement.
- Automated intervention actions. The tool surfaces signal and supports humans; it does not send emails, schedule calls, or otherwise act on accounts.
- Multi-tenant customer-facing analog. This is an internal tool for Acme's CS team; there is no external-customer version of the surface.
- CRM integration. Two-way sync with the CS team's CRM is a separate workstream and is not part of v1.

## Open product questions

- Push vs. pull on detractor arrival: whether the tool should proactively notify CS Managers (email or Slack) when a new high-ARR detractor response lands, or remain pull-only with daily CSM check-ins. Deferred to v1.1; resolution will be based on adoption telemetry from v1.
- Source of the "primary detractor driver" label: whether the plain-language driver is model-generated, rule-based, or a combination. This is an architectural choice; deferred to `tech-spec.md`.

## Acceptance criteria summary

| FR | Acceptance criterion |
|---|---|
| FR-1.1 | At-risk list refreshes within 2 minutes of a new Delighted detractor response. |
| FR-1.2 | Every row populates all five columns; missing fields show as "—". |
| FR-2.1 | Drill-down loads in under 3 seconds for accounts with <= 50 responses in the 90-day window. |
| FR-2.2 | Intervention mark and note persist and are visible to all CSMs within 60 seconds. |
| FR-3.1 | VP trend tiles update at least weekly over a 13-week trailing window. |
