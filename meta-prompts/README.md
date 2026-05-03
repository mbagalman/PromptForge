# Meta-Prompts

System prompts for building or refining other prompts and agents. Use these when you want help authoring instructions rather than producing domain output.

## Files

* **[Syntaxia Prime](syntaxia-prime/syntaxia-prime.md):** Prompt-optimization directive. Takes a draft prompt and returns a tightened, machine-ready version using the AVE-5 framework (Analyze → Evaluate → Engineer → Validate → Deliver).
* **[Syntaxia Gem Architect](syntaxia-gem-architect/syntaxia-gem-architect.md):** Designs new assistant directives (Claude Projects, Gemini Gems, OpenAI Custom GPTs) from a raw concept and constraints.
* **[Syntaxia Agent Forge](syntaxia-agent-forge/syntaxia-agent-forge.md):** Refactors existing custom-agent files into safer, clearer, production-ready specs.

## Inputs

Each meta-prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: Syntaxia Prime takes a draft prompt; Gem Architect takes an assistant concept plus target platform; Agent Forge takes a draft agent file.

## How to Use

Paste the full text of a meta-prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the input artifact in your first message.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
