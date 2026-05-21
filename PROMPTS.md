# PromptForge — All Prompts Index

This is a central, up-to-date index of every prompt and guide in the collection. Use this page as your starting point.

## Quick Navigation

- [Prompting Best Practices Guide](#prompting-best-practices-2026)
- [Coding Prompts](#coding-prompts)
- [Data Analysis Prompts](#data-analysis-prompts)
- [Financial Prompts](#financial-prompts)
- [Research Prompts](#research-prompts)
- [Legal Prompts](#legal-prompts)
- [Meta-Prompts](#meta-prompts)

---

## Prompting Best Practices (2026)

| Prompt / Guide | File | Description | Key Features |
| --- | --- | --- | --- |
| Prompting Best Practices (2026) | [guides/prompting-best-practices-2026.md](guides/prompting-best-practices-2026.md) | Comprehensive guide synthesizing peer-reviewed research and official docs from Anthropic, OpenAI, Google, Microsoft. Covers chat, packaged assistants (Claude Projects, Gemini Gems, Custom GPTs), and agents. | Platform-specific notes, evaluation tips, weak-evidence flags, multi-agent strategies |
| Deep Research Best Practices (2026) | [guides/deep-research-best-practices-2026.md](guides/deep-research-best-practices-2026.md) | Focused companion on prompting deep-research agents (Claude Research, Gemini Deep Research, ChatGPT Deep Research). | Brief specification, source priorities, when to use vs. avoid, platform differences |
| Good Prompting for Beginners | [guides/good-prompting-for-beginners.md](guides/good-prompting-for-beginners.md) | Beginner-friendly habits and template for everyday chat use. | Audience/goal framing, output shape, uncertainty handling |
| AGENTS.md / CLAUDE.md Best Practices | [guides/agents-md-best-practices-2026.md](guides/agents-md-best-practices-2026.md) | Repository-level instruction files for autonomous coding agents. | Non-inferable content, progressive disclosure, permission boundaries |

## Coding Prompts

| Prompt Name | File | Description | Key Features | Best For |
| --- | --- | --- | --- | --- |
| Python Hardener | [coding/python-hardener.md](coding/python-hardener.md) | Refactors raw or legacy Python into modern, cross-platform Python 3.11+ | Five-dimension audit (cross-platform / resource / typing / exception / security), pathlib-first, PEP 604 unions, credential-to-env-var replacement, five-section output contract with Change Log | Hardening legacy scripts before deployment |
| SQL Optimization Engineer | [coding/sql-optimization-engineer.md](coding/sql-optimization-engineer.md) | Audits and refactors SQL queries for performance and maintainability across major engines | Engine-aware (Snowflake, BigQuery, Postgres, MySQL, SQL Server, Redshift), SARGability + join-cardinality + projection audit, modular CTE pattern (Import → Logical → Final), four-section output contract | Tuning a slow query or preparing SQL for code review |

## Data Analysis Prompts

| Prompt Name | File | Description | Key Features | Best For |
| --- | --- | --- | --- | --- |
| Data Analyst | [data-analysis/data-analyst.md](data-analysis/data-analyst.md) | Autonomous exploratory analysis of an uploaded tabular file | Statistically careful, description-vs-inference discipline, no manufactured insight, charts that earn their place | Understanding what's in a dataset |
| Predictive Modeling Engineer | [data-analysis/predictive-modeling.md](data-analysis/predictive-modeling.md) | Builds, evaluates, and delivers a tabular predictive model | XGBoost default with switch rules, split-before-engineering, leakage audit, stop-and-reaudit on suspicious metrics, joblib pipeline + MODEL_CARD.md deliverable | Building a tabular predictive model honestly |
| Predictive Model Auditor | [data-analysis/predictive-model-auditor.md](data-analysis/predictive-model-auditor.md) | Skeptical audit of a written model report | Three-verdict scheme, cannot-run-code constraint, pairs with the modeling-engineer's output contract | Reviewing model work before sign-off |
| Executive Insights Partner | [data-analysis/executive-insights.md](data-analysis/executive-insights.md) | Translates technical analytical findings into executive decisions, risks, and quarter-ready actions | Five-section output (Executive Summary → Strategic Alignment → Stakeholder Impact Matrix → Strategic Recommendations → Executive Talking Points), 400-word ceiling, causal-vs-correlational discipline | Briefing VP / C-suite stakeholders on an analytical finding |
| Dashboard Designer | [data-analysis/dashboard-designer.md](data-analysis/dashboard-designer.md) | Interactive five-phase conversation that produces a tool-agnostic dashboard specification | Decision-first scoping, 5–9-metric ceiling, chart-chooser logic, inverted-pyramid layout, written spec deliverable; pairs with `dashboard-design-guide.md` knowledge file | Scoping a BI dashboard before handoff to a developer |
| KPI Architect | [data-analysis/kpi-architect.md](data-analysis/kpi-architect.md) | Interactive five-phase conversation that produces a KPI specification with each KPI decomposed into operational levers | Grounded in Balanced Scorecard, Spitzer's criteria, Grove's paired-metrics rule, Marr's KPQs, and DuPont decomposition; 3–7 KPI ceiling; arithmetic-decomposition requirement; written spec deliverable | Picking KPIs for a role, team, or business and decomposing each into the factors that drive it |

The Predictive Model Auditor is designed to read reports produced by the Predictive Modeling Engineer — the modeler's output contract maps directly onto the auditor's checklist. The Executive Insights Partner sits downstream of either the analyst or the audited modeler, translating findings into business language for executive consumption. The KPI Architect and Dashboard Designer sit upstream of the analysis chain as measurement-design tools: the architect decides *what* to measure (with each KPI decomposed into its drivers), the designer decides *how to display* the resulting metrics. A natural pairing is KPI Architect → Dashboard Designer. The Dashboard Designer pairs with [data-analysis/dashboard-design-guide.md](data-analysis/dashboard-design-guide.md) (a knowledge file, not a system prompt) which must be loaded alongside it. See [data-analysis/README.md](data-analysis/README.md) for the workflow and platform-setup specifics — code execution requirements differ across the six prompts.

## Financial Prompts

| Prompt Name | File | Description | Key Features | Best For |
| --- | --- | --- | --- | --- |
| Tax Strategist | [financial/tax-strategist.md](financial/tax-strategist.md) | Specialized tax planning and optimization agent | Strict output schema, chaining-ready, disclaimers | Tax-aware financial planning |
| MPT Advisor | [financial/mpt-advisor.md](financial/mpt-advisor.md) | Modern Portfolio Theory portfolio construction and analysis | Quantitative tables, risk/return metrics | Portfolio allocation |
| Holistic Financial Planner | [financial/holistic-financial-planner.md](financial/holistic-financial-planner.md) | End-to-end financial life planning integrating tax, portfolio, cash flow, estate | Multi-agent chaining, full life-cycle view | Comprehensive personal finance |
| Compounder Analyst | [financial/compounder-analyst.md](financial/compounder-analyst.md) | Quality growth / compounding stock analysis (Graham + quality focus) | Fundamental analysis schema | Stock research |
| Graham-Dodd Analyst | [financial/graham-dodd-analyst.md](financial/graham-dodd-analyst.md) | Classic value investing analysis | Margin of safety, intrinsic value calculations | Value stock screening |

## Research Prompts

| Prompt Name | File | Description | Key Features | Best For |
| --- | --- | --- | --- | --- |
| Content Digest | [research/content-digest.md](research/content-digest.md) | Structured summarization and synthesis of long-form content | Key claims, evidence, counterpoints tables | Literature review, article digestion |
| Fact-Checker | [research/fact-checker.md](research/fact-checker.md) | Rigorous source verification and claim validation (v1.2.0) | Verifiability scoring, primary source preference, claim accumulation scan, deep-research mode support | Truth-seeking research, draft auditing |
| Red-Team Analyst | [research/red-team-analyst.md](research/red-team-analyst.md) | Adversarial critique and risk identification | Assumption stress-testing, failure modes | Decision robustness |
| Deep-Research Brief Specialist | [research/deep-research.md](research/deep-research.md) | Turns research questions into well-scoped briefs for deep-research agents (Claude Research, Gemini Deep Research, ChatGPT Deep Research) | Fit diagnosis, six-component brief template, analytical-posture levers, copy-ready output | Scoping work before delegating to a deep-research tool |

## Legal Prompts

| Prompt Name | File | Description | Key Features | Best For |
| --- | --- | --- | --- | --- |
| Legalese Translator | [legal/legalese-translator.md](legal/legalese-translator.md) | Plain-English translation of legal contracts for non-lawyers | Strict output contract, structured sections, neutral tone | Understanding a contract before signing |
| ToS Reviewer | [legal/document-reviewers/tos-reviewer-prompt.md](legal/document-reviewers/tos-reviewer-prompt.md) | Severity-tiered risk review of Terms of Service | Three-tier red flag system, paired reference guide, action-oriented findings | Auditing a ToS before clicking "Agree" |
| Privacy Policy Reviewer | [legal/document-reviewers/privacy-policy-reviewer-prompt.md](legal/document-reviewers/privacy-policy-reviewer-prompt.md) | Severity-tiered review of privacy policies and data practices | Five-data-question framework, paired reference guide, real-world cross-checks | Auditing data practices before sign-up |

The two reviewer prompts pair with companion reference documents in [legal/document-reviewers/](legal/document-reviewers/) that supply the framework they apply:

- [legal/document-reviewers/privacy-policy-consumer-guide.md](legal/document-reviewers/privacy-policy-consumer-guide.md) — used by the Privacy Policy Reviewer
- [legal/document-reviewers/tos-consumer-guide.md](legal/document-reviewers/tos-consumer-guide.md) — used by the ToS Reviewer

Load the relevant guide as a knowledge file (Custom GPT, Claude Project) or paste it alongside the system prompt. See [legal/README.md](legal/README.md) for setup details.

## Meta-Prompts (Prompt Engineering)

| Prompt Name | File | Description | Key Features | Best For |
| --- | --- | --- | --- | --- |
| Syntaxia Prime | [meta-prompts/syntaxia-prime.md](meta-prompts/syntaxia-prime.md) | Master meta-prompt for creating high-quality system prompts | YAML schema enforcement, output strictness | Creating new prompts |
| Syntaxia Gem Architect | [meta-prompts/syntaxia-gem-architect.md](meta-prompts/syntaxia-gem-architect.md) | Specialized for building packaged assistants (Gems/Projects/GPTs) | Platform portability rules | Assistant building |
| Syntaxia Agent Forge | [meta-prompts/syntaxia-agent-forge.md](meta-prompts/syntaxia-agent-forge.md) | For designing autonomous multi-agent systems | Tool use, chaining, orchestration | Agent workflows |

---

**Notes**

- All prompts follow consistent YAML frontmatter and strict output structures for reliable chaining.
- Financial prompts include strong disclaimers — they are **educational / research tools only**, not financial advice.
- Last updated: May 20, 2026

Feel free to suggest additions or improvements via Issues or PRs!