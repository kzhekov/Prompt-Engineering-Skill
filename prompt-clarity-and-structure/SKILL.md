---
name: prompt-clarity-and-structure
description: Load when writing or fixing prompts that have vague instructions, inconsistent output format, missing delimiters, or need few-shot examples. Covers clarity, specificity, structured delimiters, role/persona assignment, few-shot prompting, and output consistency (field ordering, truncation, language alignment).
---

# Prompt Clarity and Structure

Detailed techniques for making prompts unambiguous and outputs consistent.

---

## 1. Clarity and Specificity

The single highest-leverage technique. A prompt should be followable by someone with zero context.

**Principles:**
- State the goal, audience, and success criteria explicitly.
- Specify output format, length, and style constraints up front.
- Use numbered steps for sequential processes — models follow ordered instructions more reliably than prose paragraphs.
- Replace vague verbs ("analyze", "process") with concrete actions ("extract all dates and output them as ISO 8601 strings").

**Example — vague vs. specific:**
```
# Weak
Summarize the feedback.

# Strong
Extract every piece of negative feedback from the reviews below.
For each item, output a JSON object with "quote" (verbatim text, max 20 words)
and "category" (one of: price, quality, shipping, support).
Output a JSON array and nothing else.
```

**Business and operational context:**
Models have a *default* understanding of most domains — and that default may not match yours. If your company defines "churn" as "no login in 14 days" but the model's latent understanding is "cancelled subscription," you'll get plausible-looking but wrong output.

The fix is explicit in-prompt definitions:

```
# Weak — relies on the model's default understanding
Identify churned users in the dataset.

# Strong — anchors to your operational definition
In this system, a user is considered "churned" if they have not logged in
for 14 consecutive calendar days. Do not use any other definition of churn.
```

Apply to any term with a domain-specific meaning: "conversion", "lead", "active", "resolved", "at-risk".

---

## 2. Structured Delimiters

Use delimiters to separate instructions from data and to partition complex prompts. XML-style tags work across all major models.

**Pattern:**
```xml
<context>
Background information the model needs.
</context>

<instructions>
What the model should do, step by step.
</instructions>

<input>
{{user_data}}
</input>

<output_format>
Describe the exact structure expected.
</output_format>
```

**Tips:**
- Be consistent — use the same tag names throughout and reference them in instructions ("Using only the information in <context>, ...").
- Nest tags for hierarchy with complex structures.
- For programmatic extraction, request output inside specific tags (e.g., `<r>`) for deterministic parsing.
- Tag names are arbitrary — choose names that describe their content.

---

## 3. Role / Persona Assignment

Assign the model a specific expert identity via the system message. This steers tone, depth, and domain focus.

**When to use:** Domain-specific tasks where generic responses miss the mark — legal analysis, financial modeling, code review, medical triage.

**Tips:**
- Add specificity: "Senior backend engineer specializing in distributed systems" > "software engineer".
- Role assignment is lightweight — task instruction clarity has a larger effect on output quality. Don't over-invest in persona tuning at the expense of clearer instructions.
- Pair with behavioral constraints: "You are a tax advisor. Never provide specific tax filing recommendations — only explain concepts and flag areas where the user should consult a CPA."

**Handling model refusals on legitimate tasks:**
Models sometimes refuse valid work (security analysis, content moderation, medical data processing). The fix is framing: make the role and purpose explicit so the model understands it's *classifying* harmful content, not *producing* it.

```
System: You are a content moderation system. You will receive user-submitted posts
that may contain harmful material. Your task is to classify each post by violation type.
You are not producing harmful content — you are identifying it to protect users.
```

---

## 4. Few-Shot Prompting

Provide input/output examples to demonstrate desired behavior. More effective than lengthy instructions for format-sensitive tasks.

**How many examples:** Start with 1–2, add more only when you observe inconsistency. Don't default to a fixed count — let the model's behavior tell you when you have enough.

**Principles:**
- Examples should be diverse — cover typical cases and at least one edge case.
- Keep examples representative of real inputs, not contrived toy cases.
- Wrap in clear delimiters so the model distinguishes examples from the actual task.
- **Include negative examples when fighting a recurring failure.** Show what bad output looks like, label it as wrong with an explanation, then show the correct output. Use sparingly for specific observed failure modes.

**Template:**
```xml
<examples>
  <example>
    <input>Customer says: "Your product broke after 2 days, this is unacceptable"</input>
    <o>{"sentiment": "negative", "intensity": "high", "topic": "quality"}</o>
  </example>
  <example>
    <input>Customer says: "Works fine, nothing special"</input>
    <o>{"sentiment": "neutral", "intensity": "low", "topic": "general"}</o>
  </example>
  <example>
    <!-- Edge case: sarcasm -->
    <input>Customer says: "Oh great, it stopped working again. Really impressive stuff."</input>
    <o>{"sentiment": "negative", "intensity": "high", "topic": "quality"}</o>
  </example>
  <example>
    <!-- Negative example: common failure mode -->
    <input>Customer says: "Shipping took forever but the product itself is excellent"</input>
    <incorrect_output>{"sentiment": "positive", "intensity": "high", "topic": "general"}</incorrect_output>
    <why_wrong>Ignores negative sentiment about shipping. Collapses mixed review into single polarity.</why_wrong>
    <o>{"sentiment": "mixed", "intensity": "medium", "topic": "shipping"}</o>
  </example>
</examples>

Now classify the following:
<input>{{customer_message}}</input>
```

---

## 5. Output Consistency

Techniques for reliable, predictable formatting across runs.

- **Explicit format specs** — Define the exact schema (JSON, XML, markdown template). Show the skeleton with placeholder values.
- **Few-shot examples** — Most reliable format lock-in (see above).
- **Constrained output** — Most providers offer mechanisms to constrain output format at the API level (prefill, response format, structured output mode). See `prompt-security-and-production` for provider-specific details.
- **Schema enforcement** — Use structured output / JSON mode / function calling at the API level when available.

**Field and token ordering:**
LLMs generate left-to-right; earlier tokens influence later ones. Place classification before explanation, decision before rationale, category before detail:

```json
// Weak — "category" is a post-hoc label that may contradict the already-generated "summary"
{ "summary": "The refund was processed successfully.", "category": "billing" }

// Strong — category determines the summary
{ "category": "billing", "summary": "The refund was processed successfully." }
```

**Output truncation:**
When output hits `max_tokens`, structured output breaks silently. Defenses: set `max_tokens` generously, request compact output, put critical fields early, and validate parsed output against the schema.

**Language alignment:**
The register and style of the prompt influences the output. If the prompt uses terse jargon but the expected output is customer-facing copy, the model bleeds jargon into its response. Write instructions in language close to the desired output's tone and form.

**Temperature:**
Temperature is highly model-specific. Some reasoning models ignore it entirely. Prefer structural solutions (schemas, examples, prefill) as the first fix for consistency. See `prompt-security-and-production` for provider-specific behavior.
