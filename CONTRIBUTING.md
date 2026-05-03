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

## Subfolder README Template

Every collection subfolder (`financial/`, `meta-prompts/`, `research/`, `guides/`, and any new categories added later) has a `README.md` that follows the skeleton below. Standardizing the structure makes the repo feel like one collection instead of several, and gives a reader a predictable place to find each kind of information.

### Standard Section Order

1. **Title** — folder name or canonical collection name as `# H1`.
2. **Purpose** — 2–3 sentences on what the collection is for and who it's for.
3. **Files** — a list of every prompt in the folder, each entry a clickable markdown link with a one-line description.
4. **Inputs** — what users need to bring before running the prompts. A short paragraph when inputs are universal across the folder, or a pointer to each prompt's `required_inputs` frontmatter (T5) when they vary.
5. **How to Use** — generic install/usage instructions. Usually one short paragraph telling readers to paste the prompt into their assistant builder's instructions field.
6. **Workflow** *(only for multi-prompt collections)* — when prompts are designed to chain, document the order and what each step produces. Omit entirely when prompts are independent.
7. **License & Disclaimer** — one-liner pointing to `../LICENSE`, plus any folder-specific disclaimer language (subject to the exception below).

### Voice and Style

- Prefer tight bullets over paragraph prose for Files and Inputs.
- Every file reference is a clickable markdown link (`[Display Name](filename.md)`); never bare filenames.
- Use the same H2 heading text across folders so the structure is scannable.
- Markdown only — no emoji unless the folder has a domain-specific reason (e.g., the financial disclaimer's existing `⚠️` glyphs, preserved verbatim).

### Disclaimer Handling

The `financial/` README contains an existing disclaimer block with four numbered items (NOT FINANCIAL ADVICE / NO GUARANTEES / USE AT YOUR OWN RISK / CONSULT A PROFESSIONAL). When applying this template to `financial/` (or any future folder with comparable legal exposure), two rules override the standard section order:

1. **Preserve the wording verbatim.** Legal language is load-bearing — do not paraphrase, condense, or "modernize" it.
2. **Keep it visually prominent.** Place the full disclaimer block right after Purpose, not at the bottom. The bottom "License & Disclaimer" section still exists, but for these folders it carries only the License pointer plus a short back-reference to the top-of-file disclaimer.

Burying high-stakes legal language at the bottom of a README is the wrong default for compliance reasons; the exception is deliberate.

### Note on `guides/`

The `guides/` folder currently contains a single guide (`system-prompt-guide-2026.md`) that serves as its own entry point. A folder-level `README.md` becomes worth adding once a second guide lands; until then, the top-level README's "Featured" and "Contents" sections cover discovery.

## Versioning and Changelog

Prompts use semver-ish version numbers in `MAJOR.MINOR.PATCH` form, recorded in each prompt's `version` frontmatter field. Changes that warrant a version bump are logged in the root [`CHANGELOG.md`](CHANGELOG.md).

### Bump Rules

- **MAJOR (X.0.0)** — The prompt's output structure changed in a way that breaks downstream consumers. Examples: removing or renaming an output section that another prompt or tool keys off of, changing the data type of an output field, dropping a required input. Anyone chaining this prompt into another should re-validate.
- **MINOR (0.X.0)** — Net-new behavior. A new output section, a new required input, expanded scope. Existing consumers continue to work but may want to take advantage of the new content.
- **PATCH (0.0.X)** — Wording, typo fixes, clarifications, structural rephrasing that doesn't change observable behavior. No consumer-observable change.

When in doubt between MINOR and PATCH, prefer MINOR — it's better to over-signal than under-signal.

### Metadata-Only Changes

Changes that touch only metadata (frontmatter fields other than `version`/`last_updated`, tags, README structure, file renames without behavior change) are recorded in `CHANGELOG.md` but do not bump the prompt's `version` field. The convention is to label these entries as "no bump" so the distinction stays visible.

### Changelog Format

Entries in `CHANGELOG.md` are grouped chronologically by date, most recent first. Each entry names the affected prompt(s), the new version (or "no bump"), the bump category (`major` / `minor` / `patch` / `metadata`), and a one-line description of what changed. Reference the commit SHA when useful.

Example:

```markdown
## 2026-06-01

- `financial/mpt-advisor.md` — 3.1.0 (minor): added Concentrated Position Stress Test section; new optional `concentration_threshold` input.
- `research/fact-checker.md` — 1.0.1 (patch): tightened wording in Risk Synthesis; no behavior change.
```

When a single change touches many prompts (e.g., a repo-wide metadata migration), one consolidated entry with an "Affected" sub-bullet is fine — see the seed entry in `CHANGELOG.md` for the pattern.

## Cross-References Between Prompts

When a prompt references another prompt by name (e.g., `holistic-financial-planner.md` references `tax-strategist.md` in its Context section), use the **bare filename** as the reference. Do not use stable IDs, slugs, or alias schemes.

This is a deliberate trade for simplicity: bare-filename references are immediately legible to anyone reading the prompt, but they do break when files are renamed.

### Renaming a Prompt

If you rename a prompt file, you are responsible for updating every reference to it. Before committing the rename, run:

```sh
git grep -n 'OLD_NAME\.md'
```

and update each hit. References can appear in:

- **Other prompt files** — most often in `## Context`, `## When unsure`, or in `required_inputs` frontmatter that mentions another prompt's output as input.
- **Subfolder READMEs** — file lists, Workflow sections.
- **Top-level README** — Featured and Contents sections.

References do **not** need to be updated in:

- **`CHANGELOG.md`** — historical entries reference filenames at time of commit and are intentionally frozen.
- **Prior commit messages, closed pull requests, prior tickets** — same reasoning.

### Why Bare Filenames?

The repo is small enough that grep-and-replace is reliable, and bare filenames have a real readability benefit: a reader sees `tax-strategist.md` and can immediately find the file. If the repo grows substantially or starts being consumed by tooling that needs stable identifiers, this policy can revisit.
