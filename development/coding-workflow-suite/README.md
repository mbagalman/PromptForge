# Coding Workflow Suite

An end-to-end prompt sequence that forces disciplined engineering process *before* any code is written: BRD → PRD → Tech Spec → ADR → Implementation Plan, driven by a workflow orchestrator, with an optional AGENTS.md / CLAUDE.md / SKILL.md generator that translates the planning artifacts into a runtime instruction file for an autonomous coding agent. Each stage produces a reviewable markdown artifact that is also the canonical input to the next stage — the chain works because every handoff is a parseable document, not a verbal recap.

The suite is distinct from the existing `coding/` directory, which ships per-language code-quality tools (`python-hardener.md`, `sql-optimization-engineer.md`) that operate during or after implementation. It is also distinct from `guides/agents-md-best-practices-2026.md`, which is reference material for writing AGENTS.md files; `agents-md-generator.md` is the prompt that applies that guidance to a project's Implementation Plan and ADRs.

## ⚠️ Status: experimental

Every prompt in this suite ships at `experimental` status — the first folder in the repo to do so. Every other folder (`data-analysis/`, `legal/`, `coding/`, `financial/`, `research/`, `meta-prompts/`) shipped at `stable` because its prompts had been used before publication. The workflow suite is novel and has not been exercised at scale on real chains. The controlled vocabulary supports `experimental`; the promotion to `stable` happens after field use exposes the chain's failure modes and they are addressed.

Treat outputs as scaffolding to interrogate, not specifications to ship. The chain's discipline (input-type taxonomy, ADR loop, deterministic approval gates) is the durable contribution; the per-prompt elicitation rules will shift as the suite is used.

## Files

The suite ships with seven prompts:

- **[workflow-orchestrator.md](workflow-orchestrator.md)** — the entry point. Asks the size-scaling question, recommends a chain, and drives the workflow in either Router mode (tells the user which stage prompt to run next) or Integrated mode (produces the next stage's artifact inline by applying that stage's rules).
- **[brd.md](brd.md)** — Business Requirements Document elicitation. Captures the business case (situation, stakeholders, outcomes, success criteria, constraints, non-goals) before any product or technical work begins.
- **[prd.md](prd.md)** — Product Requirements Document elicitation. Translates the BRD's *why* into a user-facing *what*: personas, functional and non-functional requirements, out-of-scope, acceptance criteria.
- **[tech-spec.md](tech-spec.md)** — Technical Specifications with Draft and Revision modes. Draft mode produces a first-pass system design and surfaces decisions that warrant ADRs; Revision mode consumes the Draft plus resolved ADRs and produces a Final Tech Spec.
- **[adr.md](adr.md)** — Architecture Decision Record in the canonical Nygard format (Title / Status / Context / Decision / Consequences). One decision per run; the `question_id` tag links each ADR back to the Tech Spec Draft open question that triggered it.
- **[implementation-plan.md](implementation-plan.md)** — Phased work breakdown with deterministic approval gates between phases, confidence-banded estimates, a risk register, a rollout plan, and an *Agent execution boundaries* table classifying actions as `allowed without approval` / `requires approval` / `prohibited`.
- **[agents-md-generator.md](agents-md-generator.md)** — Optional final stage. Translates the Implementation Plan's *Agent execution boundaries* and the resolved ADRs into an AGENTS.md / CLAUDE.md / SKILL.md instruction file for the user's project repo. Enforces the ~200-line ceiling, trust-boundary discipline, and the no-duplication-with-README rule from `guides/agents-md-best-practices-2026.md`.

## Inputs

Each prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level:

- **`workflow-orchestrator.md`** — the user's overall project intent, target timeline, project size (small / medium / large), greenfield-or-retrofit, and mode preference (Router / Integrated).
- **`brd.md`** — the business problem or opportunity in the user's own words, plus optional stakeholder list, business constraints, and timeline.
- **`prd.md`** — the BRD artifact (or equivalent business context inline), target user personas, and product scope.
- **`tech-spec.md`** — the PRD artifact (or equivalent product context inline), known technical constraints (existing stack, team capabilities, deployment environment), non-functional targets. In Revision mode, additionally the resolved ADRs.
- **`adr.md`** — the decision to be recorded (named explicitly), the architectural context (typically one open question from a Tech Spec Draft), and candidate options. Optionally the originating `question_id`.
- **`implementation-plan.md`** — the Tech Spec (Final) artifact, the relevant resolved ADRs, team capacity or timeline, and a working definition of done.
- **`agents-md-generator.md`** — the Implementation Plan artifact (with the *Agent execution boundaries* table populated), the resolved ADRs, optionally the Tech Spec (Final) for exact tooling and versioning, the target agent-file format (`AGENTS.md` / `CLAUDE.md` / `SKILL.md`), and any project metadata not in upstream artifacts (e.g., exact CI commands).

Every stage marks every input as `supplied` (came from an upstream artifact), `inferred` (filled in because upstream was missing or partial), or `user-confirmed` (provided inline by the user, replacing upstream). The marking is surfaced in each produced artifact's *Assumptions and inferred inputs* section. This is the chain's audit discipline — it makes mid-chain runs and skipped stages legible rather than silent.

## How to Use

Paste the full text of a prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the required input in your first message.

**The recommended entry point is `workflow-orchestrator.md`.** The orchestrator asks the size-scaling question on the first turn, recommends a chain (Small / Medium / Large — see *Workflow* below), and tells you which stage to run next (Router mode) or produces the artifact inline by applying that stage's rules (Integrated mode). Router mode is the default and the lower-friction starting point; Integrated mode trades convenience for the requirement that the orchestrator's encoding of each stage stays in sync with the stage prompt itself.

Each stage prompt also runs standalone. A user with a working BRD in hand can run `prd.md` directly; a user with a Tech Spec Draft and one open architectural question can run `adr.md` once per question. The input-type taxonomy handles the missing-upstream case — the stage marks the missing inputs as `inferred` or `user-confirmed` and surfaces them in the artifact's Assumptions table, rather than silently hallucinating upstream context.

## Workflow

This section is load-bearing. The suite encodes a multi-shape state machine: stages are states, artifacts are state values, handoffs are transitions, approval gates are guarded edges, and the ADR step is a guarded loop.

### Size-scaling chains

The orchestrator's first turn asks about project size and recommends a chain. The chain shape is the most consequential decision in the workflow — running the full Large chain on a one-week spike is overhead; running the Small chain on a six-month platform initiative is malpractice.

| Size | Description | Chain |
|---|---|---|
| **Small** | A spike, a small internal tool, a self-contained 1–4 week project | `prd.md` → `implementation-plan.md` |
| **Medium** | A 2–3 month team initiative, well-bounded scope, modest architectural novelty | `prd.md` → `tech-spec.md` (Draft, possibly Final via ADR loop) → `implementation-plan.md` |
| **Large** | A 6+ month platform initiative, multiple stakeholders, significant architectural novelty | `brd.md` → `prd.md` → `tech-spec.md` (Draft) → `adr.md` loop (if open questions exist) → `tech-spec.md` (Revision) → `implementation-plan.md` |

Each chain may end with the optional AGENTS.md generation stage (`agents-md-generator.md`). The orchestrator complies with user overrides in either direction — "Small but I want a BRD anyway" or "Large but skip the BRD because we already have one" — and logs the override in the next downstream stage's Assumptions table.

### The ADR loop

The most subtle shape in the chain is the iteration between the Tech Spec and the ADR. The flow is:

1. **Tech Spec Draft** lists *Open architectural questions* — decisions with architectural weight (data-store choice, sync vs. async, service split, framework choice, schema evolution, build vs. buy) that the spec deliberately does *not* decide silently. Each question gets a `Q-NNN` id.
2. **`adr.md`** runs once per open question. Each ADR consumes one question, captures Context / Decision / Consequences in the canonical Nygard format, tags its header with the originating `question_id`, and resolves the question with one of the canonical statuses (`accepted`, `proposed`, `superseded by ADR-MMM`, `deprecated`).
3. **Tech Spec Revision** consumes the Draft plus the resolved ADRs and produces a Final Tech Spec where every open question is either marked `Resolved (see ADR-NNN)` or carried forward under explicit override as an Implementation Plan risk.
4. **Implementation Plan** runs against the Final Tech Spec (open questions empty or carried under override).

The loop exists so the Implementation Plan never runs against a Tech Spec with unresolved architectural decisions baked in implicitly. When the Draft has no open architectural questions, the Draft itself serves as the Final and the loop is skipped.

### Orchestrator modes — Router and Integrated

The orchestrator runs in one of two modes, picked in Phase 1 and switchable mid-workflow:

- **Router mode (default).** The orchestrator names the next stage's prompt, tells the user what to paste from the prior stage's artifact, and waits for the user to return with the produced artifact. The user runs each stage prompt themselves in a separate session. Lower-friction handoff; the orchestrator and the stage prompts have clear separation of concerns; failures are easier to localize.
- **Integrated mode.** The orchestrator produces the next stage's artifact inline by applying that stage prompt's full rules (Knowledge & sources, How requests are handled, Output contract, Constraints, Guardrails) in its own conversation. The user does not switch tools between stages. Higher convenience; the orchestrator must internalize every stage prompt's contract, and any drift between the orchestrator's encoding and the stage prompt itself is a defect — the stage prompt wins. Every Integrated-mode artifact opens with a header announcing which stage's rules were applied so the output is auditable.

The default for first-time users is Router. Integrated is opt-in.

### Input-type taxonomy

Every stage marks every input with one of three values:

- **`supplied`** — the input came from the upstream stage's artifact pasted into this session.
- **`inferred`** — the input was filled in because the upstream artifact was missing or partial. Required to carry a one-sentence note explaining what was inferred and why.
- **`user-confirmed`** — the input was provided inline by the user, replacing what would normally come from upstream. Required to carry a one-sentence note naming what the user supplied.

Every produced artifact carries an `## Assumptions and inferred inputs` table that records the markings. `supplied` rows may be elided when the upstream artifact is named in the header's *Upstream artifacts* line; `inferred` and `user-confirmed` rows are always required.

The taxonomy is what allows mid-chain and standalone runs without sacrificing the chain's integrity. A PRD run with no BRD upstream is allowed — it just marks every BRD-derived input as `user-confirmed` (when the user provided business context inline) or `inferred` (when the prompt filled the gap). Silent hallucination of upstream context is the failure mode the taxonomy exists to prevent.

### Chain diagram

```
                  ┌──────┐
                  │ INIT │  (orchestrator Phase 1: size, mode, greenfield/retrofit)
                  └──┬───┘
                     │
                     ▼
            size selection
            ┌─────────┴──────────┐
            │                    │
        Small/Medium          Large
            │                    │
            │                    ▼
            │              ┌─────────┐
            │              │  brd.md │
            │              └────┬────┘
            │                   │  gate: stakeholders + outcomes signed off
            ▼                   ▼
            ┌─── prd.md ────────┘
            │
   ┌────────┴─────────┐
   │ Small            │ Medium/Large
   │                  ▼
   │           ┌──────────────┐
   │           │ tech-spec.md │  (Draft)
   │           └──────┬───────┘
   │                  │
   │         open_questions == 0 ?
   │            │            │
   │           yes           no
   │            │            ▼
   │            │     ┌─────────────┐
   │            │     │ adr.md loop │  (one ADR per open question)
   │            │     └──────┬──────┘
   │            │            │  gate: every question has a resolved ADR
   │            │            ▼
   │            │     ┌──────────────┐
   │            │     │ tech-spec.md │  (Revision → Final)
   │            │     └──────┬───────┘
   │            │            │
   │            └────────────┘
   │                  │  gate: tech-spec finalized
   ▼                  ▼
   └─── implementation-plan.md
                  │
                  ▼
           (optional) agents-md-generator.md
                  │
                  ▼
              COMPLETE
```

### Compact worked example

A small but realistic example to make the chain shape concrete. The hypothetical project is an **internal NPS-tracking tool** for a SaaS company: a dashboard that ingests survey responses, computes per-segment NPS, and surfaces detractor accounts to Customer Success managers so they can intervene before churn. It is large enough to exercise the full chain (BRD through Implementation Plan) and small enough that each stage's role stays legible in a paragraph.

**BRD.** The session opens with the business situation: detractor responses are landing in a survey tool no one reads, the CS team finds out about at-risk accounts on average two weeks after the response is filed, and the company estimates $1.4M in unintervened churn last year that was visible in survey data the day it arrived. The primary decision-maker is the VP of Customer Success; sign-off owners are the CFO (for instrumentation cost) and the Data team lead (for survey-data access). The business outcomes are *reduce time-to-intervention from 14 days to 5 days within two quarters of launch* and *prevent at least 30% of the historically-unintervened-churn dollar value within the same window*. Constraints: SOC 2 Type II posture must be preserved; the team must use the existing Snowflake warehouse; no new vendor relationships. Non-goals: this is not a replacement for the survey-collection tool itself, and expansion-opportunity scoring is explicitly out.

**PRD.** The PRD inherits the business outcomes and stakeholders verbatim and translates them into product behavior. Personas: the primary persona is the Customer Success Manager (uses the tool daily; views detractor accounts sorted by ARR descending; drills into per-account risk drivers); the secondary persona is the VP of Customer Success (views aggregate trend tiles weekly). Functional requirements include *FR-1.1 CS Manager can view a list of at-risk accounts (NPS ≤ 6 in the last 30 days) sorted by ARR descending*, with the acceptance criterion *list refreshes within 2 minutes of a new survey response landing*; and *FR-2.1 CS Manager can drill into per-account risk drivers shown in plain language, not raw model features*. Non-functional requirements: P95 page load under 2 seconds for accounts ≤ 5,000; SOC 2 audit obligations inherited from the BRD; WCAG 2.1 AA. Out-of-scope: predictive expansion-opportunity scoring; the mobile experience for v1.

**Tech Spec (Draft).** The Tech Spec problem statement re-states the engineering job: ingest survey responses from the survey tool's webhook into Snowflake, compute per-account NPS on a refresh cadence that satisfies the 2-minute acceptance criterion, and surface detractor accounts via a thin React frontend backed by a small read API. The component breakdown names three components (ingest worker, scoring service, frontend) with their owned data and dependencies; the data model names the `survey_response`, `account`, and `nps_score` entities. The Tech Spec surfaces three *Open architectural questions*: **Q-001 — Real-time vs. batch scoring** (how to satisfy the 2-minute refresh acceptance criterion within SOC 2 constraints), **Q-002 — Auth model for the frontend** (session-based vs. JWT given the existing stack uses Auth0), and **Q-003 — Storage of detractor drivers** (raw survey text in Snowflake vs. a vector store given the "plain language" requirement). The Draft does not pick; it surfaces. The closing message tells the user to run `adr.md` once per question, then return for Revision.

**ADR loop.** Three ADRs run. *ADR-001 (real-time vs. batch scoring)* records the decision to use a streaming ingest with a 60-second batch window, citing forces in tension: the 2-minute acceptance criterion (favors streaming), the existing Snowflake-centric stack (favors batch), and the SOC 2 audit posture (favors a single auditable path). Type 2 — reversible if the cost curve changes. *ADR-002 (auth model)* records JWT via Auth0 — the existing stack constrains the choice. Type 1 — switching auth models post-launch is a multi-team coordination cost. *ADR-003 (driver storage)* records keeping raw survey text in Snowflake with a lightweight text-summarization step at read time, deferring a vector store until the "plain language" requirement proves inadequate. Type 2. Each ADR's `Originating question-id` line links it back to the Tech Spec Draft.

**Tech Spec (Revision).** The Revision consumes the Draft plus the three ADRs and produces the Final Tech Spec. The component breakdown is updated — the scoring service now explicitly carries the 60-second batch window from ADR-001; the frontend's auth section names JWT-via-Auth0 from ADR-002; the data model section absorbs the read-time-summarization approach from ADR-003. The *Open architectural questions* section now lists each prior entry as `Q-NNN — Resolved (see ADR-NNN)`. The closing message advances the user to `implementation-plan.md`.

**Implementation Plan.** The plan breaks the work into four phases: (1) ingest pipeline + Snowflake schema, gated by *all unit and integration tests pass on the phase branch* and *security review for the survey-data access path recorded by named approver*; (2) scoring service + 60-second batch window, gated by *p95 score-write-to-read-latency under 90 seconds on staging for 48 hours*; (3) frontend MVP + Auth0 integration, gated by *all WCAG 2.1 AA checks pass via axe-core* and *the auth flow integration test suite passes against staging Auth0*; (4) rollout, gated by *feature flag toggled on in staging for 5 business days with no observability regressions* and *VP of CS sign-off recorded*. Estimated effort is banded per phase (Phase 1: 5–8 engineering-days, P50 = 6; Phase 4: 3–5 days). The *Agent execution boundaries* table classifies actions: `allowed without approval` includes running unit tests, editing under `src/lib/` and `tests/`, and running the linter; `requires approval` includes schema migrations against any environment and major-version dependency upgrades; `prohibited` includes force-pushing to `main`, modifying CI workflows, and writing survey response PII to logs. The risk register inherits the three ADRs' Type 1 / Type 2 markings as execution-time risks.

**A note on the two orchestrator modes.** Both modes would produce the same six artifacts. The difference is in the handoff: in **Router mode**, the orchestrator's turn between Tech Spec Draft and ADR runs would read something like *"Tech Spec Draft is done with three open questions (Q-001, Q-002, Q-003). Open a new session with `adr.md` and paste Q-001 into the first turn. Come back with the resolved ADR and we'll do the same for Q-002."* In **Integrated mode**, the orchestrator would produce ADR-001 inline in the same conversation, with a header announcing *"This artifact was produced by applying `adr.md`'s rules in Integrated mode."* The user does not switch tools, but the orchestrator carries the burden of staying in sync with `adr.md`'s contract.

### Full worked example

The compact example above shows the chain shape in paragraphs. The full artifact-by-artifact rendering of the same project — all nine files the suite would produce, totaling about 1,250 lines — lives in [examples/nps-tracking-tool/](examples/nps-tracking-tool/README.md). Read the [examples README](examples/nps-tracking-tool/README.md) first; it lists the reading order, illustrates a Router-mode orchestrator handoff and the same handoff in Integrated mode, and includes the AGENTS.md generator's cover note (which lives in the conversation, not in the committed AGENTS.md file itself).

## Platform setup

All prompts paste into the assistant builder's instructions field. None of the six prompts requires code execution, file creation, web search, or specific knowledge files — they are pure elicitation and document-production tools operating on text inputs and producing markdown outputs.

| Prompt | Code execution | File creation | Web search | Knowledge files |
| --- | --- | --- | --- | --- |
| [workflow-orchestrator.md](workflow-orchestrator.md) | Not needed | Not needed | Off | Optional — in Integrated mode, project knowledge containing the other five stage prompts helps the orchestrator stay in sync with each stage's contract |
| [brd.md](brd.md) | Not needed | Not needed | Off | None |
| [prd.md](prd.md) | Not needed | Not needed | Off | None |
| [tech-spec.md](tech-spec.md) | Not needed | Not needed | Off | None |
| [adr.md](adr.md) | Not needed | Not needed | Off | None |
| [implementation-plan.md](implementation-plan.md) | Not needed | Not needed | Off | None |
| [agents-md-generator.md](agents-md-generator.md) | Not needed | Not needed | Off | Recommended — `guides/agents-md-best-practices-2026.md` should be in project knowledge so the generator can cite its short-name principles in the cover note |

**Claude Projects:** leave Code Execution and File Creation off for all six prompts. For the orchestrator in Integrated mode, optionally upload the other five stage prompts to project knowledge — Claude pulls from project knowledge ambiently, which reduces the drift risk between the orchestrator's encoding of each stage and the stage prompt itself.

**OpenAI Custom GPTs:** leave Code Interpreter & Data Analysis off for all six prompts. For the orchestrator in Integrated mode, the same five stage prompts can be uploaded to Knowledge; Custom GPTs do not treat knowledge as ambient, so the orchestrator's prompt already names which stage's rules it is applying on each turn.

**Gemini Gems:** confirm code execution is off. The orchestrator in Integrated mode benefits from the stage prompts being attached as knowledge files for the same reason.

### Suggested conversation starters

- **Workflow Orchestrator:** "I want to build [project] — help me through the workflow." / "I have an idea I'd like to scope properly before writing code." / "Walk me through what I should produce before this work starts."
- **BRD:** "I need a BRD for [project]." / "Help me articulate the business case for [initiative]." / "We're proposing [work] — let's settle the why before the what."
- **PRD:** "I need a PRD for [project] — here's the BRD." / "Translate this business case into product requirements." / "Help me scope the product behavior for [capability]."
- **Tech Spec:** "I have a PRD for [project] — let's spec the system." / "Help me draft the technical spec for [system]." / "I have a Tech Spec Draft and three resolved ADRs — let's do the Revision pass."
- **ADR:** "Help me record an ADR for [decision]." / "I have an open architectural question from a Tech Spec Draft — let's resolve it." / "I'm still exploring options on [decision] — produce a decision-prep ADR."
- **Implementation Plan:** "Build the implementation plan for [project] — here's the Tech Spec and the ADRs." / "Help me phase this work with proper approval gates." / "I need the Agent execution boundaries table populated for [project]."
- **AGENTS.md Generator:** "Generate an AGENTS.md from this Implementation Plan and these ADRs." / "Produce a CLAUDE.md for [project] — target ≤200 lines." / "Translate this Implementation Plan's Agent execution boundaries into a SKILL.md."

## Maintenance notes

Guidance for anyone editing the prompts. Each prompt has load-bearing structure that should not be flattened in pursuit of brevity. The chain has more handoff seams than other folders in this repo, so the regression-test discipline is more rigorous than usual.

The concrete regression-scenario catalog (three per prompt plus three chain-handoff scenarios — twenty-four total) lives in [tests/regression-scenarios.md](tests/regression-scenarios.md). Each scenario states the input the user would paste, the expected behavior, and the named failure mode being exercised. The maintenance notes below describe the *categories* of failure each prompt is most prone to; the scenarios file is the executable rehearsal.

### Workflow Orchestrator

- **Most likely to drift:** the Router / Integrated mode separation and the *Priority when rules conflict* block. Pressure to be helpful pushes the orchestrator into producing artifacts in Router mode (becoming a stealth Integrated-mode session); pressure to be efficient pushes Integrated mode into paraphrasing stage contracts instead of applying them. Reinforce inside the Operating discipline block rather than scattering new rules across the phases.
- **Coupled sections:** the orchestrator's encoding of each stage's contract (in Integrated mode) is coupled to every other stage prompt in the suite. If any C3–C7 stage prompt's Output contract changes, the orchestrator must be re-validated against the change. The Integrated-mode disclosure header is the audit hook for this drift.
- **Regression tests:** small-project size scaling (BRD and ADR dropped from the recommended chain); user starts at Implementation Plan with no upstream (orchestrator demands the missing inputs); user skips a required stage (acknowledgement + assumption logging in the next stage); mode switch mid-workflow (orchestrator does not lose state); Integrated mode producing a stage whose underlying prompt has drifted (orchestrator either re-validates or refuses).
- **Do not flatten:** the size-selection logic in Phase 1, the Router / Integrated mode distinction, the *Priority when rules conflict* block, or the gate-enforcement discipline. The orchestrator is the canonical gate enforcer per the suite's design; stripping that role breaks the chain's discipline.

### BRD

- **Most likely to drift:** the *Outcome first, solution later* rule and the *One measurable success criterion is the floor* rule. Both erode under user pressure to "just write the BRD" without doing the elicitation work. Reinforce inside the Operating discipline block rather than adding new sections.
- **Coupled sections:** the *Output contract* sections (Executive summary / Business context / Stakeholders / Business outcomes / Success criteria / Constraints / Non-goals / Risks and dependencies / Open questions) map directly onto the PRD's expected inheritance per the suite's transition table. If the BRD's section names change, `prd.md`'s *Where this stage's inputs come from* table must be updated to match.
- **Regression tests:** vague business problem with no situational detail (verify Phase 1 pushes for specifics, scope-stops if the user cannot ground the situation); user proposes a solution instead of an outcome (verify redirect, no slide into feature spec); user cannot articulate any measurable success criterion (verify scope-stop close, no fabricated criterion); user names no primary decision-maker (verify scope-stop, no invented stakeholder); user describes a retrofit (verify the BRD frames the new initiative, not the existing system).
- **Do not flatten:** the five-exchange elicitation flow, the BRD–PRD–Tech Spec boundary examples in *Constraints*, or the scope-stop guardrails. The value is the discipline of producing a defensible business case; a BRD that hand-waves outcomes or stakeholders defeats every downstream artifact.

### PRD

- **Most likely to drift:** the *Personas are named, never invented* rule and the *Every requirement has an acceptance criterion* rule. Users will push for the prompt to synthesize personas from stakeholder language and for "the system shall" requirements without measurable checks. Reinforce inside Operating discipline.
- **Coupled sections:** the BRD-to-PRD inheritance table in *Knowledge & sources* is coupled to the BRD's *Output contract* section names. If the BRD's sections rename, this table must too. The PRD's *Functional requirements* and *Non-functional requirements* sections are likewise coupled to the Tech Spec's *Component breakdown driver* and *Reliability targets + Security considerations + Observability* inheritance.
- **Regression tests:** implementation creep (user wants the prompt to pick a database, framework, or API shape — verify redirect, file in *Open product questions* as Tech Spec input); missing-BRD mode (verify business-context inputs marked `user-confirmed` or `inferred`, surfaced in *Assumptions*); ambiguous or missing personas (verify the prompt asks, does not fabricate); scope drift into business-outcome language (verify redirect to BRD); "the system shall" phrasing (verify push for user-perspective rewrite).
- **Do not flatten:** the four-phase elicitation flow, the per-dimension *Non-functional requirements* discipline (each dimension either has requirements or an explicit "none required" line), the per-requirement acceptance criteria, or the existing-codebase redirect (PRD is for net-new initiatives).

### Tech Spec

- **Most likely to drift:** the *Surface decisions; do not bury them* rule and the *Refuse to produce a Final Tech Spec with unresolved open questions* rule. The first erodes when users want a confident-sounding spec without the ADR overhead; the second erodes when users want to "just move on" past unresolved questions. Both are headline disciplines.
- **Coupled sections:** the *Open architectural questions* section is coupled to `adr.md` via the `Q-NNN` question-id scheme — the ADR's `Originating question-id` header field matches the Tech Spec's Q-NNN values, and the Revision-mode resolution discipline depends on the match. If the question-id format changes, both prompts must change together. The mode-detection logic (Draft vs. Revision, triggered by the presence of resolved ADRs in the inputs) is coupled to the ADR's artifact-header convention.
- **Regression tests:** greenfield project (verify Draft mode proceeds without retrofit summary); retrofit project (verify Phase 1 demands current-architecture summary); missing PRD (verify offer of `user-confirmed` mode or pause-and-run-`prd.md` path); attempt to bake in an architectural decision without an ADR (verify redirect to *Open architectural questions*); mode-detection (Draft with no ADRs vs. Revision with ADRs); attempt to advance to Implementation Plan with non-empty open questions (verify refusal or explicit override).
- **Do not flatten:** the Draft / Revision mode distinction, the `Q-NNN` question-id scheme, the *Open architectural questions* section in both modes, or the two-distinct-files convention (`tech-spec-[project]-draft.md` and `tech-spec-[project]-final.md`). The ADR loop's traceability depends on both.

### ADR

- **Most likely to drift:** the *One decision per ADR* rule, the *At least two options* rule, and the *Reversibility must be named* (Type 1 / Type 2) rule. All three erode under "just write the ADR" pressure; all three are the discipline that distinguishes a useful ADR from decoration. Reinforce inside *Constraints*.
- **Coupled sections:** the `Originating question-id` header field is coupled to `tech-spec.md`'s Q-NNN scheme — the match is what allows the Tech Spec Revision pass to close out open questions programmatically. The Status vocabulary (`proposed` / `accepted` / `superseded by ADR-MMM` / `deprecated`) is coupled to the workflow orchestrator's ADR-loop completion check, which treats `proposed` as not-yet-resolved.
- **Regression tests:** multi-decision request (verify the prompt pauses, names each decision, asks which to record first); decision-not-yet-made (verify switch to decision-prep mode with Decision and Consequences blank, Status `proposed`); irreversible-decision flagging (verify Type 1 ADR's Negative subsection names the cost of reversal); superseded-ADR chaining (verify *Related ADRs* names the superseded ADR with a reason); ADR run with no source Tech Spec question — standalone mode (verify the prompt confirms the question is well-framed before producing); context recycled from upstream artifacts with no architectural force named (verify push for the forces in tension).
- **Do not flatten:** the canonical Nygard five-section shape (Title / Status / Context / Decision / Consequences), the `Q-NNN` question-id linkage, the Status vocabulary, the Type 1 / Type 2 reversibility marking, or the decision-prep mode. Each is non-negotiable for the ADR's role as a durable decision record.

### Implementation Plan

- **Most likely to drift:** the *Every gate criterion is deterministic* rule, the *No bare estimates* (confidence-bands-required) rule, and the *Agent execution boundaries is required* rule. The first erodes under "looks good"-style gates; the second under point-estimate pressure; the third under "we're not using agents on this project" pressure (which misses that the section forces the team to make implicit autonomy assumptions explicit even when no agent is planned).
- **Coupled sections:** the *Agent execution boundaries* table is what `agents-md-generator.md` consumes verbatim — if the table's three-column shape (Action class / Examples / Rationale) changes, the generator must be re-validated. The plan's *Work breakdown* and *Risk register* are coupled to the Tech Spec's *Component breakdown*, *Failure modes*, and *Security considerations*; if the Tech Spec changes the section names, the inheritance must be updated. The plan supports the C1 override path where unresolved architectural questions from the Tech Spec are carried as Risk register entries.
- **Regression tests:** phase without an approval gate (verify refusal); phase with a non-deterministic gate (verify push for objective form); bare point estimate (verify push for confidence band); missing-ADR mode (verify `inferred` marking + risk-register entry); *Agent execution boundaries* section omitted (verify refusal); vague boundary entry like "can make code changes" (verify push for verbs and surfaces).
- **Do not flatten:** the five-exchange flow, the three-class *Agent execution boundaries* table, the deterministic-gate discipline, the confidence-banded estimate discipline, or the Tech-Spec-and-ADRs-are-authoritative rule. The plan is the contract between the design upstream and the execution downstream; sloppiness here propagates into both the agent's runtime instructions and the team's execution velocity.

### AGENTS.md Generator

- **Most likely to drift:** the *~200-line ceiling* rule, the *No invented permission boundaries* rule, and the *No duplication of the project's README* rule. The first erodes when users want everything-in-one-file; the second when the Implementation Plan's *Agent execution boundaries* table is vague or missing and the generator is tempted to "fill in reasonable defaults"; the third when generation is run against a project whose README already covers tooling and commands. Reinforce inside *Constraints* and the pre-deploy checklist.
- **Coupled sections:** the generator's *Permission boundaries* output is coupled to `implementation-plan.md`'s *Agent execution boundaries* table — if the three-class table shape changes in the plan, the generator must be re-validated. The generator's *Counterintuitive conventions* section is coupled to `adr.md`'s Consequences and Decision sections — if the ADR format changes, the generator's extraction logic must match. The generator's authoritative reference is `guides/agents-md-best-practices-2026.md` — if the guide's four content categories or pre-deploy checklist change, the generator must be re-validated against the new version.
- **Regression tests:** Implementation Plan that named no permission boundaries (verify refusal and route back to `implementation-plan.md`); inflation attempt — content that would push the file past 200 lines (verify push to `skills/` or `references/` with explicit suggested files); README-duplication attempt — project has a README that already covers tooling (verify the prompt asks which is authoritative); conflicting upstream artifacts — Tech Spec says one tooling version, Implementation Plan implies another (verify the prompt surfaces the conflict rather than silently picking); secrets in upstream (verify refusal); target-format ambiguity (verify the prompt asks AGENTS.md vs. CLAUDE.md vs. SKILL.md before producing); vague boundary entry that slipped past the Implementation Plan's guardrails (verify the generator catches it).
- **Do not flatten:** the two-part Output contract (Cover note + fenced generated file), the eight-item pre-deploy checklist, the four-content-categories framework from the guide, the trust-boundaries discipline (trusted vs. untrusted inputs), or the no-frontmatter-inside-the-generated-file rule. The generated file follows AGENTS.md ecosystem conventions, not the suite's internal artifact conventions; conflating the two undermines both.

## License

Distributed under the MIT License. See [../../LICENSE](../../LICENSE) for details.
