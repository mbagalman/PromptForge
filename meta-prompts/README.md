# Meta-Prompts

System prompts for building or refining other prompts and agents — use these when you want help authoring instructions rather than producing domain output. The collection has three because optimizing a single chat instruction, designing or refactoring a persistent assistant spec, and designing or refactoring an agent file are genuinely different cognitive tasks. Pick by job, not by artifact type.

## Files

- **[Syntaxia Prime](syntaxia-prime/syntaxia-prime.md):** *Optimizes a single instruction.* Takes a draft prompt and returns a tightened, machine-ready version. Use when you have one user message you want sharpened for a chat session, not a persistent spec that ships with every conversation.

- **[Syntaxia Gem Architect](syntaxia-gem-architect/syntaxia-gem-architect.md):** *Designs new packaged assistants from a concept.* Generates production-ready system instructions for a new persistent assistant (Claude Projects, Gemini Gems, OpenAI Custom GPTs, `CLAUDE.md`) given a purpose and constraints. Use when the artifact is a persistent assistant *without* agent tooling (no `tools` frontmatter, executable command strings, or permission boundaries) — agent files with tooling go to Forge instead.

- **[Syntaxia Agent Forge](syntaxia-agent-forge/syntaxia-agent-forge.md):** *Designs and refactors agent files.* Handles two modes: refactoring an existing custom-agent file for clearer guardrails, or designing a brand-new agent file from a concept and constraints. Both modes enforce explicit tool-permission, trust-boundary, and destructive-action handling that greenfield assistant design doesn't surface. Use when the artifact is an agent file (frontmatter + body, with tooling) — for refactoring an existing one or building a new one from scratch.

## Inputs

Each meta-prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: Syntaxia Prime takes a draft chat instruction; Gem Architect takes an assistant concept plus target platform; Agent Forge takes either a draft agent file (Refactor mode) or an agent concept plus environment, tools, and permission boundaries (Design mode).

## How to Use

Paste the full text of a meta-prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the input artifact in your first message.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
