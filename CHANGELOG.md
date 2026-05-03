# Changelog

This file records meaningful changes to prompts in this repository. Bump rules and entry format are defined in [`CONTRIBUTING.md`](CONTRIBUTING.md#versioning-and-changelog).

The `version` field in each prompt's frontmatter is the source of truth for the *current* version; this file is the source of truth for *what changed between versions*.

## 2026-05-03

- `meta-prompts/syntaxia-prime/syntaxia-prime.md` — 3.2.0 → 3.2.1 (patch): structural rewrite to align with the 2026 system-prompt-guide 5-section template (Role / Context / How to handle requests / Constraints / When unsure). AVE-5 methodology, BASIC/DETAIL mode selection, and all three exception protocols preserved with byte-identical response strings. HTML comment block at file top removed (now redundant with frontmatter); "NOTE TO MODEL" preamble, all-caps bracket section headers, glossary, and END OF DIRECTIVE footer also removed. No observable change to what the prompt produces given the same input. (T10 step 1 of 2 for syntaxia-prime; output methodology update is the next step.)
- **Repo-wide metadata migration** — no bump (metadata): YAML frontmatter added to all 11 prompt files per the `CONTRIBUTING.md` schema. Body content unchanged. (T7, commit `90f3764`.)
  - Affected: `financial/compounder-analyst.md`, `financial/graham-dodd-analyst.md`, `financial/holistic-financial-planner.md`, `financial/mpt-advisor.md`, `financial/tax-strategist.md`, `meta-prompts/syntaxia-prime/syntaxia-prime.md`, `meta-prompts/syntaxia-gem-architect/syntaxia-gem-architect.md`, `meta-prompts/syntaxia-agent-forge/syntaxia-agent-forge.md`, `research/content-digest.md`, `research/fact-checker.md`, `research/red-team-analyst.md`.
