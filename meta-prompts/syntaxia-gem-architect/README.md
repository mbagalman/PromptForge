# Syntaxia Gem Architect

A meta-prompt for designing new custom assistants (Claude Projects, Gemini Gems, OpenAI Custom GPTs) from a raw concept. Output is a copy-paste-ready instruction set plus per-platform setup notes.

## Files

* **[syntaxia-gem-architect.md](syntaxia-gem-architect.md):** The architect directive. Generates production-ready system instructions for a new assistant given a purpose, target users, output format, constraints, and target platform(s).

## Inputs

Assistant purpose, target users, expected inputs, required output format, hard constraints, and target platform(s). The directive's frontmatter (`required_inputs`) lists the full set; missing items prompt the architect to ask follow-up questions before generating.

## How to Use

Paste the full contents of `syntaxia-gem-architect.md` as your assistant's instructions. In your first message, describe the assistant you want built and the constraints it must respect.

## License

Distributed under the MIT License. See [../../LICENSE](../../LICENSE) for details.
