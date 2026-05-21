# PromptForge: LLM Prompt Collection

Two main resources: a comprehensive 2026 guide to prompting frontier AI systems (chat, packaged assistants, and autonomous agents), and a curated set of ready-to-use system prompts for financial planning, research, and prompt engineering.

## Featured: Prompting Best Practices (2026)

[guides/prompting-best-practices-2026.md](guides/prompting-best-practices-2026.md) is a comprehensive 2026 guide to prompting frontier AI systems across three working contexts — chat sessions, packaged assistants (Gemini Gems, OpenAI Custom GPTs, Claude Projects), and autonomous agents (`AGENTS.md`, `CLAUDE.md`, `SKILL.md`). It synthesizes peer-reviewed 2026 research and primary documentation from Anthropic, OpenAI, Google, and Microsoft, with platform-specific notes and explicit flags for open questions and weak evidence.

If you're building your own assistants more than borrowing the prompts in this repo, start there.

## 📋 Prompt Index

**[PROMPTS.md](PROMPTS.md)** — Complete table of all prompts with descriptions, key features, direct file links, and usage notes.

## The Prompt Collection

The prompts here aren't generic chatbot personalities. Three principles run through the collection:

- **Strict output structure.** Most prompts mandate exact section orderings, table schemas, and disclaimer language so outputs can be diffed across runs or chained into other prompts mechanically.
- **Multi-agent workflows.** Several domains (notably `financial/` and `research/`) are designed as sequenced chains where one agent's output feeds the next, rather than as standalone single-shot prompts.
- **Platform-portable.** Each prompt targets the system-instruction field of any major assistant builder — `CLAUDE.md`, Gemini Gems, or OpenAI Custom GPTs — without rewrites.

## Contents

- [Prompting Best Practices (2026)](guides/prompting-best-practices-2026.md) (`guides/`)
- [Coding prompts](coding/README.md) — Python hardening and SQL optimization (`coding/`)
- [Coding workflow suite](development/coding-workflow-suite/README.md) — BRD → PRD → Tech Spec → ADR → Implementation Plan, with a workflow orchestrator and an optional AGENTS.md generator (`development/`)
- [Data analysis prompts](data-analysis/README.md) — KPI design (DuPont-style decomposition), exploratory analysis, predictive modeling (XGBoost), audit of model reports, executive translation, and dashboard design (`data-analysis/`)
- [Financial prompts](financial/README.md) — tax, portfolio allocation, holistic planning, plus optional stock-picking helpers (`financial/`)
- [Legal prompts](legal/README.md) — plain-English contract translator and severity-tiered ToS / privacy-policy reviewers, with companion consumer guides (`legal/`)
- [Meta-prompts](meta-prompts/README.md) — prompt-engineering directives, including Syntaxia (`meta-prompts/`)
- [Research prompts](research/README.md) — synthesis, fact-checking, adversarial analysis (`research/`)
- **[Full Prompt Index → PROMPTS.md](PROMPTS.md)**

## How to Use

1. Browse the full collection in **[PROMPTS.md](PROMPTS.md)**.
2. Pick a prompt and paste its full text into the model's system or custom-instructions field.
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