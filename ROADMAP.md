# ROADMAP.md

This document outlines the planned enhancements and new additions to **PromptForge**. It serves as a public, living roadmap to keep the collection focused, useful, and growing in high-value directions.

Last updated: May 16, 2026  
Status: Draft / Open for feedback

## Philosophy
PromptForge prioritizes **structured, chainable, platform-aware prompts** with strong disclaimers, strict output schemas, and clear best practices. New additions must align with truth-seeking, reliability, and practical usability — especially around using data effectively under uncertainty.

## Near-Term Priorities (Q2–Q3 2026)

- [ ] **Deep Research Prompt Template**  
  A reusable, high-structure prompt template built directly on `guides/deep-research-best-practices-2026.md`.  
  Features: query refinement loop, source hierarchy, claim tracking, contradiction flagging, verifiability scoring, and multiple output formats (summary, detailed report, literature matrix).  
  Expected location: `research/deep-research-template.md`

- [ ] **Structured Coding Workflow Suite**  
  End-to-end prompt sequence that forces disciplined process before any code is written:  
  - Business Requirements Document (BRD)  
  - Product Requirements Document (PRD)  
  - Technical Specifications  
  - Architecture Decision Record (ADR)  
  - Implementation plan with approval gates.  
  Includes orchestrator meta-prompt and agent-friendly variants.  
  Expected location: `development/coding-workflow-suite/`

- [ ] **Reliable Medical Information Guide + Prompts**  
  Responsible prompting framework for health-related queries with heavy emphasis on primary sources (PubMed, Cochrane, etc.), red-flag detection, uncertainty communication, and crystal-clear disclaimers.  
  Pairs with Fact-Checker v1.2.0.  
  Expected location: `research/medical-information-guide.md` + supporting prompts.

## Mid-Term / High-Value Additions

- [ ] **Multimodal (Image & Video) Prompting Guide + Templates**  
  Best practices and reusable prompt templates for image and video generation.  
  Covers: style locking, consistency across frames, negative prompts, art-director workflows, prompt-to-video chaining, and effective iteration strategies.

- [ ] **Business Strategy Prompts**  
  Structured prompts drawing on established strategy frameworks (e.g., Rumelt’s kernel, Blue Ocean Strategy, etc.).  
  Focus areas: strategy diagnosis, coherent action design, competitive advantage identification, and growth opportunity mapping.

- [ ] **Data-Driven Decision Making Suite**  
  Prompts and frameworks specifically designed to help users leverage data for business decisions under varying levels of uncertainty.  
  Includes: probabilistic reasoning templates, evidence weighting, scenario analysis, decision matrices with uncertainty quantification, pre-mortem analysis, and clear communication of confidence levels.

## Contribution & Prioritization
Feedback and PRs are very welcome!  
Priority is driven by:
1. Alignment with existing strengths (research, structure, reliability)
2. User demand / practical usefulness
3. Differentiation from generic prompt collections — especially supporting thoughtful data use in business contexts.

Feel free to open an Issue with “ROADMAP” label if you have suggestions or want to tackle any of the items above.

---

**PromptForge** — Building better prompts, one structured template at a time.