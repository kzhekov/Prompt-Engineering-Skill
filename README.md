# Prompt Engineering Skills

A collection of model-agnostic prompt engineering skills for [OpenCode](https://opencode.ai) and [Claude Code](https://claude.ai/code). Designed for AI engineers who write and iterate on prompts in code — API calls, agent frameworks, system prompts, and multi-step pipelines.

## Skills

The collection is structured as one router skill and six focused sub-skills. The router handles every prompt task and loads sub-skills on demand for deeper reference.

| Skill | Description |
|-------|-------------|
| `prompt-engineering` | Router — entry point for all prompt tasks. Covers the core workflow, model selection guidance, a technique quick-reference table, and a full design checklist and debugging table. |
| `prompt-clarity-and-structure` | Vague prompts and inconsistent output. Covers clarity, specificity, structured delimiters, role/persona assignment, few-shot prompting, and output consistency. |
| `prompt-reasoning-and-chaining` | Multi-step reasoning and complex workflow decomposition. Covers chain-of-thought levels, structured reasoning, reasoning model caveats, prompt chaining patterns, inter-step validation, and shared concept alignment. |
| `prompt-grounding-and-rag` | Hallucinations and RAG pipelines. Covers quote-then-answer, source restriction, confidence calibration, long context handling, lost-in-the-middle, chunking strategies, and reranking. |
| `prompt-agents-and-tools` | Tool-using agents and multi-agent systems. Covers tool descriptions, schema design, tool selection, multi-tool orchestration, error handling, context window management, termination conditions, state management, and handoff patterns. |
| `prompt-security-and-production` | Production deployment. Covers prompt injection defense, sandwich defense, canary tokens, data-borne injection, provider portability, negative constraint pitfalls, prompt caching, and multimodal prompting. |
| `prompt-evaluation` | Measuring and iterating on prompt quality. Covers eval set design, metric selection, LLM-as-judge, A/B comparison, regression testing after model upgrades, and production monitoring. |

## Installation

### Install all skills

**OpenCode:**

```bash
for skill in prompt-engineering prompt-clarity-and-structure prompt-reasoning-and-chaining prompt-grounding-and-rag prompt-agents-and-tools prompt-security-and-production prompt-evaluation; do
  mkdir -p ~/.config/opencode/skills/$skill
  curl -fsSL https://raw.githubusercontent.com/kzhekov/prompt-engineering-skill/refs/heads/main/$skill/SKILL.md \
    -o ~/.config/opencode/skills/$skill/SKILL.md
done
```

**Claude Code:**

```bash
for skill in prompt-engineering prompt-clarity-and-structure prompt-reasoning-and-chaining prompt-grounding-and-rag prompt-agents-and-tools prompt-security-and-production prompt-evaluation; do
  mkdir -p ~/.claude/skills/$skill
  curl -fsSL https://raw.githubusercontent.com/kzhekov/prompt-engineering-skill/refs/heads/main/$skill/SKILL.md \
    -o ~/.claude/skills/$skill/SKILL.md
done
```

### Install a single skill

Replace `<skill-name>` with the skill you want (e.g. `prompt-grounding-and-rag`).

**OpenCode:**

```bash
mkdir -p ~/.config/opencode/skills/<skill-name>
curl -fsSL https://raw.githubusercontent.com/kzhekov/prompt-engineering-skill/refs/heads/main/<skill-name>/SKILL.md \
  -o ~/.config/opencode/skills/<skill-name>/SKILL.md
```

**Claude Code:**

```bash
mkdir -p ~/.claude/skills/<skill-name>
curl -fsSL https://raw.githubusercontent.com/kzhekov/prompt-engineering-skill/refs/heads/main/<skill-name>/SKILL.md \
  -o ~/.claude/skills/<skill-name>/SKILL.md
```

For Claude Code, installing into `~/.claude/skills/` makes the skill available globally across all projects. To scope it to a single project, use `.claude/skills/<skill-name>/` inside the project root instead.

Skills are picked up automatically on the next session start.

## Usage

The `prompt-engineering` router skill auto-triggers when you ask about:

- Writing or improving prompts
- System prompts or agent instructions
- Few-shot examples, chain-of-thought, prompt templates
- Making an LLM behave differently
- Prompt debugging or evaluation

You can also invoke any skill manually with `/prompt-engineering`, `/prompt-grounding-and-rag`, etc.

## License

MIT
