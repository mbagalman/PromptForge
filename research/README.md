# Research Prompts

System prompts for synthesis, fact-checking, and adversarial analysis, plus a standalone brief-writing tool for deep-research agents. The three verification-pipeline prompts are designed to be used together but each works standalone; the brief specialist is independent and scopes work before handing off to an external deep-research agent.

## Files

### Verification pipeline

- **[content-digest.md](content-digest.md):** Deep synthesis and critique of a user-specified work, with citations and a Read/Skim/Skip recommendation.
- **[fact-checker.md](fact-checker.md):** QA audit on a draft synthesis or report — factual accuracy, hallucination risk, citation integrity, and source quality. Also usable as a one-shot brief in ChatGPT Deep Research, Gemini Deep Research, and Claude Research.
- **[red-team-analyst.md](red-team-analyst.md):** Adversarial stress test of an argument's thesis, evidence, and linchpin assumptions.

### Brief-writing tool

- **[deep-research.md](deep-research.md):** Turns an underspecified research question into a complete, well-scoped brief that a deep-research agent (Claude Research, Gemini Deep Research, ChatGPT Deep Research) can execute well. Produces the brief; does not perform the research itself.

## Inputs

Each prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: content-digest takes the work's title plus metadata; fact-checker takes a draft text plus its citation list; red-team-analyst takes the input artifact (full text or excerpt) and optional context; deep-research takes a research question or topic plus optional scoping dimensions.

## How to Use

Paste the full text of a prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the required input in your first message.

## Workflow

The three verification-pipeline prompts are designed to chain:

1. Run [content-digest.md](content-digest.md) on the work to produce a synthesis.
2. Run [fact-checker.md](fact-checker.md) on that synthesis and its citation list.
3. Run [red-team-analyst.md](red-team-analyst.md) on the original artifact or the synthesis to stress-test the thesis.

Each prompt also works standalone if you only need one stage.

[deep-research.md](deep-research.md) sits outside this chain. Use it before delegating to an external deep-research agent: paste its system instructions into a Custom GPT, Gem, or Project, hand it your research question, and it produces a brief you paste into the deep-research tool of your choice. The brief specialist does not perform research itself — the chain it participates in is *brief writer → external deep-research agent*, not *brief writer → fact-checker*.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
