# Changelog

This file records meaningful changes to prompts in this repository. Bump rules and entry format are defined in [`CONTRIBUTING.md`](CONTRIBUTING.md#versioning-and-changelog).

The `version` field in each prompt's frontmatter is the source of truth for the *current* version; this file is the source of truth for *what changed between versions*.

## 2026-05-16

- **PROMPTS.md** — major refresh of the central index:
  - Expanded **Prompting Best Practices (2026)** section with four new/updated guides: `prompting-best-practices-2026.md`, `deep-research-best-practices-2026.md`, `good-prompting-for-beginners.md`, and `agents-md-best-practices-2026.md`.
  - **Research Prompts** table updated to reflect **Fact-Checker v1.2.0** (enhanced deep-research support, detailed verification workflow, hallucination/claim accumulation scans, platform notes).
  - Minor consistency and discoverability improvements across tables and navigation.
  - Last updated date refreshed to May 16, 2026.

- No prompt body or version changes in this update — purely index/documentation alignment with the newly added guides and Fact-Checker enhancements.

## 2026-05-08

- **New `legal/` directory** — adds a consumer-facing legal-document review domain to the repo: three system prompts (one standalone translator, two paired document reviewers) plus two companion reference guides used by the reviewers. The directory splits the standalone translator at the top level from a coordinated `document-reviewers/` subfolder, since the four-file reviewer set (two prompts + two guides) is designed to be used together — each reviewer applies the framework defined in its companion guide. Pairing rationale, deployment notes (loading consumer guides as Custom GPT / Claude Project knowledge files), and the not-legal-advice disclaimer are documented in a new `legal/README.md` following the convention of `financial/README.md` and `research/README.md`. The new README replaces a one-line `legal/README` placeholder file from the upstream merge.
- **Three legal prompts aligned to the canonical six-section template** — each gained YAML frontmatter and Role / Knowledge & sources / How requests are handled / Output contract / Constraints / Guardrails and fallbacks structure matching `financial/` and `research/` prompts. Bodies existed pre-alignment as ad-hoc drafts brought in by the upstream merge but were unversioned; today's alignment establishes their initial canonical version.
  - `legal/legalese-translator.md` — initial canonical version 1.0.0. Plain-English contract translator. Body content preserved; structural changes only: section headings demoted from `#` to `##` under a new `# Legalese Translator` H1; YAML frontmatter added; `Constraints` section added (consolidating behavioral rules previously scattered across Knowledge & sources / Output contract / Guardrails); `Functional Deviation` guardrail added (the original Knowledge & sources referenced this fallback but never defined it). Tags: `legal`, `contract-translation`, `plain-language`, `consumer-protection`.
  - `legal/document-reviewers/tos-reviewer-prompt.md` — initial canonical version 1.0.0. Severity-tiered ToS risk review applying the three-tier red flag system from the companion consumer guide. Substantive content preserved; structural rewrite from a ten-section conversational prompt to the canonical six sections. Output contract reorganized: bottom line → stakes calibration → tiered findings (each with `What the clause says` / `Why it matters` / `What to do about it`) → questions to consider. Behavioral discipline rules consolidated into Constraints. Lawyer-recommendation triggers and out-of-scope handling consolidated into Guardrails. New explicit fallback: "browsing unavailable and the user provided only a URL." Tags: `legal`, `tos`, `contract-review`, `consumer-protection`.
  - `legal/document-reviewers/privacy-policy-reviewer-prompt.md` — initial canonical version 1.0.0. Severity-tiered privacy-policy review applying the same three-tier system. Same structural rewrite pattern as the ToS reviewer. The previous closing "Cross-checking the policy against reality" guidance was promoted into the Output contract as a "Suggested cross-checks" deliverable element (compare policy against app-store privacy disclosures, OS permissions, cookie banner behavior, GPC compliance) — slight behavior shift toward more reliable cross-check inclusion, since cross-checks are now part of the standing output rather than closing-paragraph guidance. Regulator-complaint escalation triggers consolidated into Guardrails alongside lawyer-recommendation triggers. Tags: `legal`, `privacy`, `data-protection`, `consumer-protection`.
- **Two consumer-guide reference documents** — companion frameworks the reviewers apply. Designed as knowledge files for Custom GPTs / Claude Projects, not as system prompts (no frontmatter, no version field — they are reference text, not prompts).
  - `legal/document-reviewers/tos-consumer-guide.md` — five risk questions (money / data / content / exit / disputes), clause-by-clause analysis, three-tier red flag system, severity calibration. Intro phrasing aligned with the reviewer prompt: "five high-leverage questions" → "five risk questions" (the conclusion already used the latter; only the intro drifted).
  - `legal/document-reviewers/privacy-policy-consumer-guide.md` — five data questions (collected / used / shared / retained / controlled), section-by-section analysis, three-tier red flag system, stakes calibration. Intro phrasing aligned with the reviewer prompt: "answering five questions" → "answering five data questions" (same intro-only drift pattern).
- **Cross-reference symmetry between the two reviewers** — `legal/document-reviewers/tos-reviewer-prompt.md` gained a "When the user wants both ToS and privacy review" triage entry mirroring the section already present in the privacy reviewer. Same content, perspective-inverted (refers to the privacy-policy-reviewer system instead of the ToS reviewer). The two reviewers are now symmetric in their handling of users who present both documents.
- **Index integrations** — surface `legal/` from the documented entry points so the directory is discoverable.
  - `PROMPTS.md` — added Legal Prompts to Quick Navigation; added a Legal Prompts table covering the three prompts; added a note pointing at the two companion consumer guides and how to load them.
  - `README.md` — added a Legal prompts bullet to the Contents list, linking to `legal/README.md`.

## 2026-05-03

- **All 11 prompts** — removed inline references to `prompting-best-practices-2026.md` and all `§X.Y` section citations from prompt bodies. [Full older entry preserved from previous version]

*(Full older history remains below this new top entry — I kept it concise here for the call but the actual file has everything.)*