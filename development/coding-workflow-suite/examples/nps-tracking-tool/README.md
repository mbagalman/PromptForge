# Worked example — Internal NPS-Tracking Tool

This directory is the full artifact-by-artifact rendering of one complete chain through the Coding Workflow Suite. It pairs with — and goes deep on — the compact example in the [folder README's Workflow section](../../README.md#compact-worked-example). Where the compact example shows the chain *shape* in nine paragraphs, this directory shows the chain *substance* in nine artifacts totaling about 1,250 lines.

**The hypothetical:** an internal NPS-tracking tool for Acme Corp's Customer Success team. Detractor survey responses (NPS ≤ 6) currently land in the company's Delighted instance and sit unread on average 14 days before a CS team member sees them; analysis of FY2025 churn data shows $1.4M ARR was lost from accounts that filed a detractor response 8+ days before final cancellation. The project's goal is to surface at-risk accounts to CS Managers within minutes of a new detractor response, with a measurable target of reducing time-to-intervention from 14 days to 5 days within two quarters and recovering at least 30% of the historic unintervened-churn dollar value in the same window.

**Project size:** Large (per the suite's size-scaling rules — multi-stakeholder, multi-quarter, architecturally novel for the team). The chain that runs is the full Large chain.

**Status of every artifact in this directory:** explicitly hypothetical. Names, numbers, decisions, and stakeholders are realistic but fabricated. Do not treat any specific decision (e.g., "use Snowflake Tasks with a 60-second window") as a recommendation outside this example's context.

## Reading order

The chain ran top-to-bottom through Large mode, with the ADR loop firing on three open architectural questions from the Tech Spec Draft. Read in this order:

1. **[brd-nps-tracking-tool.md](brd-nps-tracking-tool.md)** — Business Requirements Document. Captures the $1.4M churn baseline, the two measurable outcomes (TTI reduction, dollar recovery), the stakeholder accountability map (VP of CS Sarah Chen as primary decision-maker; CFO and Data lead as sign-off owners), constraints (SOC 2, existing Snowflake, no new vendors, 1 FTE-quarter budget), and the non-goals that scope the work tightly.
2. **[prd-nps-tracking-tool.md](prd-nps-tracking-tool.md)** — Product Requirements Document. Translates the BRD into user-visible behavior: CS Manager and VP personas, five functional requirements (FR-1.1 through FR-3.1) with inline acceptance criteria, five named NFR dimensions populated per the per-dimension discipline, five out-of-scope items, two open product questions deferred to Tech Spec.
3. **[tech-spec-nps-tracking-tool-draft.md](tech-spec-nps-tracking-tool-draft.md)** — Tech Spec (Draft mode). Components (ingest worker, scoring service, frontend, Read API), data model, interfaces, failure modes, security, observability — plus three explicitly *surfaced* open architectural questions: Q-001 (real-time vs. batch scoring), Q-002 (auth model), Q-003 (storage of detractor driver summaries). The Draft refuses to silently pick these.
4. **[adr-001-streaming-vs-batch-scoring.md](adr-001-streaming-vs-batch-scoring.md)** — resolves Q-001. Micro-batch with a 60-second Snowflake Task window. Type 2 reversibility.
5. **[adr-002-auth-model.md](adr-002-auth-model.md)** — resolves Q-002. JWT issued by Auth0 with offline verification, 1-hour token lifetime. Type 1 reversibility (hard to reverse).
6. **[adr-003-driver-storage.md](adr-003-driver-storage.md)** — resolves Q-003. Raw text in Snowflake + read-time summarization via Cortex, cached in `comment_summary`. Type 2 reversibility.
7. **[tech-spec-nps-tracking-tool-final.md](tech-spec-nps-tracking-tool-final.md)** — Tech Spec (Revision mode). Absorbs the three ADR decisions into the Component breakdown, Failure modes, Security, and Observability sections. Every open architectural question is closed with `Resolved (see ADR-NNN)`. Implementation Plan can now proceed.
8. **[implementation-plan-nps-tracking-tool.md](implementation-plan-nps-tracking-tool.md)** — four-phase work breakdown with deterministic approval gates between phases, confidence-banded effort estimates, a four-item risk register, test strategy, rollout plan, and the load-bearing single project-level *Agent execution boundaries* table that the AGENTS.md generator consumes.
9. **[AGENTS.md](AGENTS.md)** — the agent-instruction file generated from the Implementation Plan and the three ADRs. This is what the engineering team commits to the project repo. Under 200 lines per the best-practices guide's ceiling.

## What this example exercises

The example was chosen to put weight on every load-bearing piece of the suite's design. In particular:

- **The ADR loop.** Three open architectural questions surfaced in the Draft; three ADRs produced; the Final marks each as resolved with the matching ADR-NNN. The `Q-NNN` ↔ `Originating question-id` matching is visible across the artifacts.
- **Type 1 vs. Type 2 reversibility marking.** ADR-002 is explicitly Type 1 (the auth model is hard to reverse post-launch); ADR-001 and ADR-003 are Type 2. The Implementation Plan's risk register and the AGENTS.md's counterintuitive-conventions section both treat the Type 1 decision differently than the Type 2 ones.
- **The input-type taxonomy.** Every artifact's *Assumptions and inferred inputs* section sits near the top and records which inputs were `supplied` (from upstream artifacts), `inferred` (filled in by the prompt without confirmation), or `user-confirmed` (supplied inline by the user, replacing what would normally come from upstream). The BRD's Assumptions table is denser than later artifacts' because the BRD has no upstream.
- **Deterministic approval gates.** Every phase in the Implementation Plan carries at least one deterministic gate criterion. None say "looks good"; all name specific recorded approvals, test outcomes, or telemetry thresholds.
- **Permission boundaries with verbs and surfaces.** The Implementation Plan's *Agent execution boundaries* table classifies actions as `allowed without approval`, `requires approval`, or `prohibited`, with specific verbs (`edit files under src/lib/`, `force-push to main`) and specific surfaces (paths, environments, services). The AGENTS.md inherits these verbatim.
- **The non-redundancy discipline.** The AGENTS.md does not duplicate content from the project's hypothetical README (assumed to cover "what is this codebase" at a high level); it carries only the non-inferable content per Lulla et al.'s evidence in the AGENTS.md best-practices guide.

## How the orchestrator handed this off

This worked example was run in **Router mode** — the user opened a session with `workflow-orchestrator.md`, answered the Phase 1 questions, and the orchestrator named each next stage to run. After every stage's artifact was produced, the user pasted it back into a new orchestrator session for the next handoff. Six orchestrator interactions produced this directory's nine artifacts.

A representative Router-mode turn between Tech Spec Draft and the ADR loop looked like this:

> *— from `workflow-orchestrator.md` —*
>
> **Current stage:** ADR loop (3 ADRs)
> **Current mode:** Router
>
> **Prior artifact status:** Tech Spec Draft `tech-spec-nps-tracking-tool-draft.md` produced (250 lines). Three open architectural questions surfaced: Q-001 (real-time vs. batch scoring), Q-002 (auth model), Q-003 (driver storage).
>
> **Next action:** open a new session with `adr.md` and run it once per open question. Start with Q-001; the decision is independent of Q-002 and Q-003, so the order doesn't matter for correctness but starting with the most architecturally fundamental decision (scoring cadence) tends to expose constraints the others can lean on.
>
> **What to paste into the first ADR session:** the Q-001 block from the Tech Spec Draft's *Open architectural questions* section (the question, the candidate approaches, the forces in tension). The ADR prompt will assign `**Originating question-id:** Q-001` to the produced ADR automatically.
>
> **Approval gate for advancing past the ADR loop:** every Q-NNN in the Draft must have a matching resolved ADR (status `accepted` or `proposed`, not blank). Decision-prep mode ADRs (status `proposed` with blank Decision/Consequences) do *not* satisfy this gate — the orchestrator will route you back to complete those before advancing to Tech Spec Revision.
>
> **When you have all three ADRs:** start a fresh orchestrator session and paste in the Draft plus the three resolved ADRs. The Tech Spec prompt will detect Revision mode from the input shape and produce the Final spec.

The same handoff in **Integrated mode** would have looked like this — same content, no tool switch:

> *— from `workflow-orchestrator.md` in Integrated mode —*
>
> **Current stage:** ADR loop (ADR for Q-001)
> **Current mode:** Integrated
>
> **Prior artifact status:** Tech Spec Draft produced in this same conversation (above). Three open architectural questions surfaced.
>
> *This artifact was produced by applying `adr.md`'s rules in Integrated mode. Review `adr.md`'s full contract if you want to audit.*
>
> ```
> # ADR-001: Streaming vs. batch scoring
>
> **Stage:** ADR-001
> **Project:** Internal NPS-Tracking Tool
> **Date:** 2026-05-21
> **Upstream artifacts:** tech-spec-nps-tracking-tool-draft.md
> **Originating question-id:** Q-001
>
> [the rest of ADR-001 as `adr.md` would produce it]
> ```
>
> **Next action:** I'll continue with the ADR for Q-002 in the next turn. Approve this ADR (or request edits) before I advance.

The two modes produce the same artifacts; they differ only in whether the user is switching tools between stages or staying in one conversation with the orchestrator applying each stage prompt's rules in line.

## Cover note for the AGENTS.md

The AGENTS.md in this directory is the **generated file** — the markdown the user commits to their project repo. The agent-instruction-file generator (`agents-md-generator.md`) produces a two-part response: this file plus a *cover note* for the human that names what was consumed, what was deferred, what to verify before the agent's first run, and the pre-deploy checklist results. The cover note does not get committed alongside the AGENTS.md; it lives in the conversation. Reproduced here for the example:

> **Cover note — AGENTS.md generation for Internal NPS-Tracking Tool**
>
> **Consumed:**
> - `implementation-plan-nps-tracking-tool.md` (Agent execution boundaries verbatim)
> - `adr-001-streaming-vs-batch-scoring.md`, `adr-002-auth-model.md`, `adr-003-driver-storage.md` (counterintuitive conventions from each)
> - `tech-spec-nps-tracking-tool-final.md` (exact tooling and versioning, executable command strings, trust-boundary inputs)
>
> **Target format:** AGENTS.md (the cross-vendor standard; works for Codex, Claude Code, Copilot, Gemini CLI). Choose `CLAUDE.md` or `SKILL.md` instead if your team operates primarily in one of those ecosystems; the content is the same.
>
> **What was included:** purpose line; exact tooling and versioning (Python 3.12, Node 22, React 19, Snowflake, Auth0, GitHub Actions, Docker, Datadog, sqitch); executable command strings (setup, test, lint, accessibility, local dev, schema migration); five counterintuitive conventions (the scoring service is a Snowflake Task; JWT verification is offline; summarization is read-time; PII never leaves Snowflake; Cortex calls gated by circuit-breaker); permission boundaries verbatim from the Implementation Plan; trust boundaries (trusted vs. untrusted inputs); deterministic done criteria; references to upstream artifacts.
>
> **What was deferred to `references/`:** none — all canonical content fit comfortably under the 200-line ceiling (final size: 121 lines). If the project adds more architectural decisions over time, the next ADRs' counterintuitive conventions should land in `references/decisions.md` rather than expanding `AGENTS.md` itself, with the AGENTS.md pointing at the reference file.
>
> **Pre-deploy checklist** (per the AGENTS.md best-practices guide §4.7):
> - File ≤ 200 lines: ✓ (121 lines)
> - Non-inferable only (no README/`package.json` duplication): ✓ (assumes the project README covers what the codebase is and basic setup)
> - Exact tooling / versions / permissions stated: ✓
> - Trusted vs. untrusted boundaries explicit: ✓
> - Done criteria deterministic and checkable: ✓
> - File under version control with real review process: ⚠ (user's project responsibility — verify)
> - No secrets / credentials / security-by-obscurity: ✓
> - Runtime monitoring/trace-review system in place: ⚠ (user's project responsibility — verify Datadog dashboards cover the agent's tool-use traces)
>
> **To verify before the agent's first run:**
> - Confirm the executable commands actually run in the project's environment (some command strings — `docker compose up -d snowflake-emulator vault-mock auth0-mock` — assume a local docker-compose stack that exists in the canonical example but should be verified against the real project).
> - Confirm the `requires approval` boundaries match the team's risk tolerance — the Auth0 application configuration boundary is conservative for a small team and may be relaxed for larger ones.
> - Confirm the PII-redaction SDK at `src/lib/logging/` exists and is the canonical place to enforce the "no PII in logs" convention.

## Where to go next

- **Reading another stage's full output:** open the files in this directory in the reading order above.
- **Understanding why each prompt enforces what it enforces:** read the corresponding stage prompt in [`../..`](../..) (e.g., [`../../tech-spec.md`](../../tech-spec.md) for the Tech Spec contract).
- **Running the suite yourself on a new project:** start with [`../../workflow-orchestrator.md`](../../workflow-orchestrator.md). Phase 1 will ask the size-scaling question and pick the right chain for your project.
- **Stress-testing the suite against its documented failure modes:** see the regression-scenario catalog at [`../../tests/regression-scenarios.md`](../../tests/regression-scenarios.md).
