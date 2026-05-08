# PromptForge — All Prompts Index

This is a central, up-to-date index of every prompt and guide in the collection. Use this page as your starting point.

## Quick Navigation

- [Prompting Best Practices Guide](#prompting-best-practices-2026)
- [Financial Prompts](#financial-prompts)
- [Research Prompts](#research-prompts)
- [Legal Prompts](#legal-prompts)
- [Meta-Prompts](#meta-prompts)

---

## Prompting Best Practices (2026)

| Prompt / Guide | File | Description | Key Features |
|---------------|------|-------------|--------------|
| Prompting Best Practices (2026) | [guides/prompting-best-practices-2026.md](guides/prompting-best-practices-2026.md) | Comprehensive guide synthesizing peer-reviewed research and official docs from Anthropic, OpenAI, Google, Microsoft. Covers chat, packaged assistants (Claude Projects, Gemini Gems, Custom GPTs), and agents. | Platform-specific notes, evaluation tips, weak-evidence flags, multi-agent strategies |

## Financial Prompts

| Prompt Name | File | Description | Key Features | Best For |
|-------------|------|-------------|--------------|----------|
| Tax Strategist | [financial/tax-strategist.md](financial/tax-strategist.md) | Specialized tax planning and optimization agent | Strict output schema, chaining-ready, disclaimers | Tax-aware financial planning |
| MPT Advisor | [financial/mpt-advisor.md](financial/mpt-advisor.md) | Modern Portfolio Theory portfolio construction and analysis | Quantitative tables, risk/return metrics | Portfolio allocation |
| Holistic Financial Planner | [financial/holistic-financial-planner.md](financial/holistic-financial-planner.md) | End-to-end financial life planning integrating tax, portfolio, cash flow, estate | Multi-agent chaining, full life-cycle view | Comprehensive personal finance |
| Compounder Analyst | [financial/compounder-analyst.md](financial/compounder-analyst.md) | Quality growth / compounding stock analysis (Graham + quality focus) | Fundamental analysis schema | Stock research |
| Graham-Dodd Analyst | [financial/graham-dodd-analyst.md](financial/graham-dodd-analyst.md) | Classic value investing analysis | Margin of safety, intrinsic value calculations | Value stock screening |

## Research Prompts

| Prompt Name | File | Description | Key Features | Best For |
|-------------|------|-------------|--------------|----------|
| Content Digest | [research/content-digest.md](research/content-digest.md) | Structured summarization and synthesis of long-form content | Key claims, evidence, counterpoints tables | Literature review, article digestion |
| Fact-Checker | [research/fact-checker.md](research/fact-checker.md) | Rigorous source verification and claim validation | Verifiability scoring, primary source preference | Truth-seeking research |
| Red-Team Analyst | [research/red-team-analyst.md](research/red-team-analyst.md) | Adversarial critique and risk identification | Assumption stress-testing, failure modes | Decision robustness |

## Legal Prompts

| Prompt Name | File | Description | Key Features | Best For |
|-------------|------|-------------|--------------|----------|
| Legalese Translator | [legal/legalese-translator.md](legal/legalese-translator.md) | Plain-English translation of legal contracts for non-lawyers | Strict output contract, structured sections, neutral tone | Understanding a contract before signing |
| ToS Reviewer | [legal/document-reviewers/tos-reviewer-prompt.md](legal/document-reviewers/tos-reviewer-prompt.md) | Severity-tiered risk review of Terms of Service | Three-tier red flag system, paired reference guide, action-oriented findings | Auditing a ToS before clicking "Agree" |
| Privacy Policy Reviewer | [legal/document-reviewers/privacy-policy-reviewer-prompt.md](legal/document-reviewers/privacy-policy-reviewer-prompt.md) | Severity-tiered review of privacy policies and data practices | Five-data-question framework, paired reference guide, real-world cross-checks | Auditing data practices before sign-up |

The two reviewer prompts pair with companion reference documents in [legal/document-reviewers/](legal/document-reviewers/) that supply the framework they apply:

- [legal/document-reviewers/privacy-policy-consumer-guide.md](legal/document-reviewers/privacy-policy-consumer-guide.md) — used by the Privacy Policy Reviewer
- [legal/document-reviewers/tos-consumer-guide.md](legal/document-reviewers/tos-consumer-guide.md) — used by the ToS Reviewer

Load the relevant guide as a knowledge file (Custom GPT, Claude Project) or paste it alongside the system prompt. See [legal/README.md](legal/README.md) for setup details.

## Meta-Prompts (Prompt Engineering)

| Prompt Name | File | Description | Key Features | Best For |
|-------------|------|-------------|--------------|----------|
| Syntaxia Prime | [meta-prompts/syntaxia-prime.md](meta-prompts/syntaxia-prime.md) | Master meta-prompt for creating high-quality system prompts | YAML schema enforcement, output strictness | Creating new prompts |
| Syntaxia Gem Architect | [meta-prompts/syntaxia-gem-architect.md](meta-prompts/syntaxia-gem-architect.md) | Specialized for building packaged assistants (Gems/Projects/GPTs) | Platform portability rules | Assistant building |
| Syntaxia Agent Forge | [meta-prompts/syntaxia-agent-forge.md](meta-prompts/syntaxia-agent-forge.md) | For designing autonomous multi-agent systems | Tool use, chaining, orchestration | Agent workflows |

---

**Notes**
- All prompts follow consistent YAML frontmatter and strict output structures for reliable chaining.
- Financial prompts include strong disclaimers — they are **educational / research tools only**, not financial advice.
- Last updated: May 2026

Feel free to suggest additions or improvements via Issues or PRs!