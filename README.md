# LLM Prompt Collections

Two main resources: a practical 2026 guide to writing your own system-level prompts, and a curated set of ready-to-use system prompts for financial planning, research, and prompt engineering.

## Featured: Writing System-Level Prompts in 2026

[guides/system-prompt-guide-2026.md](guides/system-prompt-guide-2026.md) is a hands-on guide for writing the persistent instruction prompt that defines a custom assistant — whether that's a `CLAUDE.md` file, a Gemini Gem's instructions field, or an OpenAI Custom GPT's "Instructions" box. It synthesizes 2026 guidance from Anthropic, OpenAI, Google, and Microsoft into a single workflow, with platform-specific notes where they matter.

If you're building your own assistants more than borrowing the prompts in this repo, start there.

## The Prompt Collection

The prompts here aren't generic chatbot personalities. Three principles run through the collection:

- **Strict output structure.** Most prompts mandate exact section orderings, table schemas, and disclaimer language so outputs can be diffed across runs or chained into other prompts mechanically.
- **Multi-agent workflows.** Several domains (notably `financial/` and `research/`) are designed as sequenced chains where one agent's output feeds the next, rather than as standalone single-shot prompts.
- **Platform-portable.** Each prompt targets the system-instruction field of any major assistant builder — `CLAUDE.md`, Gemini Gems, or OpenAI Custom GPTs — without rewrites.

## Contents

- [The 2026 system-prompt guide](guides/system-prompt-guide-2026.md) (`guides/`)
- [Financial prompts](financial/README.md) — tax, portfolio allocation, holistic planning, plus optional stock-picking helpers (`financial/`)
- [Meta-prompts](meta-prompts/README.md) — prompt-engineering directives, including Syntaxia (`meta-prompts/`)
- [Research prompts](research/README.md) — synthesis, fact-checking, adversarial analysis (`research/`)

## How to Use

1. Pick a prompt from the [Contents](#contents) above.
2. Paste its full text into the model's system or custom-instructions field.
3. Provide your task-specific input in the user message.
4. Validate the model's output before acting on it.

## Important Notes

- **Not financial advice.** The prompts in `financial/` are for educational and research purposes only. See the full disclaimer in [`financial/README.md`](financial/README.md) before using them.
- **Prompt behavior varies by model.** Outputs differ across model providers, versions, and runs. Always validate before acting.
- **The 2026 guide is a snapshot.** It reflects current best-practice consensus across major providers; practices evolve, and the guide will be refreshed over time.

## About

These prompts reflect a broader belief that AI should augment human judgment, not replace it — by structuring information, surfacing tradeoffs, and producing outputs people can verify and act on. The collection grows over time as new prompts and guides prove worth sharing. For more on the philosophy behind this work, see [michaelbagalman.com/philosophy.html](https://michaelbagalman.com/philosophy.html).

## License

This repository is licensed under MIT. See [LICENSE](LICENSE).
