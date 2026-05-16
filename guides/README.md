# Guides

Four self-contained AI usage guides covering beginner prompting through advanced agent and deep-research workflows. All are current as of May 2026. Each draws on peer-reviewed 2026 research plus primary documentation from Anthropic, OpenAI, and Google, with weak evidence and open questions called out inline.

## Overview

These documents translate the latest consensus on prompting the current generation of frontier models from Anthropic, OpenAI, and Google into practical advice for three contexts:

- Everyday chat sessions
- Packaged assistants (Claude Projects, Gemini Gems, OpenAI Custom GPTs)
- Autonomous agents (`AGENTS.md` / `CLAUDE.md` files)

They emphasize **specification over scaffolding**: define what you want (task, output shape, constraints) rather than telling the model how to think (chain-of-thought triggers, elaborate personas, procedural steps).

## Files

- **[good-prompting-for-beginners.md](good-prompting-for-beginners.md):**  
  A concise, beginner-friendly introduction. Covers the single most important idea ("tell the AI what you want, who it's for, and what shape the answer should take"), five high-impact habits, what has changed since 2023, common anti-patterns, and a simple template. Takes ~10 minutes to read.

- **[prompting-best-practices-2026.md](prompting-best-practices-2026.md):**  
  The core reference document. A comprehensive synthesis of 2026 research and vendor guidance across chat, packaged assistants, and agents. Includes general principles (specification-style prompting, reasoning budget vs. transcript, front-loading constraints, fallback policies, evaluation loops), platform-specific notes, and explicit flags for open questions/weak evidence. Start here if you build your own assistants more than you borrow prompts.

- **[agents-md-best-practices-2026.md](agents-md-best-practices-2026.md):**  
  Deep dive into repository-level instruction files (`AGENTS.md`, `CLAUDE.md`, `SKILL.md`, etc.). Covers what belongs (exact tooling/versions, verbatim commands, counterintuitive conventions, permission boundaries), what doesn't (redundant info, long prose, marketing language), progressive disclosure patterns, and empirical findings on what actually improves agent performance.

- **[deep-research-best-practices-2026.md](deep-research-best-practices-2026.md):**  
  Focused guidance for Claude Research, Gemini Deep Research, and ChatGPT Deep Research. Covers the architectural shift (prompt seeds a planner, not a model), when deep research is and isn't the right tool, a six-component brief template, how to control source quality and recency, and how the three platforms' planning surfaces diverge.

## How to Use

- **Beginners:** Start with `good-prompting-for-beginners.md`.
- **Everyone else:** Read `prompting-best-practices-2026.md` first — it ties the other guides together.
- **Agent builders:** Follow with `agents-md-best-practices-2026.md`.
- **Heavy research users:** Add `deep-research-best-practices-2026.md`.

All guides are designed to be read standalone or as a set. They focus on principles that transfer across Claude, Gemini, and OpenAI while noting meaningful platform differences.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
