# Prompt Engineering Skill

A model-agnostic prompt engineering reference packaged as a skill for [OpenCode](https://opencode.ai) and [Claude Code](https://claude.ai/code). Designed for AI engineers who write and iterate on prompts in code — API calls, agent frameworks, system prompts, and multi-step pipelines.

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

### OpenCode

Clone the repo into your OpenCode skills directory:

```bash
git clone https://github.com/kzhekov/Prompt-Engineering-Skill ~/.config/opencode/skills/prompt-engineering
```

Or copy just the skill file:

```bash
mkdir -p ~/.config/opencode/skills/prompt-engineering
curl -o ~/.config/opencode/skills/prompt-engineering/SKILL.md \
  https://raw.githubusercontent.com/kzhekov/Prompt-Engineering-Skill/main/SKILL.md
```

OpenCode picks up skills automatically on the next session start — no further configuration required.

### Claude Code

Clone the repo into your global Claude Code skills directory:

```bash
git clone https://github.com/kzhekov/Prompt-Engineering-Skill ~/.claude/skills/prompt-engineering
```

Or copy just the skill file:

```bash
mkdir -p ~/.claude/skills/prompt-engineering
curl -o ~/.claude/skills/prompt-engineering/SKILL.md \
  https://raw.githubusercontent.com/kzhekov/Prompt-Engineering-Skill/main/SKILL.md
```

This makes the skill available globally across all your projects. To install it for a single project only, use `.claude/skills/prompt-engineering/` inside the project root instead.

## Usage

The skill auto-triggers when you ask about:

- Writing or improving prompts
- System prompts or agent instructions
- Few-shot examples, chain-of-thought, prompt templates
- Making an LLM behave differently
- Prompt debugging or evaluation

You can also invoke it manually with `/prompt-engineering` in OpenCode or Claude Code.

## License

MIT
