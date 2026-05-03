# Writing System-Level Prompts in 2026

## A practical guide for `CLAUDE.md`, Gemini Gems, and OpenAI Custom GPTs

This guide synthesizes 2026 guidance from Anthropic, OpenAI, Google, and Microsoft, plus 2026 peer-reviewed work, into one focused workflow for writing the *persistent instruction prompt* that defines a custom assistant — whether that's a `CLAUDE.md` project file, a Gemini Gem's instructions field, or an OpenAI Custom GPT's "Instructions" box.

These three artifacts serve the same function: a system-level specification that ships with every conversation and shapes how the model behaves before the user types anything. The principles below apply to all three. Platform-specific notes are called out where they matter.

---

## 1. The mental model: a spec, not a conversation

The single most important shift in 2026 guidance is that a good system prompt behaves like a **lightweight interface specification**, not a friendly note to the model. OpenAI, Microsoft, Google, and Anthropic all converge on the same skeleton:

1. **Role / scope** — what this assistant is and is not for
2. **Context** — durable facts the model should treat as background
3. **Task pattern** — the kinds of requests it should handle and how
4. **Constraints** — hard rules, things to avoid, output contracts
5. **Fallback behavior** — what to do when unsure, out-of-scope, or missing information

Every field has a diagnostic purpose: when the assistant misbehaves later, you can localize the failure. If formatting drifts, your output contract is weak. If the assistant overreaches, your fallback policy is weak. If it hallucinates, your grounding boundary is weak. Conversational, paragraph-style prompts make this kind of debugging nearly impossible.

A second key shift: **more text is not automatically better**. OpenAI's 2026 guidance explicitly says "be specific but keep the prompt simple," and Microsoft warns that overly long system messages consume context and bury requirements. The goal is precision, not volume.

---

## 2. The five-section template

Use this as your default starting structure. Every section earns its place; cut what doesn't apply.

```markdown
# Role
[One or two sentences: what the assistant is, who it serves, what domain it covers.]

# Context
[Durable background the model should treat as fact. Project facts, vocabulary,
key people, conventions, links to canonical documents. Keep tight.]

# How to handle requests
[The task pattern. What kinds of asks come in. The default workflow for a
typical request. When to ask vs. when to act.]

# Constraints
- [Hard rules, expressed as bullets.]
- [Output formats, length limits, tone requirements.]
- [Things to avoid, with reasons where useful.]

# When unsure
- If the request is ambiguous, [ask one clarifying question / state assumptions / etc.].
- If a required input is missing, [say so explicitly / write "Not specified" / etc.].
- If outside scope, [redirect / decline / suggest alternative].
```

This is the structure OpenAI's Enterprise guide recommends, what Microsoft's system-message checklist enforces, and what Anthropic's Claude documentation describes for `CLAUDE.md` project memory. It transfers cleanly across all three platforms.

---

## 3. Section-by-section guidance

### 3.1 Role

Define the assistant's expertise and scope, not its personality. "Senior cybersecurity architect specializing in cloud-native applications" is more useful than "expert and helpful AI assistant." 2026 work on persona prompting found that vague or demographically-loaded personas frequently *degrade* output quality, while professional-role personas reliably help by setting domain vocabulary and priorities.

**Good:**
> You are a technical editor for a B2B SaaS company's developer documentation. Your audience is working backend engineers.

**Avoid:**
> You are a brilliant, creative, friendly assistant who loves to help with anything!

### 3.2 Context

This is where `CLAUDE.md` and Custom GPT instructions earn their keep — they're the right place for things you'd otherwise repeat in every conversation. Project name, team conventions, key terminology, the data model, the canonical doc links, the "the 'platform' always means X in this codebase" disambiguations.

A useful test: would you have to type this same context into the chat box more than twice? If yes, it belongs in the system prompt.

Keep context **dense, not narrative**. Bullets and short factual lines outperform paragraphs.

### 3.3 How to handle requests

This is the section most people skip, and it's where Custom GPTs and Gems most often go wrong. The model needs to know what its *typical* turn looks like. A few patterns:

- *"When the user shares code, your default response is: (1) summarize what it does, (2) flag any bugs or smells, (3) suggest improvements only if asked."*
- *"For drafting requests, produce one draft first, then ask whether to revise. Don't produce three variants by default."*
- *"If the user asks a factual question about our codebase, search the attached documents before answering. If you can't find the answer there, say so before falling back to general knowledge."*

This section is also where you set **reasoning posture**. The 2026 evidence here is important and somewhat counterintuitive:

- **Do not** add "think step by step" by default. For modern reasoning-capable models (Claude 4.x, GPT-5, Gemini 2.5/3), the model already reasons internally; demanding visible chain-of-thought can hurt output quality and waste tokens.
- **Do** ask for verification when correctness matters: *"Before finalizing a numeric answer, check it once independently."*
- **Do** specify reasoning depth via the platform's effort/thinking parameter (when exposed) rather than via prompt text.

### 3.4 Constraints

Express as bullets. Be concrete. Give numbers when you have them.

**Useful constraints look like:**
- Keep responses under 400 words unless asked for more.
- Use Markdown headings only at H2 or below.
- Never invent citations. If you can't verify a source, omit it.
- Use British English spelling.
- Code examples must include type hints and one-line docstrings.

**Less useful constraints look like:**
- Be helpful and accurate.
- Use good judgment.
- Don't make things up.

The pattern: *observable, checkable, specific*. If you couldn't write a test for it, the model probably can't reliably follow it.

### 3.5 When unsure (the fallback policy)

The 2026 guidance is unanimous on this and most users still ignore it: **a good system prompt defines what safe failure looks like, not just what success looks like.** Microsoft's checklist requires it, OpenAI's enterprise guide recommends it, Google's grounding patterns formalize it.

A serviceable default:

```markdown
# When unsure
- If the request is ambiguous, ask one clarifying question before proceeding.
- If you don't know the answer, say "I don't know" rather than guessing.
- If a request is outside your scope (defined above), say so and suggest where the user might look instead.
- When citing sources, only cite ones you can name specifically. Otherwise omit the citation.
- Mark uncertain claims with "(uncertain)" rather than presenting them as facts.
```

This single section eliminates a large class of common failures: invented citations, confident wrong answers, scope creep, and silent assumptions.

---

## 4. Examples: same task, three platforms

### 4.1 A research-summarizing assistant — `CLAUDE.md` version

```markdown
# Role
You are a research assistant for a single analyst working in media-industry
strategy. You help summarize, compare, and critique reports — primarily PDFs
and web articles.

# Context
- The analyst's beat: streaming, premium video, and ad-supported video.
- Default audience for any deliverable: a non-technical executive.
- Preferred citation style: inline (Author, Year) with a source list at the end.

# How to handle requests
- For "summarize X": produce (1) a 3-sentence executive summary, (2) 3–5 key
  findings as bullets, (3) one paragraph on what's missing or weakly supported.
- For "compare X and Y": produce a table of points of agreement and disagreement
  before any prose synthesis.
- For "critique X": separate (a) what the source claims, (b) the evidence offered,
  (c) the gaps or alternative explanations. Do not editorialize before doing (a)–(c).

# Constraints
- Use only material the analyst provides. Do not pull in outside facts unless
  explicitly asked.
- Keep total length under 500 words unless asked for a long-form treatment.
- Quote sparingly: max one short quote per source, never more than 15 words.
- Never invent author names, dates, or page numbers.

# When unsure
- If the source is silent on something the analyst asked about, write "Not addressed in source."
- If two sources conflict, surface the conflict explicitly rather than picking one.
- If asked for an opinion outside the source material, say so before offering one.
```

### 4.2 The same idea — Gemini Gem instructions

Gemini Gems use a single instructions field. The structure is identical; only the framing changes slightly because Gem instructions tend to be read by the model as a direct system message:

```markdown
You are a research assistant for a single analyst working in media-industry
strategy (streaming, premium video, AVOD). You help summarize, compare, and
critique reports.

Default audience for any deliverable: a non-technical executive.
Preferred citation style: inline (Author, Year) with a source list at the end.

For "summarize X": produce (1) a 3-sentence executive summary, (2) 3–5 key
findings as bullets, (3) one paragraph on what's missing or weakly supported.

For "compare X and Y": produce a table of agreements and disagreements before
any prose synthesis.

For "critique X": separate (a) the claim, (b) the evidence, (c) the gaps —
in that order, before editorializing.

Hard rules:
- Use only material the analyst provides; don't import outside facts unless asked.
- Length cap: 500 words unless explicitly asked for more.
- Quote sparingly; never more than ~15 words per quote, max one quote per source.
- Never invent authors, dates, or page numbers.

When unsure:
- If the source is silent on something, write "Not addressed in source."
- Surface conflicts between sources rather than picking one silently.
- Flag opinions as opinions when stepping outside the source material.
```

### 4.3 The same idea — OpenAI Custom GPT instructions

OpenAI Custom GPTs work the same way, with one platform-specific tweak: the instructions field is paired with a separate "Conversation starters" UI and uploaded knowledge files. Reference them explicitly:

```markdown
# Role
You are a research assistant for a media-industry strategy analyst. You help
summarize, compare, and critique reports.

# Knowledge files
The analyst has uploaded reference documents. Treat the uploaded files as
authoritative for any factual claim about the analyst's company, products,
or internal terminology. For external industry facts, prefer the user's
pasted material; only fall back to your training data if explicitly asked.

# Output patterns
- Summaries: 3-sentence exec summary → 3–5 key findings → one paragraph on gaps.
- Comparisons: agreements/disagreements table → prose synthesis.
- Critiques: claim → evidence → gaps, in that order.

# Constraints
- Default length cap: 500 words.
- Quotes: max one per source, under 15 words each.
- Never fabricate citations, authors, or dates.

# When unsure
- "Not addressed in source" beats guessing.
- Surface source conflicts explicitly.
- Flag opinions as opinions.
```

---

## 5. Patterns worth knowing

These are smaller patterns that show up repeatedly in 2026 guidance and are worth keeping in your back pocket.

### Bounded extraction before synthesis

For anything evidence-grounded (research, legal, medical, internal-doc Q&A), force the model to *extract* first and *synthesize* second:

> First, extract the relevant passages verbatim into a list. Then, and only then, produce the summary. Do not synthesize until extraction is complete.

This dramatically reduces hallucination at the cost of slightly longer outputs. The 2026 medical-extraction work confirms it for systematic reviews; the same pattern works for any retrieval-flavored task.

### Few-shot for format, not for reasoning

If you want consistent output structure (label drift, JSON shape, tone), include 2–3 examples in the prompt. If you want better reasoning, examples are a weak lever — adjust effort/thinking parameters or restructure the task instead.

The 2026 academic evidence is sharp on this: in medical paper screening, more detailed prompts improved precision but *hurt* recall. Examples bias the model toward the example pattern, which is good when you want pattern conformity and bad when you want broad coverage.

### Recall vs. precision is a prompt design choice

Decide which one matters before writing. A "find every relevant X" prompt looks different from a "only flag X if confident" prompt. The same template can be tuned in either direction by swapping a few constraint lines:

- **Recall-tuned:** *"When in doubt, include the item and mark it (uncertain)."*
- **Precision-tuned:** *"Only include items where you have direct evidence. When in doubt, exclude."*

### Avoid "anti-laziness" theatrics

Capitalized injunctions like *"YOU MUST ALWAYS..."* and *"THIS IS CRITICAL"* are leftovers from 2023–2024 prompting that no longer help on frontier models — and per Anthropic's 2026 guidance, can over-trigger tool use and degrade output. Write in the same calm, declarative voice you'd use in a technical spec.

### Don't aggressively shorten outputs

Anthropic's April 2026 postmortem documents that adding a "be concise" system instruction measurably hurt coding output quality. If you want shorter outputs, request shorter outputs *for specific kinds of requests* rather than installing a global brevity rule.

---

## 6. Iteration: the part most users skip

A system prompt is never "done" — it gets refined against real failures. The 2026 industry-standard loop:

1. **Define** the quality properties that matter (e.g., "summaries must include all five required sections," "code examples must run").
2. **Test** the prompt against ~10–20 realistic requests, including edge cases.
3. **Diagnose** failures: which section of the prompt is responsible? (This is why the structured template pays off — failures localize.)
4. **Fix** by editing the responsible section, not by piling on more instructions elsewhere.

Two anti-patterns to watch for:

- **Instruction stacking.** Each time something goes wrong, a new bullet gets added. After ten iterations, the prompt is bloated, contradictory, and worse than the original. When you add a constraint, ask whether it replaces an existing one.
- **Single-example tuning.** A prompt that handles one tricky case well often regresses on five other cases. Always re-run your test set after a change.

---

## 7. Platform-specific notes

### `CLAUDE.md`

- Lives at the root of a project (or in `~/.claude/CLAUDE.md` for global scope).
- Particularly good for codebase conventions, build/test commands, file-structure facts, and the "things you'd otherwise re-explain every session" category.
- A `.claudeignore` companion file excludes noise from context — use it.
- Markdown headings (`#`, `##`) are read meaningfully; structure pays off.

### Gemini Gems

- Single instructions field, no separate sections — but internal Markdown structure still helps the model parse.
- Gems support knowledge files; reference them explicitly in the instructions ("the uploaded files are authoritative for X").
- No public chain-of-thought control, but Gemini's internal "thinking" handles most reasoning needs without prompting tricks.

### OpenAI Custom GPTs

- Instructions field has a soft length cap; concision matters more here than in `CLAUDE.md`.
- "Conversation starters" populate the new-chat screen — write them as common task patterns the GPT is optimized for.
- Knowledge files behave like a small RAG corpus; tell the GPT *when* to consult them, not just that they exist.
- Actions (API calls) are configured separately but should be referenced in the instructions if the GPT should default to using them.

---

## 8. A self-check before shipping

Before saving any system prompt, run through this:

- [ ] Could a new team member read this and predict how the assistant will behave on a typical request?
- [ ] Is there a clear answer to "what should the assistant do when it doesn't know"?
- [ ] Have I tested the prompt against at least one ambiguous request, one out-of-scope request, and one missing-information request?
- [ ] Are my constraints observable — could I tell from the output whether they were followed?
- [ ] Did I resist the urge to add "think step by step," "be helpful and accurate," or capitalized urgency?
- [ ] Is anything in the prompt redundant with the model's defaults? (If so, cut it.)

If you can answer "yes" to all six, you have a prompt that will hold up across sessions and degrade gracefully when things go wrong — which is the actual measure of a good system prompt in 2026.

---

## Sources

This guide synthesizes:

- Anthropic, *Prompting best practices* (Claude API docs, 2026); *Claude Code quality postmortem* (April 2026); *Introducing Claude Opus 4.7* (April 2026).
- OpenAI, *Prompting fundamentals* (Academy, April 2026); *ChatGPT Enterprise Prompting Guide* (April 2026); *Codex Prompting Guide* (February 2026); *Testing Agent Skills with Evals* (January 2026).
- Google AI for Developers, *Prompt design strategies* (April 2026); *Gemini 3 Developer Guide* (April 2026); Google Cloud, *Lessons from generating code at scale* (February 2026); Google Research, *Vantage protocol* (April 2026).
- Microsoft Learn, *Prompt engineering techniques* (February 2026); *System message design for Azure OpenAI* (February 2026); *Run evaluations from Foundry portal* (May 2026).
- Adam et al., *Prompt engineering for paper screening in medical meta-analyses* (Research Synthesis Methods, 2026).
- Oami et al., *LLM data extraction in systematic reviews* (Frontiers in Digital Health, 2026).
- Yang et al., *Persona Prompting as a Lens on LLM Social Reasoning* (EACL 2026).
- Li et al., *Debiasing LLMs via Adaptive Causal Prompting with Sketch-of-Thought* (Findings of EACL 2026).
- Devadiga & Chopra, *Structured Prompting for Tulu* (LoResLM 2026).
