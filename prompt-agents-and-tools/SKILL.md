---
name: prompt-agents-and-tools
description: Load when building an agent with tool/function calling, multi-turn conversation systems, agent loops, or multi-agent handoffs. Covers tool descriptions, schema design, tool selection control, multi-tool orchestration, error handling, system prompt durability, context window management, termination conditions, state management, and handoff patterns.
---

# Prompt Agents and Tools

Detailed techniques for tool-using agents, multi-turn systems, and multi-agent architectures.

---

## 1. Tool Use and Function Calling

**Writing good tool descriptions:**
The tool description is a prompt in itself. Be specific about what the tool does, when to use it, and what it returns. Include constraints: "Use this tool ONLY for queries about order status. Do not use for returns or refunds."

Describe parameters precisely. "query: The exact SQL WHERE clause to execute" outperforms "query: The search query."

**Example:**
```json
{
  "name": "lookup_order",
  "description": "Retrieve the current status and tracking info for a customer order. Use this when the user asks about an order by ID or wants a shipping update. Do NOT use for returns or refunds.",
  "parameters": {
    "type": "object",
    "properties": {
      "order_id": {
        "type": "string",
        "description": "The order ID, formatted as ORD-XXXXX"
      }
    },
    "required": ["order_id"]
  }
}
```

**Schema design tips:**
- Use `enum` fields wherever possible to constrain choices. `enum: ["asc", "desc"]` > string described as "sort direction."
- Mark fields as required vs. optional explicitly. Models hallucinate optional parameters if the boundary is unclear.
- Keep tool count manageable. 20+ tools drops accuracy. Group related operations under fewer tools with a discriminating parameter, or use classify-then-route.

**Controlling tool selection:**
- Most APIs offer `tool_choice`: `"auto"` for general agents, `"required"` when the model must use a tool, or name a specific tool when you know which applies.
- If the model calls tools when it shouldn't (or vice versa), fix the system prompt first: "Only call a tool if you cannot answer from the conversation alone."

**Multi-tool orchestration:**
Instruct the model to plan before acting:
```
When given a task:
1. First, inside <plan> tags, list the tools you will call and in what order, and why.
2. Then execute the plan step by step.
3. After all tool calls are complete, write your final response to the user.

Do not call any tool before completing your plan.
```

**Error handling:**
Instruct the model on what to do when a tool call fails or returns unexpected/empty data. Without this, agents hallucinate results or silently proceed as if the call succeeded.
```
If a tool returns an error or no results, inform the user that the lookup failed.
Do not fabricate a response.
```

---

## 2. Multi-Turn and Agent Patterns

**System prompt durability:**
- Instructions at the beginning of a long conversation get diluted as context fills. Put critical behavioral rules both at the start and reiterated near the end of the system prompt.
- Use strong language for rules that must hold across turns: "NEVER", "ALWAYS", "Under no circumstances."
- Test your system prompt at turn 50, not just turn 1.

**Context window management:**
Every turn adds tokens. Eventually you hit the limit. Strategies:
- Truncate older turns.
- Summarize conversation history with a separate LLM call: "Summarize the conversation so far in under 300 words, preserving all decisions made and open questions."
- Use a sliding window over recent turns.
- Be aware that summarization is lossy — critical details can be dropped. For high-stakes agents, extract key facts verbatim into a structured block rather than relying on prose summaries.
- Be explicit: "The following is a summary of prior conversation. Treat it as ground truth. Do not contradict it."

**Termination conditions:**
Agents need to know when to stop. Without explicit instructions, they loop, hallucinate tasks, or ask unnecessary follow-up questions.
```
You are done when you have produced the final JSON output.
Do not ask follow-up questions.
```
For iterative agents: "After refining twice, output your best result and stop."

**State management in agent loops:**
Pass a structured state object (not just raw conversation history) to keep the model oriented:
```python
state = {"task": "...", "steps_completed": [...], "current_step": "...", "findings": [...]}
# Inject as a <state> block in each turn so the model knows where it is
```

---

## 3. Multi-Agent Handoff Patterns

For multi-agent systems, define clear handoff protocols in each agent's system prompt: what to produce, in what format, and what the next agent expects.

**Format contracts:**

```
# Agent 1 system prompt (Researcher)
You are a research agent. Your only job is to gather relevant information.
Output a JSON object with this exact structure — nothing else:
{
  "topic": "...",
  "key_facts": ["...", "..."],
  "sources": ["...", "..."],
  "gaps": ["areas where information was insufficient"]
}
Do not draft, summarize, or editorialize. Only gather and structure facts.

# Agent 2 system prompt (Writer) — receives Agent 1's JSON
You are a writing agent. You will receive a structured research payload.
Write a 3-paragraph summary based only on the key_facts provided.
If gaps are listed, acknowledge them in the final paragraph.
Input format: JSON with fields: topic, key_facts, sources, gaps.
```

Explicit format contracts between agents prevent silent mismatches where Agent 2 assumes a structure Agent 1 didn't produce.

**Error recovery between agents:**

When an upstream agent produces malformed or incomplete output, the downstream agent needs instructions for how to handle it — otherwise it hallucinates missing data or crashes silently.

```
# Agent 2 system prompt — with error recovery
You will receive a JSON research payload from a prior agent.

If the JSON is malformed or missing required fields:
1. Note the specific error in an "errors" field in your output.
2. Work with whatever valid data is present.
3. Do not fabricate data to fill gaps.

If the payload is completely unusable (empty, non-JSON, or no recognizable fields),
output: {"status": "handoff_failed", "reason": "description of what was received"}
```

For the orchestrator managing the pipeline, implement retry logic:
- Parse the downstream agent's output. If it contains `handoff_failed`, re-run the upstream agent (with a correction note appended) up to 2 times.
- If retry budget is exhausted, escalate to a fallback (human review, default response, or graceful error to the end user).

**Timeout and runaway prevention:**

Agents called in sequence can hang or loop. The orchestrator — not the agent itself — should enforce timeouts:
- Set a wall-clock timeout per agent call (e.g., 30 seconds for simple tasks, 120 seconds for research).
- Set a max-token budget per agent. If an agent is generating far more output than expected, it may be looping or producing padding.
- Set a max-calls budget for the full pipeline. If the chain exceeds N total LLM calls (including retries), terminate and return a partial result or error.

**Fallback routing:**

When an agent in a pipeline fails repeatedly or produces low-confidence output, route to an alternative:

```python
result = call_agent(research_agent, task)

if result.get("status") == "handoff_failed" or confidence_score(result) < 0.3:
    # Fallback: try a different model, a simpler prompt, or escalate
    result = call_agent(research_agent_fallback, task)

if result.get("status") == "handoff_failed":
    # Final fallback: human escalation
    return {"status": "needs_human_review", "task": task, "attempts": 2}
```

Define fallback strategy per agent based on criticality. A research agent might fall back to web search; a classification agent might fall back to a rules-based system.
