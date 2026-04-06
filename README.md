# Prompt Engineering Skill for OpenCode

A model-agnostic prompt engineering reference packaged as an [OpenCode](https://opencode.ai) skill. Designed for AI engineers who write and iterate on prompts in code — API calls, agent frameworks, system prompts, and multi-step pipelines.

## What it does

When triggered, this skill gives the AI assistant a structured workflow and reference covering 13 techniques for writing effective prompts, a pre-delivery checklist, and a debugging table for common failure modes. It works across all major providers: Anthropic, OpenAI, Google, Azure, and open-source models.

## Techniques covered

1. Clarity and specificity
2. Structured delimiters (XML tags, markdown)
3. Role and persona assignment
4. Few-shot (multishot) prompting
5. Chain of thought (CoT)
6. Prompt chaining
7. Grounding and hallucination reduction
8. Output consistency
9. Long context handling
10. Security and robustness (prompt injection defense)
11. Tool use and function calling
12. Multi-turn and agent patterns
13. Message placement across providers

## Installation

Copy the skill directory into your OpenCode skills folder:

```bash
mkdir -p ~/.config/opencode/skills/prompt-engineering
cp SKILL.md ~/.config/opencode/skills/prompt-engineering/SKILL.md
```

Or clone the repo directly into the skills directory:

```bash
git clone https://github.com/KamenZhekov/prompt-engineering-skill ~/.config/opencode/skills/prompt-engineering
```

OpenCode picks up skills automatically on the next session start — no further configuration required.

## Usage

The skill auto-triggers when you ask about:

- Writing or improving prompts
- System prompts or agent instructions
- Few-shot examples, chain-of-thought, prompt templates
- Making an LLM behave differently
- Prompt debugging or evaluation

You can also invoke it manually with `/prompt-engineering` in the OpenCode chat.

## License

MIT
