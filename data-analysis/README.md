# Data Analysis Prompts

System prompts for working with tabular data and communicating the results: KPI design, exploratory analysis, predictive modeling, audit of modeling work, executive translation of findings, and dashboard design. The analysis prompts are designed to chain — a data analyst report or a modeling-engineer report can be handed to the auditor, and any of those can be handed to the executive translator — but each works standalone. The measurement-design prompts (KPI architect and dashboard designer) are independent of the analysis chain and sit upstream of it: defining what to measure and how to display it before the analyst or modeler runs.

## ⚠️ Not a substitute for domain expertise

These prompts are configured for tabular data only and assume the user can validate the model or analysis before acting on it. Outputs can be wrong; metrics can mislead; "the data shows" is not "the world is." For decisions with real stakes, treat outputs as analysis to interrogate, not conclusions to ship.

## Files

### Analysis

- **[data-analyst.md](data-analyst.md):** Autonomous exploratory analysis of an uploaded tabular file. Produces a structured report (TL;DR → What's in the data → Findings → What I'd look at next → Caveats) with charts. Statistically careful; refuses to manufacture insight when the data is thin.
- **[predictive-modeling.md](predictive-modeling.md):** Builds, evaluates, and delivers a predictive model on an uploaded tabular file. Defaults to XGBoost; switches to a regularized linear model on small data. Produces a fitted sklearn `Pipeline` artifact, a `MODEL_CARD.md`, and a report (TL;DR → Setup → Performance → What the model learned → Caveats → Files).
- **[predictive-model-auditor.md](predictive-model-auditor.md):** Reads a model report skeptically and produces a short audit memo with one of three verdicts: ready to use, has fixable issues, or should not be used as reported. Cannot run code by design — the spec depends on it.

### Measurement design and reporting

- **[kpi-architect.md](kpi-architect.md):** Interactive five-phase conversation (Scope → Objectives & framework → KPI selection → DuPont-style decomposition → Spec) that produces a written KPI specification with each KPI decomposed into the operational levers that mathematically drive it. Grounded in Kaplan & Norton's Balanced Scorecard (perspective coverage), Spitzer's criteria for effective measures (per-candidate filter), Grove's paired-metrics rule (gaming resistance), Marr's Key Performance Questions, and DuPont decomposition. Enforces a 3–7 KPI ceiling.
- **[executive-insights.md](executive-insights.md):** Translates technical analytical findings into executive decisions, risks, and quarter-ready actions for VP and C-suite stakeholders. Five-section output (Executive Summary → Strategic Alignment → Stakeholder Impact Matrix → Strategic Recommendations → Executive Talking Points) under a 400-word ceiling. Does not perform analysis; consumes analyst or modeling output.
- **[dashboard-designer.md](dashboard-designer.md):** Interactive five-phase design conversation (Scope → Metrics → Charts → Layout → Spec) that produces a tool-agnostic written specification a BI developer could implement in Tableau, Power BI, Looker, or comparable tools. Pairs with the dashboard design guide below.
- **[dashboard-design-guide.md](dashboard-design-guide.md):** Practitioner's reference on dashboard design — decision-first framing, metric selection, chart-chooser logic, layout, color and accessibility. Knowledge file consumed by the dashboard designer, not a system prompt.

## Inputs

Each prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: the analyst and modeling-engineer both take an uploaded tabular file (CSV, Excel, or similar) plus optional context; the auditor takes a written model report plus optional supporting artifacts (model card, plots, code excerpts); the executive-insights translator takes a core finding plus business context, evidence quality, decision objective, and constraints; the dashboard designer takes the user's intent for the dashboard plus optional persona, cadence, and tool details; the KPI architect takes the user's role or business area plus optional decision cadence, existing objectives, and industry context.

## How to Use

Paste the full text of a prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the required input in your first message. Specific code-execution, file-creation, and knowledge-file requirements differ across prompts — see *Platform setup* below.

## Workflow

The analysis prompts and the executive translator are designed to chain:

1. Run [data-analyst.md](data-analyst.md) on a dataset to understand what's there, or run [predictive-modeling.md](predictive-modeling.md) to build a model and produce a written report.
2. (Optional) Run [predictive-model-auditor.md](predictive-model-auditor.md) on the modeling-engineer's report to catch issues the modeler may have missed or downplayed. The modeler's output contract (TL;DR → Setup → Performance → What the model learned → Caveats → Files) maps directly onto the auditor's checklist.
3. (Optional) Run [executive-insights.md](executive-insights.md) on the analyst or audited modeling output to translate the findings into business implications, prioritized risks, and quarter-ready actions for VP / C-suite consumption.

Each prompt also works standalone — use [data-analyst.md](data-analyst.md) for analysis without a modeling step, [predictive-model-auditor.md](predictive-model-auditor.md) on a report produced outside this collection, or [executive-insights.md](executive-insights.md) on any analytical finding the user supplies inline.

[kpi-architect.md](kpi-architect.md) and [dashboard-designer.md](dashboard-designer.md) sit outside this chain. They are upstream measurement-design tools: use the KPI architect to decide *what* to measure (with each KPI decomposed into its operational levers), and the dashboard designer to decide *how to display* the resulting measures. A natural pairing is KPI architect → dashboard designer: the architect's spec names the metrics; the designer turns the top KPIs into a layout. Neither consumes the analyst's report.

## Platform setup

All prompts paste into the assistant builder's instruction field. Knowledge-file and tool-permission requirements differ:

| Prompt | Code execution | File creation | Web search | Knowledge files |
| --- | --- | --- | --- | --- |
| [data-analyst.md](data-analyst.md) | **Required** — every numeric claim must come from a computation | Required for charts | Off by default | None |
| [predictive-modeling.md](predictive-modeling.md) | **Required** — training, evaluation, serialization | **Required** — produces a joblib model + `MODEL_CARD.md` | Off by default | None |
| [predictive-model-auditor.md](predictive-model-auditor.md) | **Disabled** — the auditor reviews reports, it does not run analysis | Not needed (output is a short memo) | Off | None |
| [executive-insights.md](executive-insights.md) | Not needed | Not needed | Off | Optional: `stakeholder-map.md`, `business-context.md` for organization-specific grounding |
| [dashboard-designer.md](dashboard-designer.md) | Not needed | Not needed | Off | **Required:** [dashboard-design-guide.md](dashboard-design-guide.md) |
| [kpi-architect.md](kpi-architect.md) | Not needed | Not needed | Off | None — the five frameworks (Balanced Scorecard, Spitzer, Grove, Marr, DuPont) are named inline in the prompt |

**Claude Projects:** toggle Code Execution and File Creation per the table. For [dashboard-designer.md](dashboard-designer.md), upload `dashboard-design-guide.md` to project knowledge — Claude pulls from project knowledge ambiently. For [executive-insights.md](executive-insights.md), optionally upload organization-specific stakeholder and context files.

**OpenAI Custom GPTs:** enable or disable *Code Interpreter & Data Analysis* per the table. For [dashboard-designer.md](dashboard-designer.md), upload `dashboard-design-guide.md` to Knowledge — Custom GPTs do not treat knowledge as ambient, but the prompt's Knowledge & sources section already instructs the assistant to consult it.

**Gemini Gems:** confirm code execution is enabled (or disabled, for the auditor). Attach `dashboard-design-guide.md` for the dashboard designer; the prompt instructs the assistant to consult it explicitly.

### Suggested conversation starters

- **Data Analyst:** "Analyze this file." / "Why did [metric] change?" / "What's interesting in this data?" / "Is there a relationship between [A] and [B]?"
- **Predictive Modeling Engineer:** "Build a model on this file." / "Predict [target] from this." / "Is there a signal in this data?"
- **Predictive Model Auditor:** "Audit this model report." / "Is this model ready to use?" / "What's wrong with this writeup?" / "Review this before I sign off."
- **Executive Insights Partner:** "Translate this finding for the exec team." / "What's the business implication of this?" / "Turn this into a board update." / "What should leadership do about this?"
- **Dashboard Designer:** "I need a dashboard for [persona]." / "Help me scope a dashboard for [decision]." / "Review this dashboard mockup." / "Translate this report into a recurring dashboard."
- **KPI Architect:** "Help me pick KPIs for [role / team / business]." / "Decompose [KPI] into its drivers." / "What should we measure for [objective]?" / "Audit my current KPI list."

## Maintenance notes

Guidance for anyone editing the prompts. Each prompt has load-bearing structure that should not be flattened in pursuit of brevity.

### Data Analyst

- **Most likely to drift:** the *Statistical posture* section. If the assistant is being too aggressive about causal claims, too quick to call findings "significant," or too eager to chart everything, the fix lives there — add a one-line rule, do not stack new constraints in TL;DR or Findings.
- **Coupled sections:** the Output contract is the most platform- and audience-sensitive. If the audience changes (technical colleagues → non-technical stakeholders), revise the voice paragraph in Role and the TL;DR bullet rule together.
- **Regression tests:** keep a set of test files representing the failure modes the guardrails cover — an empty file, a non-tabular file, a 12-row file, a multi-sheet workbook, and one normal file with an accompanying question the data cannot answer. Each should hit the documented fallback.
- **Do not flatten:** the workflow phases (Orient → Audit → Explore → Interrogate → Synthesize) are load-bearing for analytical quality.

### Predictive Modeling Engineer

- **Most likely to drift:** the *When XGBoost is NOT the right choice* rules and the *Default splits* defaults. New data shapes (heavy time-series work, ranking problems, very small data) should be added there rather than scattering exceptions through the workflow phases.
- **Coupled sections:** the *Hand off* phase and the `MODEL_CARD.md` field list. If downstream consumers change (e.g., the model needs MLOps signatures or compliance fields), update both together.
- **Regression tests:** a 50-row file (forces linear model), a file with no obvious target, a 99.5%-one-class target, a multi-sheet workbook, a file with a leaky timestamp, and one normal file with a clean target.
- **Do not flatten:** the phase ordering (frame → split → audit → engineer → build → evaluate → interpret → hand off). The "split BEFORE engineering" rule and the "stop-and-reaudit on suspiciously high metrics" rule are the two most important guardrails in the spec.

### Predictive Model Auditor

- **Most likely to drift:** the verdict thresholds (checklist §9). Calibrate "ready to use" vs. "fixable issues" vs. "should not be used as reported" inside the verdict definitions, not in the concern bullets.
- **Pairs with the modeling-engineer spec.** A common drift mode is the two specs diverging on what counts as a complete report. If the modeler's output contract changes, update the auditor's checklist to match.
- **Regression tests:** a report with a random split on time-structured data, a report with AUC > 0.99 and no leakage audit, a report missing subgroup analysis, a report where the model card contradicts the writeup, and one clean report. The first four should produce specific findings; the clean one should produce the "ready to use" verdict in two sentences.
- **Do not flatten:** the three-verdict scheme. The temptation is to add intermediate verdicts ("ready with minor revisions," "ready conditionally"); resist it. The "cannot run code" constraint is also non-negotiable — it defines the assistant's role.

### Executive Insights Partner

- **Most likely to drift:** the 400-word ceiling and the "claim causality only when evidence supports it" rule. Both erode under pressure — users will ask for more detail, and findings will get presented with more confidence than they deserve. Reinforce inside the existing Constraints section rather than adding new sections.
- **Coupled sections:** the five output sections are tightly bound to the five analytical steps in *How requests are handled*. If a new output section is added (e.g., "Regulatory exposure"), add the corresponding analytical step.
- **Regression tests:** a correlational finding presented as causal (verify the prompt flags it), a finding with no business context (verify it asks rather than fabricating), an out-of-scope request to write SQL (verify it declines), a finding with no decision objective (verify it asks).
- **Do not flatten:** the five-section output contract or the under-400-word ceiling. The value is in the discipline of compression; if the output sprawls, the assistant has degraded into a generic synthesizer.

### Dashboard Designer

- **Most likely to drift:** the "decision-first" rule and the 5–9-metric ceiling. Users will push to skip Phase 1 and to add metrics in Phase 2; the prompt is designed to push back once and comply if pressed. If the assistant is complying too easily, tighten the operating discipline at the top of *How requests are handled*.
- **Coupled sections:** the chart-chooser logic in Phase 3 and the chart-chooser table in the design guide. If the guide's chooser changes, update the Phase 3 list in the prompt to match.
- **Pairs with the design guide.** The prompt cites principles by short name and assumes the guide is available as a knowledge file. If the guide is renamed or restructured, update the Knowledge & sources section and any short-name references.
- **Regression tests:** a request that names no persona (verify Phase 1 starts with the three-question opener), a request listing 15 metrics (verify the prompt cuts to 5–9), a request for "a pie chart with 12 slices" (verify the prompt pushes back), a request for DAX code (verify the prompt declines and refers to tool docs).
- **Do not flatten:** the five-phase conversation flow or the final Phase 5 specification template. The value is producing a written spec, not coaching the user through the design.

### KPI Architect

- **Most likely to drift:** the 3–7 KPI ceiling and the Mode A vs. Mode B distinction. Both erode under user pressure — pad-the-list pressure on the ceiling (now mitigated by the Candidate / monitoring metrics section), and "force a multiplicative identity on a survey-based KPI" pressure on the mode distinction. Both are headline disciplines; reinforce inside Constraints rather than adding new sections.
- **Coupled sections:** the framework list in *Knowledge & sources* and the framework choice in Phase 2. The Spitzer criteria checklist in Phase 3 mirrors the Knowledge & sources description; keep them aligned. The Metric taxonomy block (KPI / counter-metric / driver / diagnostic / candidate-monitoring) is referenced throughout the spec — if the taxonomy changes, audit every cross-reference. The per-KPI fields in the Output contract template (formula, unit, inclusion/exclusion, data source, refresh cadence) are the definition-hygiene contract; if a downstream consumer needs new fields (e.g., regulatory ID, system-of-record version), add them here.
- **Pairs with no knowledge file by design.** All five default frameworks and the domain-standard adaptations (Donabedian, Triple Aim, SRE SLOs, SCOR, logic models, etc.) are named inline so the prompt is self-contained across platforms (Custom GPTs and Gems do not treat knowledge as ambient). If the framework set grows substantially, externalize before the Knowledge & sources section becomes longer than the operational guidance.
- **Regression tests:** a request that names no scope of accountability (verify Phase 1 stops with a scope-clarification close, not a fabricated Phase 5 spec); a request listing 12 KPIs (verify the prompt cuts to 3–7 and files the overflow as candidate / monitoring metrics, or marks the spec "expanded KPI set" if the user explicitly overrides); a request to "decompose customer happiness into drivers" (verify the prompt switches to Mode B measurement model and labels it honestly); a request to build a Mode A decomposition for NPS or employee engagement (verify the prompt refuses to invent a multiplicative identity and switches to Mode B); a request to use lines-of-code as a productivity KPI (verify Grove's pairing rule fires with the gaming pattern named); a healthcare-quality request (verify Donabedian or Triple Aim is invoked and named); a request to design a dashboard (verify the prompt redirects to `dashboard-designer.md`).
- **Do not flatten:** the five-phase conversation flow, the Spitzer checklist, the Metric taxonomy block, or the Mode A / Mode B distinction. The DuPont mechanic (Mode A) is what differentiates a KPI tree from a brainstorm for quantitative outcome KPIs; the measurement model (Mode B) is what keeps the prompt honest about KPIs that have no arithmetic identity. Collapsing them into "decomposition" loses the epistemic distinction.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
