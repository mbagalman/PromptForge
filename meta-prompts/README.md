# Meta-Prompts

System prompts for building or refining other prompts and agents — use these when you want help authoring instructions rather than producing domain output. The collection has three because optimizing a single chat instruction, designing a new persistent spec, and refactoring an existing one are genuinely different cognitive tasks. Pick by job, not by artifact type.

## Files

* **[Syntaxia Prime](syntaxia-prime/syntaxia-prime.md):** *Optimizes a single instruction.* Takes a draft prompt and returns a tightened, machine-ready version. Use when you have one user message you want sharpened for a chat session, not a persistent spec that ships with every conversation.

* **[Syntaxia Gem Architect](syntaxia-gem-architect/syntaxia-gem-architect.md):** *Designs a new spec from scratch.* Generates production-ready system instructions for a new persistent assistant (Claude Projects, Gemini Gems, OpenAI Custom GPTs, `CLAUDE.md`, etc.) given a purpose and constraints. Use when you have a concept but no draft yet.

* **[Syntaxia Agent Forge](syntaxia-agent-forge/syntaxia-agent-forge.md):** *Audits and refactors an existing spec.* Tightens an existing custom-agent file (frontmatter plus instructional body), with explicit handling for tool-permission and destructive-action concerns that greenfield design doesn't surface. Use when you have a draft already and need it hardened — especially if it gives the agent tool access.

## Inputs

Each meta-prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: Syntaxia Prime takes a draft prompt; Gem Architect takes an assistant concept plus target platform; Agent Forge takes a draft agent file.

## How to Use

Paste the full text of a meta-prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the input artifact in your first message.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
