# Best Practices for Deep Research Agents

*A synthesis of 2026 vendor guidance and peer-reviewed research on prompting Claude Research, Gemini Deep Research, and ChatGPT Deep Research, current as of May 2026.*

---

## Introduction

The deep-research feature now offered by all three major frontier labs — Claude Research, Gemini Deep Research, ChatGPT Deep Research — is a different kind of tool from the chat models it sits on top of. The user types a prompt; the system invokes a planning layer; that planner decomposes the work, dispatches tool calls or subagents, gathers sources, and synthesizes a structured report over several minutes. The prompt is no longer the instruction the model executes. It is the seed for a multi-step plan the agent will execute on the user's behalf.

That structural change has reshaped what good prompting looks like in this context. The 2026 consensus across Anthropic, OpenAI, and Google is that the value has shifted away from clever phrasing and procedural scaffolding toward **completeness of brief and disciplined engagement with the planning layer**. The prompt is closer to a project brief for a junior analyst than to a search query.

This guide is a companion to PromptForge's 2026 prompting best practices. Where that guide treats deep research as one slice of a broader prompting landscape, this one goes deep on the specifics: what to put in the brief, how source quality and scope are actually controlled, how the three platforms diverge in their planning surfaces, what kinds of tasks deep research is and isn't the right tool for, and what the empirical evidence does and doesn't support.

A note on sources: this guide draws on official 2026 guidance from Anthropic, OpenAI, and Google, on the peer-reviewed and arXiv literature on deep-research agent evaluation, and on the public engineering retrospectives from the labs themselves. Specific quantitative claims are flagged with their underlying study. Where the evidence is thin or the framing has drifted toward marketing, that's called out inline.

---

## Part 1: What deep research actually is, and when not to use it

Before recommending how to prompt these tools, it's worth being explicit about what they are and what they aren't good for. Most of the bad outcomes practitioners report come from using deep research as a smarter search engine rather than as the analyst-style tool it's been designed as.

### 1.1 The architectural shift that matters

All three deep-research products are built around the same architectural pattern: an orchestration layer between the user and the underlying reasoning model. OpenAI's flow runs a clarification model that asks a small number of scoping questions, then a "rewriter" model that expands the user's prompt into a structured research brief before dispatching tool calls. Gemini exposes an editable multi-step plan as the primary specification surface — the user prompt seeds the plan, but the plan itself is where the work gets scoped. Claude's lead-agent orchestrator decomposes the prompt into subagent tasks and typically proceeds straight to execution without a visible planning step.

Different surfaces, same underlying shift: the user-facing prompt feeds a planner, and the planner is what drives the work. That means the prompt's job is to **specify the brief**, not to direct the method. Procedural scaffolding ("first, search for X; then summarize Y; then compare Z") that worked in older chat-based workflows tends to constrain the planner in ways that degrade output.

### 1.2 When deep research is the right tool

Deep research pays off when the task involves source synthesis across many references, novel comparison of entities the agent has to identify, or sustained investigation that would take a human analyst hours. The vendors are consistent on this. Concrete fits:

- Market landscape or competitive analysis across many companies.
- Vendor or technology evaluation with multiple comparison axes.
- Policy or regulatory comparison across jurisdictions.
- Literature synthesis or evidence reviews.
- Due diligence and executive briefings on unfamiliar domains.
- Multi-source factual investigation where the agent benefits from following citation trails.

### 1.3 When deep research is the wrong tool

Equally important is recognizing the cases where deep research overdelivers, burns quota, and produces worse output than a standard chat session would. The vendor guidance is explicit about this — Anthropic's documentation specifically warns against triggering Research for fact lookups — but it's the most-ignored piece of advice in practitioner literature. Concrete anti-fits:

- Single-fact lookups ("when did X ship," "what was Y's revenue last year"). Standard chat with web search is faster and at least as accurate.
- Summarizing a single document you already have. Use chat with the document attached.
- Brainstorming or ideation. Deep research over-constrains generative work.
- Coding tasks, debugging, or short comparisons that don't require source synthesis.
- Anything where you can predict the answer in a sentence. If the report could be replaced by a paragraph, you don't need the report.

A useful practitioner heuristic, not a formal vendor specification: if the task plausibly involves more than roughly five tool calls and would take a competent human at least an hour to do well, deep research is plausibly the right tool. If either condition fails, it probably isn't. OpenAI's official documentation describes deep research in terms of reasoning steps and web search calls rather than human-labor equivalents, so this is a rule of thumb, not a published threshold.

### 1.4 The performance ceiling worth knowing about

The benchmark literature offers a sobering anchor. Scale AI's *ResearchRubrics* (arXiv 2511.07685, November 2025) and the DeepResearch Bench / Mind2Web 2 / FINDER suites converge on the finding that frontier deep-research agents currently operate at roughly **50–70% of expert human researcher performance**, measured against rubric-based assessment of complete report quality on representative tasks. Failures cluster in three areas: evidence integration across sources, source-quality filtering, and missed implicit context.

The three failure modes are the ones the prompt can most directly address. That's where the prompting advice in the rest of this guide is concentrated.

---

## Part 2: The structural template that works across all three platforms

The 2026 consensus on what a deep-research prompt should contain is striking. OpenAI's published rewriter template (Cookbook, June 2025), Anthropic's engineering retrospective on their multi-agent research system (June 2025), and the fields exposed by Gemini's research-plan editor converge on the same six components.

### 2.1 The six components

```
# Objective
[One sentence: the decision or deliverable this report will support.]

# Audience and use
[Who will read it and what they'll do with it.]

# Scope
- Time frame: [explicit dates, or "open-ended"]
- Geography: [specific regions, or "global"]
- Language: [English / other / multilingual]
- Comparison set: [the specific entities, or "let the model identify"]
- Key entities: [list, or "open-ended"]

# Source priorities
- Prefer: [peer-reviewed / primary / official / regulatory / specific domains]
- Deprioritize: [aggregators / SEO blogs / Wikipedia]
- Seed URLs: [if any]

# Output
[Format: report / memo / comparison table / decision brief]
[Length: approximate word count or page count]
[Structure: section headers you want]
[Citation style: inline, footnoted, or per-claim]

# Analytical posture
[Data-rich / develops competing hypotheses / states confidence levels /
focuses on actionable implications. See section 2.3 for specific levers.]
```

This is not a magic template. It's a checklist for the things that, when underspecified, produce the failure modes the benchmark literature most often identifies. Each field maps to one of the dimensions a competent research brief would explicitly state.

### 2.2 The open-ended-not-omitted rule

OpenAI's rewriter rules and Anthropic's prompt-engineering guidance both insist on a discipline that's easy to overlook: **unstated dimensions should be marked open-ended rather than left blank**. If geography doesn't matter, say "global / no geographic constraint" rather than omitting it. If the comparison set is the agent's call, say "let the model identify the relevant comparison set" rather than omitting it.

This matters because the planning layer treats omissions as either (a) a default to invent silently or (b) a reason to pause and ask. Both are worse than an explicit "open-ended" marker. The single rule eliminates a large fraction of unnecessary clarifier round-trips on ChatGPT and unnecessary plan-revision cycles on Gemini.

### 2.3 Tunable levers in "analytical posture"

Of the six components, analytical posture is the most abstract — and the one where practitioner-level specification adds the most lift. Concrete levers that work across all three platforms:

- **Confidence and uncertainty.** "Provide qualitative confidence tiers (High / Medium / Low) for each claim." Or, "Provide a 95% confidence interval for each numeric estimate, or state explicitly that the estimate is point-only."
- **Counterhypotheses.** "For any causal claim, explicitly state the strongest alternative hypothesis and what evidence would distinguish them."
- **Source attribution at the claim level.** "Cite a primary source for every factual claim; label anything sourced from secondary aggregation as `[secondary]`."
- **Quantitative posture.** "Be data-rich. Where a number is available, prefer it to a verbal characterization. Flag estimates whose range exceeds 2x as 'speculative.'"
- **Decision orientation.** "End with a section titled 'What this means for the decision,' with actionable implications stated as bullet points."

These read as small refinements, but they are the difference between a report that reads as a competent summary and one that supports a decision. They also force the planning layer to allocate tool calls and synthesis time to the dimensions that actually matter.

### 2.4 What does *not* transfer from chat prompting

Three patterns that work well in chat prompting are actively counterproductive for deep research:

- **Chain-of-thought triggers.** "Think step by step before answering" was a useful prompt in 2023 and is now redundant or harmful. The deep-research backbones (the GPT-5.5 series, Claude Opus 4.7, Gemini 3 Thinking) already reason internally. Recent OpenAI prompting guidance explicitly notes that over-instruction now suppresses planning quality on frontier models.
- **Role-play personas.** "You are a senior McKinsey analyst with twenty years of experience" no longer improves output on frontier models and may anchor stylistic patterns you don't want. Specify the audience and the analytical posture instead; let the model handle the rest.
- **Few-shot example stuffing.** Pasting a sample of the report you want is **not** a recommended pattern for deep research on any platform. The 2026 OpenAI and Anthropic guidance is unanimous: where examples help in this context, it's for modeling output *structure* (a section-header outline), not for teaching content.

The arXiv preprint *You Don't Need Prompt Engineering Anymore: The Prompting Inversion* (2510.22251, October 2025) frames this as a reversal: sophisticated scaffolding now degrades frontier-model performance on research tasks. The phrasing in the paper's title is more absolute than the underlying claim supports — prompt engineering still matters, it just shifted upstream into brief specification, and the underlying experiment is on a specific benchmark (GSM8K) with a specific contrast between "Sculpting" and standard chain-of-thought. But the directional finding aligns with the vendor guidance: simpler is now usually better at the procedural layer for frontier-tier models.

---

## Part 3: Specificity is a constraint-balance problem

The single most consistent failure mode named by all three vendors is **over-instruction**. OpenAI's Deep Research team (Fulford and Sun, OpenAI Forum, April 2025) explicitly warned that "too many details or overly rigid instructions can limit creativity and reduce the accuracy and depth of insights" and recommended telling the model *the objective*, not the method. Anthropic's context-engineering guidance frames this as finding "the right altitude — a Goldilocks zone" between brittle if-then rules and vague high-level direction. Google's PM guidance for Gemini Deep Research goes further: don't engineer the initial prompt; tighten in the plan editor.

The 2026 empirical evidence supports the asymmetry. Scale AI's *ResearchRubrics* (arXiv 2511.07685) found that frontier deep-research agents fail not on task comprehension but on **missed implicit context** — under-specifying what you actually want hurts far more than imperfect phrasing. The intuition behind the result is straightforward: a vague prompt like "research the semiconductor shortage" gives the planner no way to choose between several plausible interpretations (which shortage, which era, which segment of the supply chain) and forces it to guess. The cure is richer objective specification, not more procedural direction.

The practical rule that emerges across all three labs: **be exhaustive about objective, scope, audience, sources, and output; be terse about method**. The framing that helps practitioners most is to think of prompt design as a constrained optimization problem: maximize the boundary conditions the planner needs (the *what* — objective, scope, output shape, source priorities, exclusions) and minimize the procedural constraints (the *how* — execution sequence, search-query construction, ordering of subtasks). Control the boundaries; leave the path to the reasoning engine.

A useful self-check, from the OpenAI Forum guidance: ask whether the prompt "clearly articulates the research objective without imposing excessive constraints or leaving too much ambiguity."

---

## Part 4: Source quality, recency, and exclusions

Source control has bifurcated into UI/connector controls and in-prompt natural-language specification. Both are now needed; neither alone is sufficient.

### 4.1 The UI and connector layer

- **ChatGPT Deep Research** added explicit "Sites → Manage sites" controls in February 2026, letting users either restrict research to an allowlist or "prioritize these sites, but allow full-web search." Read-only connectors include GitHub, Google Drive, Gmail, Calendar, SharePoint, Outlook, Teams, Slack, Dropbox, Notion, Canva, HubSpot, Intercom, FactSet, PitchBook, and custom MCP servers.
- **Gemini Deep Research** lets users toggle Google Search on/off and add or remove Workspace sources (Gmail, Drive, Chat, NotebookLM notebooks, uploaded PDFs). Deselecting Google Search restricts the run to private sources.
- **Claude Research** exposes source breadth through Integrations (Google Workspace plus remote MCP servers — Atlassian, Linear, Asana, Zapier, Cloudflare, Sentry, and a growing roster). Per Anthropic's Help Center, the recommended way to force their use is the explicit prompt phrase "Pull relevant context from [relevant internal knowledge source]."

The high-leverage move that practitioners under-use is **seeding with private sources**. All three platforms support file attachments, document connectors, and (in Gemini's case) linked NotebookLM notebooks. A deep-research run grounded in three of the user's own PDFs plus the open web outperforms a run on the open web alone on almost any specialized topic.

### 4.2 The in-prompt layer

In-prompt source guidance does most of the work for source quality, recency, and exclusions because none of the three platforms exposes formal search operators in the deep-research composer. Search-engine syntax (`site:`, `before:`, `-keyword`) is not recognized.

The pattern to use is natural-language specification of priorities and exclusions. The vendor guidance converges on the same content:

- **Source preferences.** "Prefer peer-reviewed academic literature, regulatory filings, and primary corporate communications. Use industry analyst reports (Gartner, Forrester) where peer-reviewed evidence is unavailable."
- **Source exclusions.** "Exclude SEO-optimized blogs, content farms, and aggregator sites. Do not use Wikipedia as a primary source."
- **Recency.** "Only sources published since January 2025." Or, "Prioritize sources from the last 18 months; older sources should be cited only for definitional or historical claims."
- **Language and locale.** "English-language sources only." Or, "English and German sources both acceptable."

Both Anthropic's and OpenAI's published guidance for their research products emphasize this kind of explicit source-quality direction. OpenAI's rewriter template encodes the priorities directly — "prefer linking directly to official or primary websites rather than aggregator sites or SEO-heavy blogs." The underlying observation is consistent: without explicit source-quality direction in the prompt, the default failure mode of all three agents is over-reliance on highly-ranked but low-authority sources. SEO-optimized content farms outrank academic PDFs in search results, and the agents inherit that ranking unless told otherwise.

The empirical literature reinforces the leverage of in-prompt source guidance. *SourceBench* (arXiv 2602.16942) evaluates the quality of sources cited by deep-research agents across multiple metrics (objectivity, freshness, domain authority, and others), and finds wide variation across agents and prompts. The implication for practitioners is that source-quality specification at the prompt level has a meaningful effect on output quality, often comparable in magnitude to changes in reasoning effort. Investing time in the source-priority field of the brief pays off more reliably than almost any other prompt-level intervention.

### 4.3 Bound the depth, don't maximize it

A counterintuitive finding from the 2026 academic literature, named clearly in *Cited but Not Verified* (arXiv 2605.06635): factual accuracy degrades meaningfully as deep-research agents are pushed to gather more sources. The reported figure across the underlying benchmarks is roughly **42% degradation in citation faithfulness** — the rate at which cited claims are actually supported by the cited source — as retrieval depth increases beyond the agent's optimal operating point. The mechanism has two plausible components: **cross-document entity resolution failure** (the agent struggles to reconcile competing data points across many sources) and **retrieval-noise accumulation** (low-signal paragraphs pulled into the synthesis stage distract from high-signal evidence). *ResearchRubrics* finds that synthesis, not retrieval, is the dominant failure mode, which is consistent with this picture.

The prompt-level mitigation is to bound depth explicitly. Phrases that work:

- "Cite 8–12 high-quality sources rather than maximizing breadth."
- "Prioritize depth and faithfulness over comprehensiveness."
- "If you find conflicting evidence, surface the conflict explicitly rather than averaging across sources."

The general principle: **more sources is not more accurate**. The agent's job is to be selective; the prompt's job is to give it permission to be.

---

## Part 5: Working with the clarification and plan step

The three platforms diverge most visibly here, and exploiting each one's flow is a meaningful quality lever. The UI patterns reflect different underlying engineering architectures, and the prompting strategy follows from the architecture.

### 5.1 ChatGPT Deep Research

ChatGPT runs a sequential, stateful flow: a clarification model asks a small number of scoping questions (typically a handful, covering dimensions like time frame, audience, geography, format, and key entities); the user's answers feed a rewriter model that produces the actual research brief; only then are the heavy reasoning and tool-call tokens spent. The exact question count varies by task; the architecture is consistent with using lighter compute to pin down parameters before expensive compute begins.

The practical implications:

- **Answer clarifying questions thoroughly rather than skipping them.** OpenAI's official guidance is explicit: "Requests are more expensive than standard queries, so take time to clarify." Skipping or terse answers cause the rewriter to invent defaults, which are usually wrong.
- **Pre-empt clarifiers in the initial prompt.** If the brief covers time frame, audience, geography, format, and key entities up front, the clarifier will typically proceed to the plan with brief confirmation rather than a full round-trip.
- **Use mid-run interrupts.** Since February 2026, ChatGPT supports interrupting a running task to redirect focus or change sources. This is a real lever — use it when the agent's first few minutes of output suggest it's drifting from intent.

### 5.2 Gemini Deep Research

Gemini's flow centers on an **editable multi-step plan**. The user prompt seeds an auto-generated plan; the user edits the plan in natural language ("focus more on differences between Google TPUs and competitor hardware, less on history"); the user explicitly executes. The architecture leverages Gemini's native long-context window and graph-based execution mapping — the plan is essentially a visible representation of the execution graph the agent will follow.

The practical implications:

- **Keep the initial prompt simple.** Google's PM guidance is explicit: don't engineer the initial prompt; the plan editor is where specificity is meant to be added. Treat the prompt as a seed and the plan as the actual brief.
- **Edit the plan, don't restart.** If the initial plan misses a dimension, edit it in place rather than reprompting. The plan editor is more precise than natural-language reprompting for tightening scope, adding sources, and reordering priorities.
- **API gotcha for developers.** The Gemini API exposes the equivalent flow through a `collaborative_planning: true` flag, with `previous_interaction_id` for iterating. The flag must be explicitly flipped to `false` to trigger execution — saying "go ahead" in natural language does not start the run. This is a documented surprise that has caused real production incidents.

### 5.3 Claude Research

Claude does not consistently surface a user-facing plan step. The lead-agent orchestrator proceeds directly from prompt to subagent decomposition and tool-call dispatch. The architecture is a hierarchical multi-agent routing tree designed to parallelize subtasks aggressively, which trades planning visibility for token-throughput.

The practical implications:

- **Front-load everything in the first prompt.** Anthropic's general prompt-engineering guidance is explicit: "Specify the task, intent, and relevant constraints upfront in the first human turn. Ambiguous or underspecified prompts conveyed progressively over multiple user turns tend to reduce token efficiency and sometimes performance." This applies with extra force to Claude Research, where progressive prompting is documented as an anti-pattern.
- **Add a clarification clause if you want a pause.** Anthropic's modeled starter-prompt template on claude.ai includes the phrase *"If you need more information from me, ask me 1-2 key questions right away."* Adding this clause to the brief is the explicit way to opt into a clarification round.
- **The full brief is what runs.** With no editable plan and no clarifier by default, the prompt template in section 2.1 is the operating specification. There's no downstream surface to fix omissions in.

### 5.4 What's the same across all three

The convergence under the surface is more striking than the divergence. All three planners look for the same six components; all three reward complete briefs over clever phrasing; all three respond to explicit source priorities and exclusions; all three degrade under chain-of-thought triggers and persona scaffolding; and all three accept seed URLs and attached documents as ways to bootstrap source quality.

The differences worth memorizing reduce to a small table:

| Dimension | ChatGPT Deep Research | Gemini Deep Research | Claude Research |
|---|---|---|---|
| **Planning surface** | Clarifier (2–6 questions) → editable plan | Editable multi-step plan (primary surface) | Lead-agent decomposition; usually no visible plan |
| **Where specificity belongs** | Pre-empt clarifiers in prompt; engage clarifier when it surfaces | Keep prompt simple; tighten in plan editor | Front-load everything in the first prompt |
| **Source whitelisting in UI** | Yes (Sites → Manage sites, Feb 2026) | Partial (toggle Search, choose Workspace sources) | No (via Integrations only) |
| **Mid-run interrupt** | Yes (Feb 2026) | No (must restart) | No |
| **Run duration** | Several minutes | 5–10 min consumer; up to 60 min API Max | 5–15 min standard; up to 45 min Advanced Research |
| **Best fit** | Enterprise data via connectors (FactSet, PitchBook, etc.); iterative refinement via interrupt | Plan interactivity; multimodal grounding; Workspace integration | Pure breadth via multi-agent parallelism; MCP-heavy workflows |
| **Platform-specific "don't"** | Don't skip clarifying questions; don't over-instruct method | Don't engineer the initial prompt — use the plan editor | Don't trigger Research for fact lookups; don't reveal task across turns |

---

## Part 6: Verification, citation hallucination, and what to check before trusting the output

Deep research agents produce reports that look authoritative. They are not always faithful to their own citations. The 2026 academic literature is sharper on this than the vendor guidance has been.

### 6.1 The citation-faithfulness problem

*Detecting and Correcting Reference Hallucinations in Commercial LLMs and Deep Research Agents* (arXiv 2604.03173) reports that **3–13% of cited URLs are fabricated** across commercial deep-research agents even with retrieval enabled. The wider category — claims attributed to a real source that the source does not actually support — runs higher and is harder to measure. The NeurIPS 2025 taxonomy work in *Compound Deception in Elite Peer Review* (arXiv 2602.05930) documents over 100 fabricated citations across the corpus studied, including in deep-research-assisted manuscripts. The phenomenon is real, reproducible, and not solved by prompting alone. The mitigations are partly prompt-level and partly workflow-level.

### 6.2 Prompt-level mitigations

- **Request primary sources by name.** "Cite the original peer-reviewed paper, regulatory filing, or official corporate communication. Do not cite secondary summaries when the primary source is accessible."
- **Bound depth.** Per section 4.3, citation faithfulness degrades with retrieval depth. "8–12 sources" outperforms "as many as possible."
- **Require quoted evidence.** "For any non-trivial factual claim, include a one-sentence quoted excerpt from the cited source." This makes hallucinated citations easy to detect on review.
- **Ask for confidence labels.** Per section 2.3, requesting High/Medium/Low confidence tiers surfaces the agent's own uncertainty about its claims.

### 6.3 Workflow-level mitigations

The prompt is necessary but not sufficient. A post-research validation pass is part of the workflow:

- **Spot-check primary sources.** Click into 3–5 cited links and confirm they exist and support the claim. Fabricated URLs are usually obvious on inspection.
- **Use a cheaper model to critique the report.** A standard Sonnet, GPT-4o, or Gemini Flash session asked "find the weakest claims in this report and check whether the cited sources support them" catches a meaningful share of issues that the original agent missed.
- **Reconcile conflicting data points.** Deep-research reports often paper over disagreement between sources. If two cited sources give different numbers for the same fact, the report should surface the conflict; if it doesn't, the synthesis layer ran past the disagreement.
- **Watch for chronological mixing.** A recurring synthesis failure is treating evidence from different periods as if it described a single state of the world. Spot-check the dates on claims about "current" conditions.
- **Watch for false equivalence.** Agents tend to present competing hypotheses with similar word counts even when the evidence is asymmetric. The post-review pass should ask whether the weight given to each position reflects the evidence weight.

A useful framing from the broader 2026 evaluation literature: deep-research agents are at roughly 50–70% of expert human performance, and the gap is closed by user judgment in scoping the brief and reviewing the output. The prompt is the seed; the review is the closing move.

---

## Part 7: Iteration, follow-ups, and chaining

Once an initial report is in hand, the three platforms differ in how iteration works.

**ChatGPT and Gemini both support in-place follow-ups.** Gemini's documented pattern: "you can ask Deep Research to add something new to the report after it's been generated and it will adjust the report in real time. For example, 'add camp cost details to my report.'" ChatGPT supports follow-up turns and, since February 2026, mid-run interrupts.

**Claude is less explicit about iteration.** Follow-ups in the same conversation work, but Anthropic has not documented whether each follow-up triggers a full multi-agent cycle or a lighter response. The community pattern is to chain prompts referring to specific sections rather than asking for a fresh run.

A few iteration patterns that work across all three:

- **Reference specific sections.** "Go deeper on the methodology section. Find three additional 2025 sources and integrate them." Specific is meaningfully better than "make it better."
- **Use a cheaper non-research model for prompt refinement.** OpenAI's research team explicitly recommends this — GPT-4o or GPT-5 Instant for iterating on the prompt, then commit the polished prompt to the deep-research queue once it's solid. The same logic applies to Sonnet on the Claude side and Flash on the Gemini side. Deep research burns meaningful quota; iterate on the brief before spending it.
- **Treat the first run as a draft.** The benchmark literature suggests roughly half of high-stakes deep-research outputs benefit from at least one refinement pass. Budget for it.

---

## Part 8: A worked example

Translating principles into a prompt is the part that's easiest to get wrong. A short before/after.

**Before — typical underspecified prompt:**

> Research the market for AI coding assistants and tell me which companies are winning.

This is a plausible chat prompt and a bad deep-research prompt. It's vague on time frame, scope, comparison set, audience, output shape, and source priorities. The clarifier on ChatGPT will round-trip on most of those dimensions; the planner on Gemini will guess at them; Claude will proceed straight to subagent decomposition with its own assumptions.

**After — a deep-research brief built from the template:**

> **Objective:** Identify the market leaders in AI coding assistants and assess which are best positioned for enterprise adoption over the next 18 months, to support a procurement decision for our 200-developer engineering org.
>
> **Audience and use:** Engineering leadership at a mid-market SaaS company evaluating a multi-seat license. Output will be reviewed by the CTO and the head of platform engineering.
>
> **Scope:** Time frame — products and traction from 2024 through May 2026. Geography — global, with weighting toward North American enterprise adoption. Language — English. Comparison set — GitHub Copilot, Cursor, Anthropic Claude Code, Cognition Devin, Replit, and any other product with documented enterprise adoption at 1,000+ seats; let the model identify additional entrants meeting that bar. Key entities — the products above plus their underlying model providers.
>
> **Source priorities:** Prefer primary corporate communications (product announcements, pricing pages, enterprise case studies), peer-reviewed evaluations where available, and analyst coverage from Gartner, Forrester, and Redmonk. Deprioritize SEO blogs, listicles, and aggregators. Do not cite Wikipedia. Seed URL: [internal evaluation doc URL].
>
> **Output:** Decision brief, approximately 2,500 words, with sections for (1) market overview, (2) per-product capability comparison, (3) enterprise adoption signals, (4) pricing and licensing models, (5) limitations and risks, (6) recommendation. Include a per-product comparison table on capability, pricing, and adoption. Inline citations after every factual claim.
>
> **Analytical posture:** Be data-rich; prefer specific numbers (seat counts, ARR figures, model benchmark scores) to verbal characterizations. Provide High/Medium/Low confidence tiers for each adoption claim. Where two products are close on a dimension, surface the trade-off explicitly rather than smoothing over it. Cite 10–15 high-quality sources rather than maximizing breadth.
>
> *Ask 1–2 key questions before researching if anything material is unclear; otherwise proceed and state your assumptions.*

The second version is longer, but the length is doing work. Every component eliminates either a clarifier round-trip, a planner guess, or a class of failure mode the benchmark literature identifies. The vendor guidance from all three labs converges on prompts in roughly this shape for non-trivial deep-research tasks.

---

## Part 9: Open questions and weak evidence

A few areas where the 2026 evidence is thin enough to be worth flagging:

- **The "inversion" framing.** The arXiv preprint *You Don't Need Prompt Engineering Anymore: The Prompting Inversion* (2510.22251) is the most cited public source for the "less prompt engineering is better" claim, and its underlying finding — that procedural scaffolding hurts on the highest-tier models — is well-supported. But the title is more absolute than the result. The actual experiment compares "Sculpting" (constrained chain-of-thought) against standard CoT on GSM8K across three model tiers, and the effect inverts only at the GPT-5 tier. Prompt engineering has shifted upstream into brief specification; it hasn't disappeared. The strong version of the claim circulating in practitioner literature should be treated with calibration.
- **The 50–70% expert-human-parity figure.** This is reported across DeepResearch Bench, Mind2Web 2, ResearchRubrics, and FINDER, but each benchmark uses slightly different methodology (rubric-graded report quality, task-completion rates, or paired human-preference testing). *ResearchRubrics* specifically reports leading agents under 68% compliance with expert criteria, with some domains as low as the high 50s. The convergence on a similar range is informative; the specific numbers should not be treated as precise.
- **The 42% citation-faithfulness degradation with retrieval depth.** Reported in *Cited but Not Verified* (arXiv 2605.06635) for a specific set of agents and tasks; the underlying mechanism (cross-document entity resolution failure plus retrieval-noise accumulation) is well-described but the specific magnitude does not necessarily generalize to every model or task.
- **Multi-agent quality claims.** Anthropic's reported gain of its multi-agent system over a single-agent baseline on internal evals is a meaningful signal about Claude Research's architectural direction, but it's on internal benchmarks and not directly comparable to public deep-research benchmarks.
- **Domain-specific quality.** All three labs' deep-research products have been evaluated mostly on general-knowledge and STEM benchmarks. Quality on specialized domains (clinical, legal, niche engineering) is less documented and more variable. Practitioners report higher variance and stronger benefit from seeded private sources in these domains.
- **The pace of feature change.** ChatGPT's Manage sites and mid-run interrupts shipped in February 2026; Gemini's Workspace integration expanded in November 2025; Claude's MCP roster has expanded continuously. Model versions also turn over quickly — by mid-2026 the named backbones above (GPT-5.5, Claude Opus 4.7, Gemini 3) will likely already have successors. Vendor-specific specifics in this guide should be cross-checked against current help-center documentation before being relied on in production.

---

## Sources

This guide synthesizes the following sources, organized by tier:

**Tier 1 — Peer-reviewed and arXiv research (2025–2026):**

- Sharma et al. *ResearchRubrics: A Benchmark of Prompts and Rubrics for Evaluating Deep Research Agents.* arXiv 2511.07685, November 2025.
- Jin et al. *SourceBench: Can AI Answers Reference Quality Web Sources?* arXiv 2602.16942, February 2026.
- *Cited but Not Verified: Parsing and Evaluating Source Attribution in LLM Deep Research Agents.* arXiv 2605.06635, 2026.
- *Detecting and Correcting Reference Hallucinations in Commercial LLMs and Deep Research Agents.* arXiv 2604.03173, 2026.
- *Compound Deception in Elite Peer Review: A Failure Mode Taxonomy of 100 Fabricated Citations at NeurIPS 2025.* arXiv 2602.05930, 2026.
- Khan, I. *You Don't Need Prompt Engineering Anymore: The Prompting Inversion.* arXiv 2510.22251, October 2025.
- DeepResearch Bench; Mind2Web 2; FINDER / DEFT (benchmark suites referenced throughout the 2026 deep-research literature).

**Tier 2 — Primary documentation from frontier AI labs (2025–2026):**

- OpenAI: *Deep Research rewriter template* (Cookbook, June 2025); ChatGPT Deep Research Help Center; Forum session with Isa Fulford and Edward (Zhiqing) Sun (April 2025); *GPT-5.2 Prompting Guide* (2026); Sites/Manage sites and mid-run interrupt documentation (February 2026).
- Anthropic: *How we built our multi-agent research system* (engineering blog, June 2025); *Context engineering for Claude* (2026); Claude Research and Advanced Research Help Center; *Prompting best practices* (Claude API docs); MCP integration documentation.
- Google: Gemini Deep Research product blog and Help Center; Gemini API documentation including `collaborative_planning` flag and `URL Context` tool; Aarush Selvan (Gemini PM) public guidance on plan-first prompting; NotebookLM integration documentation.

**Tier 3 — Secondary technical sources used selectively:**

- InfoQ coverage of deep-research benchmarks and Anthropic multi-agent retrospective.
- Practitioner playbooks and vendor-adjacent technical write-ups, used only where they corroborated claims from Tier 1 or Tier 2 sources.

**Sources downweighted or excluded:** Practitioner blog content with proprietary-sounding framework names (Executive LLM Protocol, Hierarchical Chain-of-Thought, Adaptive Graph of Thoughts), specific percentage claims sourced to single benchmarks on narrow tasks, and Medium/Reddit posts that did not corroborate higher-tier evidence. Where these sources echoed Tier 1 or Tier 2 findings, those findings are included via the higher-tier citation.

---

*Last updated: May 2026. Deep research products are evolving rapidly; specific UI elements, connector rosters, and run-duration figures should be verified against current vendor documentation before relying on them in production.*

---

*Compiled and maintained by [Michael Bagalman](https://michaelbagalman.com/). For the philosophy behind this work, see [michaelbagalman.com/philosophy.html](https://michaelbagalman.com/philosophy.html).*
