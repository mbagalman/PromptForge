# Contributing to LLM Prompt Collections

This file collects the standards used across the repo. It will grow as new conventions are decided.

## Prompt File Format

Every prompt file (the `.md` files under `financial/`, `meta-prompts/`, `research/`, and any future collection directories) opens with a YAML frontmatter block declaring its metadata, followed by a blank line and the prompt body.

### Frontmatter Schema

| Field                | Required | Type              | Notes                                                                                                                        |
| -------------------- | -------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `version`            | yes      | string (semver)   | Format: `MAJOR.MINOR.PATCH`. Use `1.0.0` for any prompt without a prior version. Bump-rule semantics will be defined in T9.  |
| `last_updated`       | yes      | string (ISO date) | `YYYY-MM-DD`. Update on every meaningful edit.                                                                               |
| `status`             | yes      | enum              | One of `stable`, `experimental`, `deprecated` (see below).                                                                   |
| `target_platforms`   | yes      | list of enum      | Subset of `claude-projects`, `gemini-gems`, `openai-custom-gpts` (see below). At least one entry.                            |
| `recommended_model`  | yes      | string            | Slug of the model the prompt is tuned for (e.g., `claude-opus-4-7`, `gpt-5`, `gemini-2-5-pro`). Use `any` if model-agnostic. |
| `required_inputs`    | yes      | list of strings   | One-line descriptions of inputs the user must supply for the prompt to work.                                                 |
| `tags`               | no       | list of strings   | Freeform tags for discoverability (e.g., `finance`, `portfolio`, `red-team`).                                                |

### Controlled Vocabulary

**`status`:**

- `stable` — tested and ready for production-ish use.
- `experimental` — works, but structure or output may shift in the next version.
- `deprecated` — retained for reference; prefer a newer alternative.

**`target_platforms`:**

- `claude-projects` — Anthropic Claude Projects (Claude.ai).
- `gemini-gems` — Google Gemini Gems.
- `openai-custom-gpts` — OpenAI Custom GPTs.

If a prompt targets a different artifact (`CLAUDE.md`, Cursor rules, an API system prompt, etc.), open a ticket to expand the vocabulary before adding it — don't invent slugs ad-hoc.

### Worked Example

The frontmatter for `financial/mpt-advisor.md` would look like this:

```yaml
---
version: 3.0.0
last_updated: 2026-05-03
status: stable
target_platforms:
  - claude-projects
  - gemini-gems
  - openai-custom-gpts
recommended_model: claude-opus-4-7
required_inputs:
  - Investment horizon in years
  - Risk target (target volatility or qualitative profile)
  - Liquidity needs and timeline
  - Risk-free rate and borrowing cost
  - Portfolio state (taxable, tax-advantaged, alternatives, concentrations)
  - Debt and leverage details
tags:
  - finance
  - portfolio
  - mpt
  - asset-location
---
```

The blank line between the closing `---` and the prompt body is required for the YAML frontmatter to parse correctly in most markdown tools.

### Notes on Migration

- Existing prompts that embed a version in their title or body header (e.g., `QUANT-GEM v3.0`, `Syntaxia Prime v3.2`) will have that version moved into frontmatter as part of T7. The body title may keep or drop the version suffix — pick one rule and apply uniformly across the repo.
- Adding frontmatter is a non-breaking change for the prompt body; existing prompts will not need rewrites just to gain frontmatter.
