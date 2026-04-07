---
name: prompt-grounding-and-rag
description: Load when the model hallucinates facts, ignores source documents, or when building a RAG pipeline. Covers hallucination reduction, quote-then-answer, source restriction, confidence calibration, long context handling, lost-in-the-middle, chunking strategies, and reranking.
---

# Prompt Grounding and RAG

Techniques for keeping the model anchored to provided facts and handling large contexts effectively.

---

## 1. Hallucination Reduction

**Permission to say "I don't know":**
Explicitly state: "If the answer is not contained in the provided context, say 'Not found in provided context' rather than guessing." This single line meaningfully reduces fabrication.

**Quote-then-answer:**
For document-based tasks, instruct the model to first extract relevant quotes, then reason only from those quotes. Forces retrieval before generation.

```
Answer the question below using only the provided document.

Step 1: Inside <quotes> tags, copy the exact sentences from the document
        that are relevant to the question. If none are relevant, write "None found."
Step 2: Inside <answer> tags, answer the question using only what you quoted.
        Do not introduce any information not present in your quotes.

<document>{{document_text}}</document>

Question: {{question}}
```

Note: the model can still selectively quote — cherry-picking passages that confirm a predetermined answer — so this reduces hallucination but doesn't eliminate motivated reasoning. Pair with source restriction for stronger grounding.

**Source restriction:**
```
You are answering questions based solely on the documents provided below.
Do not use any prior knowledge or make inferences beyond what the documents state.
If the documents do not contain enough information to answer, say so explicitly.

<documents>{{retrieved_chunks}}</documents>

Question: {{question}}
```

**Citation requirements:**
"For each claim, cite the source document and section." Makes outputs verifiable and discourages unsupported assertions.

**Self-verification:**
Add a final instruction: "Before responding, verify each factual claim against the source material. Remove any claim you cannot substantiate."

**Confidence calibration:**
Ask the model to rate confidence (1–5 or high/medium/low) for each claim. Cheap and surprisingly effective as a triage filter — especially in RAG pipelines where some retrieved chunks are only tangentially relevant. Low-confidence answers can be routed to human review or flagged.

```
For each answer, rate your confidence from 1 (uncertain, weak source support)
to 5 (certain, directly stated in source). If confidence is below 3,
prefix the answer with [LOW CONFIDENCE].
```

Note: confidence scores are rough signals, not calibrated probabilities. Useful for triage, not statistical guarantees.

**No technique eliminates hallucination entirely.** For high-stakes outputs, always validate externally.

---

## 2. Long Context Handling

For prompts with large inputs (long documents, many retrieved chunks, extensive context).

**Put documents first, query last (strong default).**
Placing the question after the context yields better results, especially with large inputs. Exception: for complex extraction with many criteria, stating criteria before the document can help the model know what to look for as it reads. Test both orderings if performance matters.

**Tag and label documents:**
```xml
<documents>
  <document id="1" source="Q3 Report">...</document>
  <document id="2" source="Competitor Analysis">...</document>
</documents>

Based on the documents above, answer: {{question}}
```

**The "lost in the middle" problem:**
Early research (2023–2024) showed models paid less attention to content in the center of long contexts. 2025-era frontier models have substantially reduced this effect. However, these mitigations remain good practice:
- Place the most critical documents or instructions at the top or bottom, not the middle.
- If you have 10 retrieved chunks and only 2–3 are truly relevant, rerank and select before injecting — reduces noise and token cost.
- For very long documents, ask the model to first scan and identify relevant sections: "First identify the section(s) most relevant to the question. Quote them in <relevant_passages> tags, then answer."

**If working with older or smaller models**, lost-in-the-middle is more pronounced and these mitigations are more critical.

---

## 3. Chunking Strategies

If content exceeds the context window, you must chunk — but chunking naively (fixed character counts) breaks semantic units.

**Prefer:**
- Chunk at natural boundaries: paragraphs, sections, or logical units.
- Include small overlap between chunks (e.g., last 100 tokens of chunk N repeated at the start of chunk N+1) to avoid losing context at cut points.
- For multiple chunks through separate calls, use a map-reduce pattern: each chunk gets its own prompt extracting relevant information, then a final prompt synthesizes results.

---

## 4. Reranking Before Injection

Retrieved chunks from RAG are often ranked by embedding similarity — which doesn't equal relevance to the actual question. Before injecting into the prompt, pass chunks through a reranker (cross-encoder or smaller model) to select the top 3–5 truly relevant chunks. Reduces noise, shrinks token cost, and avoids the model hallucinating from irrelevant content.
