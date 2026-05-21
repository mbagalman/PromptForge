# ROADMAP.md

This document outlines the planned enhancements and new additions to **PromptForge**. It serves as a public, living roadmap to keep the collection focused, useful, and growing in high-value directions.

Last updated: May 20, 2026
Status: Open for feedback

## Philosophy

PromptForge prioritizes **structured, chainable, platform-aware prompts** with strong disclaimers, strict output schemas, and clear best practices. New additions must align with truth-seeking, reliability, and practical usability — especially around using data effectively under uncertainty.

## Recently landed (May 2026)

Items below were on this roadmap or in close adjacency and shipped over the past few weeks. Recorded here so the roadmap retains memory of what's delivered.

- **Deep-research support, distributed across three artifacts.** What was originally scoped as a single in-repo "Deep Research Prompt Template" shipped as a different shape — a methodology guide, a brief-writing tool, and audit-tool support for deep-research outputs:
  - [`guides/deep-research-best-practices-2026.md`](guides/deep-research-best-practices-2026.md) — methodology guide synthesizing 2026 vendor guidance and research on Claude Research, Gemini Deep Research, and ChatGPT Deep Research.
  - [`research/deep-research.md`](research/deep-research.md) — Deep-Research Brief Specialist, a prompt that turns underspecified research questions into well-scoped briefs to paste into an external deep-research agent. Implements fit diagnosis, a six-component brief template, and seven analytical-posture levers.
  - [`research/fact-checker.md`](research/fact-checker.md) v1.2.0 — Source priorities, arXiv verification, Verification discipline, Severity-vs-confidence split, and `deep-research-modes` in `target_platforms`.

  The originally-proposed monolithic executor at `research/deep-research-template.md` was not built; the brief-writer + external-agent-delegation pattern was preferred because the 2026 evidence is that deep-research planners degrade under procedural scaffolding from the prompt. If a self-contained in-repo executor is still wanted (for use without an external deep-research subscription), it remains an open question — re-propose under Near-Term if so.

- **Data analysis collection.** A new [`data-analysis/`](data-analysis/) directory shipped with three prompts (autonomous data analyst, predictive-modeling engineer, predictive-model auditor) plus a folder README documenting the modeler↔auditor chain and per-prompt platform-setup requirements. This is adjacent to but distinct from the Data-Driven Decision Making Suite still on the roadmap (see below) — `data-analysis/` covers tabular EDA and ML modeling honestly; decision frameworks (probabilistic reasoning, scenario analysis, pre-mortem) remain open.

- **Beginner-prompting guide and AGENTS.md guide.** [`guides/good-prompting-for-beginners.md`](guides/good-prompting-for-beginners.md) and [`guides/agents-md-best-practices-2026.md`](guides/agents-md-best-practices-2026.md) shipped alongside the deep-research guide, expanding `guides/` from a single document to a four-guide set covering beginner chat through autonomous agent instruction files.

## Near-Term Priorities (Q2–Q3 2026)

- [ ] **Structured Coding Workflow Suite**
  End-to-end prompt sequence that forces disciplined process before any code is written:
  - Business Requirements Document (BRD)
  - Product Requirements Document (PRD)
  - Technical Specifications
  - Architecture Decision Record (ADR)
  - Implementation plan with approval gates.

  Includes orchestrator meta-prompt and agent-friendly variants. Distinct from [`guides/agents-md-best-practices-2026.md`](guides/agents-md-best-practices-2026.md), which is reference material for writing AGENTS.md / CLAUDE.md files — the suite would be the workflow prompts an engineer runs *before* an agent starts coding.
  Expected location: `development/coding-workflow-suite/` — this introduces a new top-level `development/` directory intended as the home for engineering-process prompts (workflow suites, code-review prompts that don't fit narrow `coding/`, retrospectives, and similar). Distinct from `coding/`, which is for per-language code-quality tools.

- [ ] **Reliable Medical Information Guide + Prompts**
  Responsible prompting framework for health-related queries with heavy emphasis on primary sources (PubMed, Cochrane, etc.), red-flag detection, uncertainty communication, and crystal-clear disclaimers.
  Pairs with the Fact-Checker v1.2.0 Source priorities and arXiv-verification additions.
  Expected location: `research/medical-information-guide.md` + supporting prompts.

## Mid-Term / High-Value Additions

- [ ] **Multimodal (Image & Video) Prompting Guide + Templates**
  Best practices and reusable prompt templates for image and video generation.
  Covers: style locking, consistency across frames, negative prompts, art-director workflows, prompt-to-video chaining, and effective iteration strategies.

- [ ] **Business Strategy Prompts**
  Structured prompts drawing on established strategy frameworks (e.g., Rumelt's kernel, Blue Ocean Strategy, etc.).
  Focus areas: strategy diagnosis, coherent action design, competitive advantage identification, and growth opportunity mapping.

- [ ] **Data-Driven Decision Making Suite**
  Prompts and frameworks specifically designed to help users leverage data for business decisions under varying levels of uncertainty.
  Includes: probabilistic reasoning templates, evidence weighting, scenario analysis, decision matrices with uncertainty quantification, pre-mortem analysis, and clear communication of confidence levels.
  Distinct from the just-landed [`data-analysis/`](data-analysis/) directory, which covers tabular exploratory analysis and ML modeling — this suite is about the decision framing and uncertainty communication layer that sits on top of any analysis (data-analysis output included).

## Contribution & Prioritization

Feedback and PRs are very welcome!
Priority is driven by:
1. Alignment with existing strengths (research, structure, reliability)
2. User demand / practical usefulness
3. Differentiation from generic prompt collections — especially supporting thoughtful data use in business contexts.

Feel free to open an Issue with "ROADMAP" label if you have suggestions or want to tackle any of the items above.

---

**PromptForge** — Building better prompts, one structured template at a time.
