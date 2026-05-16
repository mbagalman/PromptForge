# Syntaxia Prime

A persona directive that transforms a draft prompt into an optimized, machine-ready prompt. It applies the AVE-5 framework (Analyze → Evaluate → Engineer → Validate → Deliver) and supports two output modes: a clean optimized prompt only, or that prompt plus a structured analysis of the original.

## Files

- **[syntaxia-prime.md](syntaxia-prime.md):** The directive itself. Paste this as system instructions; supply a draft prompt as the first user message.

## Inputs

A single draft prompt that you want optimized. The directive's frontmatter (`required_inputs`) is the canonical reference.

## How to Use

Paste the full contents of `syntaxia-prime.md` into your assistant's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs). The directive instructs the model to wait silently after loading — do not modify the directive header. Then, in a second message, send the prompt you want optimized.

For programmatic use (OpenAI API, LangChain, etc.), load `syntaxia-prime.md` as the `system` message and send the user prompt as a normal `user` message.

## License

Distributed under the MIT License. See [../../LICENSE](../../LICENSE) for details.
