# Corporate Prompts

System prompts for the everyday work of running a function, a team, or a small organization: setting cadence, structuring a strategy deck, planning a project, prioritizing a backlog, and reporting status. Industry-agnostic — the domain context comes from the user, not the prompt.

## Files

- **[chief-of-staff.md](chief-of-staff.md):** Daily/weekly operating-cadence partner. Reality-checks estimates against past patterns, surfaces 50/30/20 time-budget drift, logs significant decisions, applies the Head/Heart/Hand diagnostic to stalls, and produces a five-section daily brief (Context Sync → Strategic Layer → Landscape → Focus (max 3) → Accountability) under a 350-word ceiling.
- **[presentation-deck-consultant.md](presentation-deck-consultant.md):** Structures board-ready strategy decks using the Pyramid Principle, MECE, and the Situation/Complication/Resolution frame. Produces a three-part deliverable: SCR synthesis → Ghost Deck (6–12 Action Titles, with a MECE check) → slide-by-slide layouts with explicit evidence placeholders where the user has not supplied data. Industry-agnostic.
- **[project-planner.md](project-planner.md):** Turns vague project ideas into structured execution plans. Five-section output (Overview → Milestones → Execution Phases → Dependencies and Risks → Resource Allocation), Three-Point/PERT estimates, T-shirt sizing, explicit critical path, and a named 20–30% contingency buffer that does not get optimized away.
- **[project-prioritizer.md](project-prioritizer.md):** Cuts workload noise via the PICK matrix (Payoff × Effort → Implement / Challenge / Possible / Kill). Refuses to classify against undefined axes, defends a 1–2-Challenge ceiling, surfaces capacity overload, and produces an explicit "Not-To-Do" list with drop / delegate / defer-with-kill-trigger recommendations.
- **[status-report-architect.md](status-report-architect.md):** Distills raw activity notes into a RAG-status report for executive or internal audiences. Outcomes-not-activities discipline, mitigation-first when health is Red, quantified variance, and a six-section structure (Executive Summary → Key Outcomes → Planned Outcomes → RAG Breakdown → Risks and Blockers → Milestone Tracking).

## Inputs

Each prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: the chief of staff needs today's hard constraints plus open work and the operating phase; the deck consultant needs the strategic question, audience, and available evidence; the project planner needs the goal, timeline, team, and constraints; the prioritizer needs the candidate list plus working definitions of payoff and effort; the status-report architect needs the activity log, blockers, and reporting period.

## How to Use

Paste the full text of a prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the required inputs in your first message. None of the prompts require code execution, file creation, web search, or knowledge files — they run against what the user supplies in the session.

## Platform setup

All five prompts paste directly into the assistant builder's instruction field. No tool toggles or knowledge files are required.

| Prompt | Code execution | File creation | Web search | Knowledge files |
| --- | --- | --- | --- | --- |
| [chief-of-staff.md](chief-of-staff.md) | Not needed | Not needed | Off | None |
| [presentation-deck-consultant.md](presentation-deck-consultant.md) | Not needed | Not needed | Off (or on if the user wants you to look up benchmarks the user has not supplied; the prompt's evidence-integrity rules still apply) | None |
| [project-planner.md](project-planner.md) | Not needed | Not needed | Off | None |
| [project-prioritizer.md](project-prioritizer.md) | Not needed | Not needed | Off | None |
| [status-report-architect.md](status-report-architect.md) | Not needed | Not needed | Off | None |

### Suggested conversation starters

- **Chief of Staff:** "Plan my day." / "What should I focus on?" / "I have these meetings and these open commitments — what's the brief?" / "Log this decision for me."
- **Presentation Deck Consultant:** "Help me structure a board deck on [topic]." / "I need a deck recommending [direction] to [audience]." / "Audit this draft deck for MECE and Action Titles." / "Turn this analysis into a strategy deck."
- **Project Planner:** "Plan this project." / "Break this initiative into phases." / "Estimate this and identify the critical path." / "Build me a risk register for [project]."
- **Project Prioritizer:** "Triage this list." / "What should I drop?" / "PICK-matrix my backlog." / "I have too much on my plate — help me cut."
- **Status Report Architect:** "Turn these notes into a status report." / "Draft this week's update for the [audience]." / "Is this project Green, Amber, or Red based on these facts?" / "Translate this activity log into an executive update."

## Maintenance notes

Guidance for anyone editing the prompts. Each prompt has load-bearing structure that should not be flattened in pursuit of brevity.

### Chief of Staff

- **Most likely to drift:** the three-action ceiling for The Focus and the calibration-over-compliance rule. Both erode under user pressure ("just add one more," "no, my estimate is right"). Reinforce inside Constraints rather than adding new sections.
- **Coupled sections:** The Focus block in the output contract and the five-step analytical pass in *How requests are handled*. If a new analytical step is added, surface it in the output rather than letting it live silently in reasoning.
- **Do not flatten:** the five-section output contract, the under-350-word ceiling, or the Decision Log format. The discipline of compression is what differentiates the chief of staff from a generic productivity assistant.

### Presentation Deck Consultant

- **Most likely to drift:** the evidence-integrity rule (no fabricating numbers, no synthesizing source names) and the Action-Title-is-a-sentence rule. Under pressure to produce a complete deck, both regress toward "plausible-sounding" content. Reinforce inside Constraints.
- **Coupled sections:** the three-pass workflow in *How requests are handled* (SCR → Ghost Deck → slide composition) maps directly onto the three output sections. If a new pass is added (e.g., a tone/voice pass), add the corresponding output section.
- **Do not flatten:** the three-section output contract, the 6–12-slide default, or the explicit-placeholder rule for missing evidence. The placeholder discipline is what keeps the prompt from quietly hallucinating market data.

### Project Planner

- **Most likely to drift:** the 2–8h leaf-task zone and the 20–30% contingency buffer. The first erodes when the user wants "fewer tasks"; the second erodes when the user wants "a more aggressive timeline." Both are defenses against the most common planning failures; hold the line in Constraints rather than negotiating them away.
- **Coupled sections:** the *Definition of done is observable* rule in Constraints and the Done criteria column in the Execution Phases table. They are the same rule applied at two scopes (milestone-level and task-level); keep them aligned.
- **Do not flatten:** the five-section output contract, the named critical path, or the named contingency buffer. A plan without a critical path is incomplete; a plan that hides the buffer inside task estimates is a plan that loses its absorption capacity in the first slip.

### Project Prioritizer

- **Most likely to drift:** the "refuse to classify against undefined axes" rule and the Kill-decisiveness rule. Users will push the prompt to "just sort the list," and the prompt will tend to soften Kill calls into "consider deprioritizing." Both regressions are the prompt becoming useless. Reinforce inside Constraints.
- **Coupled sections:** the four output sections map to the four PICK quadrants. If the quadrant taxonomy changes, update the output contract together.
- **Do not flatten:** the four-section output contract, the 1–2-Challenge ceiling, or the explicit Kill list. The Kill list is the prompt's reason to exist — a PICK matrix without a Kill column is a polite prioritization exercise, not a prioritization tool.

### Status Report Architect

- **Most likely to drift:** the outcomes-not-activities rule and the mitigation-first-for-Red rule. The first erodes when the activity log is rich but no outcomes have landed (the temptation to dress activity as progress is strong); the second erodes under user pressure to "explain what happened" before "say what we're doing." Both are the headline disciplines of the prompt; hold the line in Constraints.
- **Coupled sections:** the *RAG determination* analytical step and the RAG Breakdown table in the output. The per-dimension RAG colors in the table must be consistent with the overall health and with the activity log. If a new dimension is added (Quality, Regulatory), update both together.
- **Do not flatten:** the six-section output contract, the mitigation-first ordering for Red, or the quantified-variance rule. A status report without quantified variance reverts to an emotional narrative; the report's value is its discipline of measurement.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
