# Legal Prompts

System prompts for working with legal documents you encounter as a consumer. The directory contains two distinct things: a standalone plain-English contract translator, and a paired set of document-review prompts that depend on companion reference guides.

## ⚠️ Not Legal Advice

These prompts are for **educational and research purposes only**. They are not a substitute for a qualified attorney. Outputs from any of these prompts can be incomplete, wrong, or out of date. For active disputes, regulated data, business contexts, or any high-stakes situation, consult a lawyer.

## Files

### Standalone

* **[legalese-translator.md](legalese-translator.md):** Translates a legal contract into plain English using a fixed output contract (overview, key terms, what to watch for, obligations, questions for your lawyer). Works on any contract type.

### Document-reviewer set

The four files in [document-reviewers/](document-reviewers/) form a coordinated set. Each reviewer prompt applies the framework defined in its companion guide, so the prompt and guide are designed to be used together.

* **[document-reviewers/tos-reviewer-prompt.md](document-reviewers/tos-reviewer-prompt.md):** Severity-tiered risk review of a Terms of Service.
* **[document-reviewers/tos-consumer-guide.md](document-reviewers/tos-consumer-guide.md):** Reference framework the ToS Reviewer applies — five risk questions, clause-by-clause analysis, three-tier red flag system.
* **[document-reviewers/privacy-policy-reviewer-prompt.md](document-reviewers/privacy-policy-reviewer-prompt.md):** Severity-tiered review of a privacy policy and the data practices it describes.
* **[document-reviewers/privacy-policy-consumer-guide.md](document-reviewers/privacy-policy-consumer-guide.md):** Reference framework the Privacy Policy Reviewer applies — five data questions, section-by-section analysis, three-tier red flag system.

The two reviewers are also designed to be used together when a user has both documents from the same service: the ToS controls contract-level risks (termination, billing, disputes) while the privacy policy controls data-level risks (collection, sharing, retention).

## How to Use

### Translator

Paste the full text of [legalese-translator.md](legalese-translator.md) into your assistant builder's instructions field (Claude Projects, Gemini Gems, OpenAI Custom GPTs). Then send the contract text in your first message.

### Document reviewers

Each reviewer prompt expects its companion consumer guide to be available as a reference document. Two ways to wire that up:

1. **Knowledge file (recommended for packaged assistants).** Create the assistant with the reviewer prompt as the system instruction, then upload the matching consumer guide as a knowledge file (Custom GPT "Knowledge", Claude Project files, Gemini Gem files).
2. **Inline paste (chat sessions).** Paste the reviewer prompt as the system instruction, then paste the consumer guide as the first user message before any document to review.

The reviewers also instruct the model to fetch URLs the user provides. That requires a browsing-enabled assistant — Custom GPTs with browsing, or Claude with web search. Without that capability, the user has to paste the document text directly.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
