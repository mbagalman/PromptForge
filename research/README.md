# Research Prompts

System prompts for synthesis, fact-checking, and adversarial analysis. The three prompts are designed to be used together as a verification pipeline, but each works standalone.

## Files

* **[content-digest.md](content-digest.md):** Deep synthesis and critique of a user-specified work, with citations and a Read/Skim/Skip recommendation.
* **[fact-checker.md](fact-checker.md):** QA audit on a draft synthesis or report — factual accuracy, hallucination risk, citation integrity, and source quality.
* **[red-team-analyst.md](red-team-analyst.md):** Adversarial stress test of an argument's thesis, evidence, and linchpin assumptions.

## Inputs

Each prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: content-digest takes the work's title plus metadata; fact-checker takes a draft text plus its citation list; red-team-analyst takes the input artifact (full text or excerpt) and optional context.

## How to Use

Paste the full text of a prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the required input in your first message.

## Workflow

The three prompts are designed to chain:

1. Run [content-digest.md](content-digest.md) on the work to produce a synthesis.
2. Run [fact-checker.md](fact-checker.md) on that synthesis and its citation list.
3. Run [red-team-analyst.md](red-team-analyst.md) on the original artifact or the synthesis to stress-test the thesis.

Each prompt also works standalone if you only need one stage.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
