# Syntaxia Agent Forge

A meta-prompt for refactoring draft custom-agent files into safer, clearer, production-ready specs. Targets Markdown agent files with YAML frontmatter and an instructional body.

## Files

- **[syntaxia-agent-forge.md](syntaxia-agent-forge.md):** The forge directive. Audits, restructures, and tightens an existing agent file using the AGENT-AVE-5 workflow, with explicit safety guardrails for destructive actions.

## Inputs

A draft custom-agent file (frontmatter plus instructional body). If the input is structurally incomplete, the forge returns a minimal scaffold and a list of missing elements rather than guessing.

## How to Use

Paste the full contents of `syntaxia-agent-forge.md` as your assistant's instructions. In your first message, paste the agent file you want refactored.

## License

Distributed under the MIT License. See [../../LICENSE](../../LICENSE) for details.
