---
name: prompt-evaluation
description: Load when building, running, or improving evaluations for prompts. Covers eval set design, metric selection, A/B comparison methodology, regression testing after model upgrades, and continuous monitoring of production prompts.
---

# Prompt Evaluation

Techniques for systematically measuring and improving prompt performance.

---

## 1. Eval Set Design

An eval set is a reusable collection of representative inputs paired with expected outputs or acceptance criteria. It replaces "try it a few times and eyeball it."

**Building the set:**
- Start with 10–30 examples. Cover the common case (60–70% of examples), known edge cases (20–30%), and adversarial/malformed inputs (10%).
- Source examples from real production data when possible, not invented toy cases. Synthetic examples miss the distribution of actual inputs.
- For each example, define the expected output precisely. Three levels of specificity:
  - **Exact match**: output must equal a specific string/JSON. Use for structured extraction, classification.
  - **Criteria-based**: output must satisfy a checklist (e.g., "contains a date", "sentiment is negative", "no hallucinated entities"). Use for generation tasks where wording varies.
  - **Comparative**: output must be preferred over a baseline by a human or LLM judge. Use for open-ended generation.
- Store eval sets in version control alongside the prompt. They are part of the prompt's specification.

**Maintaining the set:**
- When a production failure occurs, add the failing input to the eval set with the correct expected output. The set grows from real failures.
- Review quarterly: remove examples that no longer represent real traffic, add new patterns.

---

## 2. Metric Selection

Choose metrics that match your task type:

| Task type | Primary metrics | Notes |
|-----------|----------------|-------|
| Classification | Accuracy, precision/recall per class, confusion matrix | Break down by class — aggregate accuracy hides failures on rare classes |
| Extraction | Field-level exact match, F1 over extracted entities | Partial credit matters — "got 4 of 5 fields" is useful signal |
| Generation (factual) | Factual accuracy (human or LLM-judged), hallucination rate, citation coverage | Automated: use a second LLM call to verify claims against source |
| Generation (creative/open) | Human preference ranking, LLM-as-judge with rubric | Define the rubric explicitly — "quality" is not a metric |
| Agent/tool use | Tool selection accuracy, task completion rate, average steps to completion, error recovery rate | Track both success and efficiency |

**LLM-as-judge pattern:**
Use a separate LLM call to grade outputs against criteria. Provide the grading rubric explicitly — don't ask "is this good?"

```
You are evaluating an AI assistant's response.

<criteria>
1. Factual accuracy: all claims supported by the provided source (1-5)
2. Completeness: all parts of the question addressed (1-5)
3. Format compliance: output matches the required JSON schema (pass/fail)
</criteria>

<source>{{source_document}}</source>
<question>{{original_question}}</question>
<response>{{model_response}}</response>

Score each criterion. Output JSON: {"accuracy": int, "completeness": int, "format": "pass"|"fail", "issues": ["..."]}
```

---

## 3. A/B Comparison

When iterating on a prompt, compare the new version against the current version on the full eval set before deploying.

**Process:**
1. Run both prompts against every eval input.
2. Score both outputs using the same metric/judge.
3. Compare aggregate scores AND review individual cases where results diverge.
4. A prompt that improves aggregate score but regresses on 3 specific inputs may have introduced a new failure mode — investigate before shipping.

**Statistical significance:** For small eval sets (10-30), don't over-index on percentage differences. A 2-point swing on 20 examples is one example changing. Look at the specific cases, not just the number.

---

## 4. Regression Testing After Model Upgrades

Model updates can silently break prompts that were working. When the underlying model changes:

1. Re-run the full eval set. Compare against the last known-good baseline.
2. Investigate any regressions — they reveal where the prompt relied on model-specific behavior rather than clear instructions.
3. Fix by tightening the prompt (more explicit instructions, additional examples), not by reverting the model. The next upgrade will break it again.
4. Update the baseline scores after fixing.

---

## 5. Production Monitoring

For prompts running at scale, monitor continuously:

- **Output format compliance rate**: percentage of responses that parse correctly. A drop signals model drift or new input patterns.
- **Latency and token usage**: track per-prompt. Sudden increases may indicate the model is reasoning more (or looping).
- **User feedback signals**: thumbs up/down, escalation rate, retry rate.
- **Hallucination spot-checks**: periodically sample outputs and verify claims against sources. Automated with LLM-as-judge for scale.
- **Input drift**: monitor whether production inputs are shifting outside the distribution your eval set covers. When they do, update the eval set.
