# Regression scenarios — Coding Workflow Suite

Minimal scenario catalog for exercising the suite's documented failure modes. Three scenarios per prompt plus three chain-handoff scenarios — twenty-four total. Each scenario states the input the user would paste, the expected behavior the prompt should produce, and the named failure mode being exercised.

These are not automated tests. They are rehearsals — load a prompt into Claude Projects, Custom GPT, or Gemini Gem, paste the input, and confirm the prompt produces the expected behavior. Failures should be triaged: either the prompt needs a fix or the scenario needs to be sharpened.

**Scenario codes** use a two-letter prompt prefix plus an index: `OR` (orchestrator), `BR` (BRD), `PR` (PRD), `TS` (tech spec), `AD` (ADR), `IP` (implementation plan), `AG` (AGENTS.md generator), `CH` (chain handoff).

**Status legend:** `untested` · `passing` · `failing` · `flaky`. Each scenario starts at `untested`; update inline as you run them.

## Run log

A cross-cutting record of which scenarios have been exercised against which prompt versions on which platforms. Add a row per run; update the per-scenario `Status:` field inline below. The run log tells you "where do we stand across the catalog"; the inline statuses tell you "which specific scenarios have been confirmed."

| Date tested | Prompt(s) | Prompt version | Model / platform | Scenarios run | Pass / fail / flaky | Notes |
|---|---|---|---|---|---|---|
| _untested_ | _all_ | _1.0.0 (initial)_ | _—_ | _—_ | _—_ | The catalog has not yet been exercised; this row will populate as runs occur. |

Convention: each run records one row. *Scenarios run* names them by code (`OR.1`, `OR.2`, …) or a range (`OR.1–3`, `CH.1–3`); for a full-catalog run, *all*. *Pass / fail / flaky* lists counts (e.g., `21 pass / 2 fail / 1 flaky`). *Notes* names which scenario codes failed or behaved unexpectedly, and links to any prompt-fix PR that addressed the failures.

---

## Workflow Orchestrator (`workflow-orchestrator.md`)

### OR.1 — Start mid-chain at Implementation Plan with only PRD supplied

**Status:** untested

**Input:** Phase 1 opener answer: "I want to jump to the Implementation Plan. I have a PRD for an internal NPS-tracking tool. Project is Large, greenfield, Router mode."

**Expected behavior:** Phase 1 recap acknowledges the user's stated size and mode but does *not* route to `implementation-plan.md`. Instead, the orchestrator names what is missing for a Large chain (Tech Spec Final and resolved ADRs) and offers two paths: (a) run `tech-spec.md` (Draft) → `adr.md` loop → `tech-spec.md` (Revision) to produce the missing artifacts, or (b) supply the missing content inline so the Implementation Plan run marks the relevant inputs `user-confirmed` in its *Assumptions and inferred inputs* table. Does not silently advance to the Implementation Plan with inferred upstream context.

**Failure mode exercised:** start-mid-chain with insufficient upstream artifacts — orchestrator demands the missing inputs rather than hallucinating them.

### OR.2 — Attempt to skip PRD (non-skippable)

**Status:** untested

**Input:** After Phase 1 (Large chain, greenfield, Router mode) and a completed BRD: "Skip the PRD — let's go straight to the Tech Spec. I'll fill in the product details as we go."

**Expected behavior:** orchestrator blocks the request. Cites C1 skip behavior: PRD is non-skippable for any chain. Refusal language: *"PRD is required for the workflow. If you have an out-of-band PRD, paste its content as the next stage's input and the orchestrator will mark it `user-confirmed`."* Offers the out-of-band path explicitly.

**Failure mode exercised:** skipping a required stage — orchestrator enforces the non-skippable list rather than complying.

### OR.3 — Mode switch from Router to Integrated mid-workflow

**Status:** untested

**Input:** Mid-workflow after the BRD and PRD have shipped in Router mode: "Switch to Integrated mode for the Tech Spec — I don't want to context-switch tools again."

**Expected behavior:** orchestrator acknowledges the mode switch, restates the current state (PRD complete; Tech Spec is next), and produces the Tech Spec Draft inline by applying `tech-spec.md`'s rules. The artifact opens with the required disclosure header: *"This artifact was produced by applying `tech-spec.md`'s rules in Integrated mode."* The canonical artifact-identity header block (`**Stage:** Tech Spec (Draft)` etc.) follows the disclosure line. State is preserved — the orchestrator does not re-elicit Phase 1.

**Failure mode exercised:** mid-workflow mode switch — orchestrator preserves state and produces the artifact in Integrated mode without re-running upstream phases.

---

## Business Requirements Document (`brd.md`)

### BR.1 — No measurable success criterion

**Status:** untested

**Input:** "We want to build an internal dashboard for the customer success team. They've been asking for better visibility into churn. Success means the team is happier with their tools."

**Expected behavior:** the BRD prompt does not produce an artifact. It pushes back on the "happier with their tools" framing and asks the user to articulate at least one measurable success criterion. If the user cannot, the session closes with a scope-stop note pointing the user to talk to stakeholders (CS leadership, finance) about what measurable outcome the work needs to produce. Does not invent a criterion.

**Failure mode exercised:** missing measurable success criterion — BRD enforces the floor rather than producing a vague artifact.

### BR.2 — User proposes a solution instead of an outcome

**Status:** untested

**Input:** "I need a BRD for a new feature: we're adding a churn-prediction model to our CS dashboard. The model will use logistic regression on engagement data and surface at-risk accounts."

**Expected behavior:** the BRD prompt redirects. *"The model architecture and surfacing logic are implementation details — the Tech Spec captures those. The BRD captures the business case: what outcome are we trying to produce, who feels the pain today, what is the cost of not solving it? Let's start there."* If the user resists, the prompt holds the line; the redirect happens at least once.

**Failure mode exercised:** implementation-creep into the BRD — the prompt enforces *outcome first, solution later*.

### BR.3 — No named primary decision-maker

**Status:** untested

**Input:** Full BRD elicitation with concrete situation, outcomes, and constraints, but: "The CS leadership, finance, product, and engineering teams all need to be involved. There isn't really one person who owns this decision — it's a cross-functional effort."

**Expected behavior:** BRD prompt does not produce the artifact. Surfaces that *"primary decision-maker"* is a role, not a literal single person — what's needed is the role with ultimate decision authority for the work. Asks the user to identify which role would approve or veto the project at the executive level. If the user cannot, the session closes with a scope-stop note: KPI / project sponsorship needs to be settled before the BRD can be produced.

**Failure mode exercised:** stakeholder-accountability fabrication — BRD enforces the *primary decision-maker is non-negotiable* rule rather than listing four roles as co-equal.

---

## Product Requirements Document (`prd.md`)

### PR.1 — Implementation creep (database choice)

**Status:** untested

**Input:** Mid-PRD elicitation after personas and functional requirements: "For the at-risk accounts list, should we use Postgres or MongoDB to back it? Also, should the frontend be React or Vue?"

**Expected behavior:** the PRD prompt redirects. *"Database and frontend framework choices are Tech Spec territory. The PRD specifies what the user can do (e.g., 'CS Manager views a sorted list of at-risk accounts'); the Tech Spec specifies how the system is built. Let's keep the PRD focused on user-visible behavior."* If the question hints at a non-functional requirement (e.g., scale or latency), the prompt captures that as an NFR but still defers the implementation choice.

**Failure mode exercised:** implementation-creep into the PRD — boundary between user-visible behavior (PRD) and system design (Tech Spec) enforced.

### PR.2 — Missing BRD; business context supplied inline

**Status:** untested

**Input:** PRD prompt opened with: "I don't have a BRD. Here's the context: we lose ~$1.4M/year to unintervened churn. The CS team owns the response. We want to surface at-risk accounts within 5 days of the survey response. Please proceed."

**Expected behavior:** PRD prompt accepts the inline business context, marks the BRD-equivalent inputs (business outcomes, stakeholders, success criteria) as `user-confirmed` in the produced artifact's *Assumptions and inferred inputs* table with notes naming what the user supplied. The artifact header's *Upstream artifacts* line reads `none` (or names "user-confirmed business context"). Does not silently treat the inline content as authoritative without surfacing it.

**Failure mode exercised:** missing-BRD mid-chain run — input-type taxonomy correctly marks user-supplied content as `user-confirmed` rather than `supplied` or `inferred`.

### PR.3 — "The system shall" phrasing on a functional requirement

**Status:** untested

**Input:** During functional requirements elicitation: "FR-1: The system shall send a notification email to the account owner when the risk score crosses the threshold."

**Expected behavior:** PRD prompt rewrites the requirement in user-perspective language and asks the user to confirm. Suggested rewrite: *"FR-1: The account owner receives a notification email when the risk score crosses the threshold."* If the user pushes back ("'the system shall' is fine"), the prompt explains briefly that the PRD describes what users see and do, not what the system does internally, and continues to push for the rewrite. Does not silently accept implementation-perspective phrasing.

**Failure mode exercised:** functional-requirement perspective drift — PRD enforces user-perspective rewriting.

---

## Tech Spec (`tech-spec.md`)

### TS.1 — Architectural decision baked in silently

**Status:** untested

**Input:** Mid-Tech-Spec elicitation: "For the scoring service, we'll use a streaming ingest with a 60-second batch window. The score will be cached in Redis with a 24-hour TTL. Let's continue with the data model."

**Expected behavior:** Tech Spec prompt pauses and asks whether these decisions have ADRs. If not, it redirects: the streaming-vs-batch trade-off and the Redis TTL choice are both architecturally significant (they affect downstream reliability, cost, and observability), and they belong in the *Open architectural questions* section as `Q-NNN` entries, not as fait accompli decisions in the Component breakdown. Suggests recording them via `adr.md` after the Draft is produced. Does not silently bake the decisions into the spec.

**Failure mode exercised:** decisions buried instead of surfaced — Tech Spec enforces the *surface decisions; do not bury them* rule.

### TS.2 — Retrofit project with no current-architecture summary

**Status:** untested

**Input:** Tech Spec prompt opened with: "We have an existing customer success dashboard built on Django + Postgres. We want to add a new at-risk-accounts page. Please produce the Tech Spec."

**Expected behavior:** Tech Spec prompt detects retrofit mode and pauses Phase 1. Demands a current-architecture summary before producing any design content: what does the existing dashboard's component breakdown look like, what's in the data model, where do new components plug in? Refuses to produce a Tech Spec that pretends the project is greenfield. Once the user provides the summary, the Tech Spec frames the new at-risk-accounts page as a delta against the existing system.

**Failure mode exercised:** retrofit-as-greenfield — Tech Spec enforces the *demand current-architecture summary on retrofits* rule.

### TS.3 — Mode detection from resolved ADRs in input

**Status:** untested

**Input:** Tech Spec prompt opened with a Tech Spec Draft pasted in plus three ADRs (`adr-001-real-time-vs-batch-scoring.md`, `adr-002-auth-model.md`, `adr-003-driver-storage.md`), each with `Status: accepted` and `Originating question-id` linking back to Q-001, Q-002, Q-003 from the Draft. User says: "Please produce the final Tech Spec."

**Expected behavior:** Tech Spec prompt detects Revision mode from the presence of resolved ADRs. Phase 1 announces *"Revision mode — consuming the Draft plus three resolved ADRs."* Produces a Final Tech Spec (`tech-spec-[project]-final.md` in the artifact filename) where every open architectural question is marked `Resolved (see ADR-NNN)` and the relevant sections (Component breakdown, Data model, etc.) are updated with the ADR decisions. The artifact header's *Stage* line reads `Tech Spec (Final)` and *Upstream artifacts* names the Draft plus the three ADRs.

**Failure mode exercised:** mode detection — Tech Spec correctly identifies Revision mode from the input shape rather than running Draft mode by default.

---

## Architecture Decision Record (`adr.md`)

### AD.1 — Multi-decision request

**Status:** untested

**Input:** ADR prompt opened with: "I need to record our decision to use Postgres as the primary database with Prisma as the ORM."

**Expected behavior:** ADR prompt pauses and names the two decisions: (1) Postgres as the primary database; (2) Prisma as the ORM. Asks which one to record first, and offers to chain a second ADR run for the other. Refuses to bundle both into a single record. Notes that bundled ADRs break the `Q-NNN` ↔ `Originating question-id` matching that Tech Spec Revision relies on for closing out open questions.

**Failure mode exercised:** multi-decision bundling — ADR enforces *one decision per ADR* and chains follow-up runs.

### AD.2 — Decision not yet made (decision-prep mode)

**Status:** untested

**Input:** ADR prompt opened with: "I want to record an ADR for the auth model. We're debating between session-based auth and JWT via Auth0. I haven't decided yet but want to capture the options."

**Expected behavior:** ADR prompt switches to decision-prep mode. Produces an ADR with Status `proposed`, populated Context (forces in tension, constraints) and Options considered (both alternatives with their respective trade-offs), but with Decision and Consequences sections left blank with explicit *"Decision deferred — see Options considered"* placeholders. The artifact header's Status line reads `proposed`. The orchestrator's ADR-loop completion check should treat this as not-yet-resolved (per `tech-spec.md`'s Revision-mode detection rule that `proposed` ADRs do not satisfy the loop's completion gate).

**Failure mode exercised:** decision-not-yet-made — ADR switches to decision-prep mode rather than fabricating a decision.

### AD.3 — Standalone ADR with no source Tech Spec question

**Status:** untested

**Input:** ADR prompt opened with: "Record an ADR for our decision to use feature flags for the at-risk-accounts page rollout. We're using LaunchDarkly."

**Expected behavior:** ADR prompt asks the user to confirm the architectural context — what forces are in tension, what alternatives were considered, what makes this decision architecturally significant rather than an implementation detail. Assigns `**Originating question-id:** user-supplied` in the artifact header. Marks the architectural-context input as `user-confirmed` in the Assumptions table. Once the user provides sufficient context, produces a full ADR. Does not produce a thin ADR with no forces named.

**Failure mode exercised:** standalone-ADR-with-no-question-frame — ADR enforces the *well-framed question* requirement even in standalone mode.

---

## Implementation Plan (`implementation-plan.md`)

### IP.1 — Phase with non-deterministic gate

**Status:** untested

**Input:** During work-breakdown elicitation: "Phase 1 is done when the team feels good about the ingest pipeline."

**Expected behavior:** Implementation Plan prompt pushes back. *"'Feels good' is not a deterministic gate — it cannot be checked objectively. What specific conditions would the team be looking at? Tests passing? A particular latency target? Security review sign-off?"* Helps the user articulate an objective gate criterion (e.g., *"all unit and integration tests pass on the phase branch AND security review for the survey-data access path recorded by named approver"*). Refuses to ship the plan with the non-deterministic gate.

**Failure mode exercised:** non-deterministic gate — Implementation Plan enforces the *every gate criterion is deterministic* rule.

### IP.2 — Agent execution boundaries section omitted

**Status:** untested

**Input:** User to Implementation Plan prompt after producing the work breakdown, risk register, and test strategy: "Skip the Agent execution boundaries section — we're not using agents on this project. Just give me the plan."

**Expected behavior:** Implementation Plan prompt refuses. Explains that the section forces the team to make implicit autonomy assumptions explicit even when no agent is planned (e.g., *"who can run database migrations without sign-off"* is a meaningful question for human teams too). Offers to populate the table with team-execution language if the user prefers. Refuses to ship the plan without the section because `agents-md-generator.md` (and downstream automation if the project later adopts agents) cannot function without it.

**Failure mode exercised:** missing *Agent execution boundaries* — Implementation Plan enforces the required-section rule.

### IP.3 — Vague boundary entry

**Status:** untested

**Input:** During Agent execution boundaries elicitation: "Under `allowed without approval`, the agent can edit code and run tests."

**Expected behavior:** Implementation Plan prompt pushes for specific verbs and surfaces. *"'Edit code' is too broad — does it cover all files? Just source files? What about CI config? Test fixtures? The point of the table is to make boundaries auditable for the AGENTS.md generator. Examples of specific entries: 'edit files under `src/lib/` and `tests/`'; 'run the test suite via `npm test`'; 'update fixtures under `tests/fixtures/`.'"* Refuses to accept the vague version.

**Failure mode exercised:** vague boundary entry — Implementation Plan enforces the *verbs and surfaces* discipline.

---

## AGENTS.md Generator (`agents-md-generator.md`)

### AG.1 — Implementation Plan with empty Agent execution boundaries

**Status:** untested

**Input:** AGENTS.md generator opened with an Implementation Plan artifact whose *Agent execution boundaries* section reads only *"To be determined."*

**Expected behavior:** generator refuses to produce an AGENTS.md. Routes the user back to `implementation-plan.md` to populate the boundaries table with specific verbs and surfaces in each of the three classes (`allowed without approval` / `requires approval` / `prohibited`). Does not invent boundaries to fill the gap. The refusal message names the specific section that needs work.

**Failure mode exercised:** invented permission boundaries — generator enforces the *refuse to invent* rule and routes back to the upstream stage.

### AG.2 — Content that would exceed 200 lines

**Status:** untested

**Input:** AGENTS.md generator opened with a substantial Implementation Plan (12 phases, dense Agent execution boundaries with 8 entries per class) plus 7 resolved ADRs (each contributing counterintuitive conventions) plus a Tech Spec with rich tooling/versioning details. User: "Generate the AGENTS.md."

**Expected behavior:** generator estimates the file size and surfaces that the content would exceed ~200 lines. Pushes overflow content into named files: e.g., *"the 12-phase work breakdown's per-phase notes belong in `references/work-breakdown.md`; the 7 ADRs' counterintuitive-conventions detail belongs in `references/decisions.md`; the always-loaded `AGENTS.md` is the short summary plus pointers."* The cover note lists each deferred file with suggested contents. Does not produce a 300-line `AGENTS.md` with a "should be shorter" note.

**Failure mode exercised:** inflation past the 200-line ceiling — generator pushes overflow to `references/` rather than producing a too-long file.

### AG.3 — Project README already covers tooling

**Status:** untested

**Input:** AGENTS.md generator opened with full inputs plus the user's note: "Our project's README already has a 'Development setup' section with the tooling versions and install/test/build commands."

**Expected behavior:** generator pauses and asks which content is authoritative — the README or the AGENTS.md. Cites Lulla et al.'s finding (~3% task-success degradation, ~20% inference-cost penalty for redundancy). If the README is the source of truth, the generator's AGENTS.md uses pointers to the README sections rather than duplicating; if the AGENTS.md should be the source, the README's section should reference the AGENTS.md instead. Does not silently duplicate.

**Failure mode exercised:** README-duplication — generator enforces the *no duplication* rule and surfaces the source-of-truth question.

---

## Chain handoffs

These scenarios exercise the suite end-to-end across multiple stages. Each requires running two or more prompts in sequence and verifying the handoff works.

### CH.1 — Standalone PRD with no upstream BRD; business context supplied inline

**Status:** untested

**Input chain:** Run `prd.md` without an upstream BRD. The user pastes business context inline (the four-sentence business case from BR.1's hypothetical-good ending), then proceeds through the PRD elicitation.

**Expected behavior:**

1. PRD's Phase 1 confirms the BRD-equivalent content the user supplied and marks each piece (business outcomes, stakeholders, success criteria, constraints) as `user-confirmed` in a pre-emptive Assumptions table.
2. The PRD elicitation proceeds normally — personas, functional requirements, NFRs, out-of-scope, acceptance criteria.
3. The produced PRD artifact carries:
   - Artifact header with *Upstream artifacts: none (business context supplied inline)*
   - *Assumptions and inferred inputs* table at the top with rows for each user-confirmed input, each carrying a one-sentence note naming what the user supplied
   - Standard PRD section content otherwise

**Failure mode exercised:** standalone PRD run preserving chain integrity via the input-type taxonomy — no silent hallucination of upstream context; mid-chain runs remain auditable.

### CH.2 — Full ADR loop: Tech Spec Draft → 2 ADRs → Tech Spec Revision

**Status:** untested

**Input chain:**

1. Run `tech-spec.md` on a small PRD. The Draft surfaces two open architectural questions: Q-001 (real-time vs. batch scoring) and Q-002 (auth model).
2. Run `adr.md` twice. Each ADR addresses one question; each carries `**Originating question-id:** Q-001` and `**Originating question-id:** Q-002` respectively in its header.
3. Run `tech-spec.md` again with the Draft plus the two resolved ADRs pasted in.

**Expected behavior:**

- Stage 1: Tech Spec Draft produced with the two open questions surfaced as `Q-001` and `Q-002` entries.
- Stage 2: Two ADRs produced, each with `Status: accepted` (or `proposed`) and the `Originating question-id` line correctly tagged.
- Stage 3: Tech Spec prompt detects Revision mode from the input shape (resolved ADRs present). Produces the Final Tech Spec where the *Open architectural questions* section lists both questions as `Q-001 — Resolved (see ADR-001)` and `Q-002 — Resolved (see ADR-002)`. The relevant Tech Spec sections (Component breakdown, Data model, etc.) absorb the decisions from the ADRs.

**Failure mode exercised:** ADR-loop matching — `Q-NNN` ↔ `Originating question-id` plumbing works end-to-end across three stage runs.

### CH.3 — Override path: Tech Spec Revision with one unresolved question

**Status:** untested

**Input chain:**

1. Run `tech-spec.md` Draft surfacing three open questions Q-001, Q-002, Q-003.
2. Run `adr.md` twice — for Q-001 and Q-002 only. Q-003 remains unresolved (no ADR produced).
3. Run `tech-spec.md` Revision with the Draft plus the two ADRs. When the prompt refuses to advance (Q-003 still unresolved), invoke explicit override: *"Accept unresolved Tech Spec; carry Q-003 forward as Implementation Plan risk."*
4. Run `implementation-plan.md` against the override-stamped Final Tech Spec.

**Expected behavior:**

- Stage 3 (Tech Spec Revision): produces a Final Tech Spec where Q-001 and Q-002 are marked `Resolved (see ADR-001)` and `Resolved (see ADR-002)`, and Q-003 is marked `Unresolved — carried as Implementation Plan risk`. The artifact header's Stage line reads `Tech Spec (Final)` (not Draft). The *Assumptions and inferred inputs* table includes a row tagged `inferred (override)` naming the unresolved question.
- Stage 4 (Implementation Plan): the unresolved Q-003 carries into the Risk register as a named risk with the original Tech Spec question text preserved. The mitigation owner is identified; the rollout plan accounts for the unresolved decision.

**Failure mode exercised:** override path — explicit override is logged in the Assumptions table, the Final Tech Spec is stamped Final (not Draft), and the downstream Implementation Plan correctly inherits the unresolved question as a Risk register entry rather than dropping it.

---

## Triaging failures

When a scenario fails (the prompt does not produce the expected behavior):

1. **Confirm the failure is real** — run the scenario twice. Frontier models are non-deterministic; one failure may be a one-off.
2. **Classify** — is the failure in the *prompt* (the prompt needs a fix) or in the *scenario* (the scenario's expected behavior is too strict, or under-specifies what success looks like)?
3. **Localize** — which of the prompt's six sections owns the failure? Role / Knowledge & sources / How requests are handled / Output contract / Constraints / Guardrails. The C2 conventions audit notes that fixes should land in the responsible section, not stacked elsewhere.
4. **Update the prompt or the scenario**, set the scenario's status to `passing` once the fix is verified, and note the version of the prompt the fix landed in.
5. **Flag systemic patterns** — if the same failure mode shows up across multiple prompts (e.g., guardrails being bypassed when the user is forceful), surface it in the ticket file's working scaffold for future C1 / C2 spec amendments.

The scenarios in this catalog are the initial minimal set. Expanding to four-or-more scenarios per prompt, adding adversarial inputs, and adding scenarios for the Integrated-mode orchestrator path are normal maintenance work; add new scenarios inline and bump the catalog's coverage as they land. No separate ticket is required.
