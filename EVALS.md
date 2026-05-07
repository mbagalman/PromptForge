# EVALS.md — Prompt Evaluation Framework

This file provides a lightweight, repeatable process for testing and improving prompts in **PromptForge**. It follows the 2026 best-practice loop described in [guides/prompting-best-practices-2026.md](guides/prompting-best-practices-2026.md).

## Why Evaluate?

Well-structured prompts still need testing. Evaluation turns "this feels good" into "this reliably meets success criteria across realistic cases."

## Core Evaluation Loop (4 Steps)

1. **Define Success Criteria**  
   Make them measurable and observable.  
   - Good: "Output always contains exactly these 6 sections in this order, with disclaimers present."  
   - Bad: "The response is helpful and professional."

2. **Build a Test Set** (5–12 cases)  
   - 4–6 **Typical cases** (common user inputs)  
   - 2–3 **Edge cases** (missing data, ambiguous requests, extreme values)  
   - 1–2 **Adversarial cases** (out-of-scope, conflicting inputs, stress tests)  
   - Optional: Cross-model tests (Claude 4, GPT-5, Gemini 2.5+)

3. **Run & Score**  
   Use a simple scoring rubric (0–2 per criterion):

   | Criterion                  | Score | Notes |
   |----------------------------|-------|-------|
   | Output structure followed  | 2     | All required sections present |
   | Disclaimers included       | 2     | Full financial disclaimer |
   | Chaining compatibility     | 1     | Output can be fed to next agent |
   | Hallucination / Fallback   | 2     | Correctly says "I don't have enough info" |
   | **Total**                  | **7/8**|       |

4. **Diagnose & Iterate**  
   Localize failures to specific sections (Role, Task, Output Format, Guardrails, etc.).  
   Avoid instruction stacking — revise the root cause instead.

## Template: Prompt Evaluation Card

Copy this for each major prompt:

```markdown
### Prompt: Holistic Financial Planner

**Success Criteria**
- [ ] Exactly 5 sections in fixed order
- [ ] Tax-aware allocation table present
- [ ] Prioritized action plan with timelines
- [ ] Full disclaimers included
- [ ] Graceful handling of missing inputs

**Test Cases** (add results below)
1. Typical: 45yo married couple, $250k income...
2. Edge: Single filer with incomplete tax data...
3. Adversarial: Request for stock picks outside scope...

**Results** (Model | Date | Score | Notes)
- Claude 4 Sonnet – May 7 2026 – 8/8 – Perfect structure
- GPT-5 – May 7 2026 – 6/8 – Missed one disclaimer
```

## Recommended Test Cases by Category

**Financial Prompts**
- Incomplete tax profile
- Conflicting goals (e.g., high growth + low risk)
- High-net-worth estate planning edge cases
- Market crash scenario

**Research Prompts**
- Conflicting sources
- Outdated information requests
- Opinion vs. fact distinction

**Meta-Prompts (Syntaxia)**
- Creating a prompt with ambiguous requirements
- Testing platform portability
- Self-referential meta tasks

## Tools & Automation Ideas

- Manual spreadsheet / Markdown table (good for v0.x)
- Simple script in `evals/` folder (future)
- Model-based judge prompts (use Syntaxia Prime to score outputs)
- Track regressions across prompt versions

## Contribution Rule

Any new prompt PR **must** include:
- Updated entry in `PROMPTS.md`
- At least one filled evaluation card in this file
- Before/after scores if the prompt was revised

---

**Last Updated**: May 2026  
**Status**: Draft for v0.1.0 — community feedback welcome!