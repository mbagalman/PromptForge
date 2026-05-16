# Data Analysis Prompts

System prompts for working with tabular data: exploratory analysis, predictive modeling, and audit of modeling work. The three prompts are designed to chain — a data analyst report or a modeling-engineer report can be handed to the auditor — but each works standalone.

## ⚠️ Not a substitute for domain expertise

These prompts are configured for tabular data only and assume the user can validate the model or analysis before acting on it. Outputs can be wrong; metrics can mislead; "the data shows" is not "the world is." For decisions with real stakes, treat outputs as analysis to interrogate, not conclusions to ship.

## Files

- **[data-analyst.md](data-analyst.md):** Autonomous exploratory analysis of an uploaded tabular file. Produces a structured report (TL;DR → What's in the data → Findings → What I'd look at next → Caveats) with charts. Statistically careful; refuses to manufacture insight when the data is thin.
- **[predictive-modeling.md](predictive-modeling.md):** Builds, evaluates, and delivers a predictive model on an uploaded tabular file. Defaults to XGBoost; switches to a regularized linear model on small data. Produces a fitted sklearn `Pipeline` artifact, a `MODEL_CARD.md`, and a report (TL;DR → Setup → Performance → What the model learned → Caveats → Files).
- **[predictive-model-auditor.md](predictive-model-auditor.md):** Reads a model report skeptically and produces a short audit memo with one of three verdicts: ready to use, has fixable issues, or should not be used as reported. Cannot run code by design — the spec depends on it.

## Inputs

Each prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: the analyst and modeling-engineer both take an uploaded tabular file (CSV, Excel, or similar) plus optional context; the auditor takes a written model report plus optional supporting artifacts (model card, plots, code excerpts).

## How to Use

Paste the full text of a prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the required input in your first message. Specific code-execution and file-creation requirements differ across the three prompts — see *Platform setup* below.

## Workflow

The three prompts are designed to chain:

1. Run [data-analyst.md](data-analyst.md) on a dataset to understand what's there, or run [predictive-modeling.md](predictive-modeling.md) to build a model and produce a written report.
2. Run [predictive-model-auditor.md](predictive-model-auditor.md) on the modeling-engineer's report to catch issues the modeler may have missed or downplayed. The modeler's output contract (TL;DR → Setup → Performance → What the model learned → Caveats → Files) maps directly onto the auditor's checklist.

Each prompt also works standalone — use [data-analyst.md](data-analyst.md) for analysis without a modeling step, or [predictive-model-auditor.md](predictive-model-auditor.md) on a report produced outside this collection.

## Platform setup

All three prompts paste into the assistant builder's instruction field with no required knowledge files. The differences are in tool permissions:

| Prompt | Code execution | File creation | Web search |
| --- | --- | --- | --- |
| [data-analyst.md](data-analyst.md) | **Required** — every numeric claim must come from a computation | Required for charts | Off by default |
| [predictive-modeling.md](predictive-modeling.md) | **Required** — training, evaluation, serialization | **Required** — produces a joblib model + `MODEL_CARD.md` | Off by default |
| [predictive-model-auditor.md](predictive-model-auditor.md) | **Disabled** — the auditor reviews reports, it does not run analysis | Not needed (output is a short memo) | Off |

**Claude Projects:** toggle Code Execution and File Creation per the table. No project knowledge files required for any of the three.

**OpenAI Custom GPTs:** enable or disable *Code Interpreter & Data Analysis* per the table. Leave *Knowledge* empty unless you have a stable reference (a column dictionary, a feature glossary, organization-specific review standards). Custom GPTs do not treat knowledge as ambient — if you add files, append one sentence to the prompt's Knowledge & sources section telling the assistant to consult them.

**Gemini Gems:** confirm code execution is enabled (or disabled, for the auditor). Attach knowledge files only if you have a stable reference corpus; like Custom GPTs, Gems do not treat knowledge as ambient.

### Suggested conversation starters

- **Data Analyst:** "Analyze this file." / "Why did [metric] change?" / "What's interesting in this data?" / "Is there a relationship between [A] and [B]?"
- **Predictive Modeling Engineer:** "Build a model on this file." / "Predict [target] from this." / "Is there a signal in this data?"
- **Predictive Model Auditor:** "Audit this model report." / "Is this model ready to use?" / "What's wrong with this writeup?" / "Review this before I sign off."

## Maintenance notes

Guidance for anyone editing the prompts. The three prompts each have load-bearing structure that should not be flattened in pursuit of brevity.

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

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
