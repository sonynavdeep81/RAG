---
title: RAG Evaluation Metrics
type: concept
tags: evaluation, rouge, bertscore, metrics, ragas
sources:
  - Lin 2004 — "ROUGE: A Package for Automatic Evaluation of Summaries" (ACL Workshop)
  - Zhang et al. 2020 — "BERTScore: Evaluating Text Generation with BERT" (ICLR 2020)
  - Es et al. 2024 — "RAGAS: Automated Evaluation of Retrieval Augmented Generation" (EACL 2024)
  - Wang et al. 2024 — "Searching for Best Practices in RAG" (EMNLP 2024)
updated: 2026-05-28
---

## Summary
RAG evaluation requires measuring quality at two separate layers: retrieval (did we find the right chunks?) and generation (did the LLM produce the right answer?). No single metric covers both layers. Using the wrong metric for the wrong layer leads to misleading conclusions.

---

## The Two Layers of RAG Quality

```
Layer 1 — RETRIEVAL
"Did the retriever find the right chunks?"
Metrics: Precision@k, Recall@k, MRR, Hit Rate

Layer 2 — GENERATION
"Did the LLM produce the right answer from those chunks?"
Metrics: ROUGE-L, BERTScore, Faithfulness, Answer Relevance
```

A RAG system can fail at either layer independently:
- Good retrieval + bad generation → LLM ignores the context and hallucinates anyway
- Bad retrieval + good generation → LLM gives a fluent, confident, wrong answer
- **You must measure both.**

---

## ROUGE-L (Retrieval & Generation)

**Paper:** Lin 2004 — ACL Workshop  
**Full name:** Recall-Oriented Understudy for Gisting Evaluation — Longest common subsequence variant

### What it measures
The length of the **longest common subsequence (LCS)** between the generated answer and a reference answer, expressed as an F1 score.

**Simple example:**
```
Reference: "add salt then stir for two minutes"
Generated: "stir for two minutes after adding salt"

LCS: "stir for two minutes"  (4 words in common, in order)

Precision = 4/7 = 0.57   (4 of 7 generated words matched)
Recall    = 4/6 = 0.67   (4 of 6 reference words matched)
ROUGE-L F1 = 2×(0.57×0.67)/(0.57+0.67) = 0.61
```

### What it catches
- Whether key phrases from the reference appear in the generated text
- Rough measure of information overlap

### What it misses
- Meaning — "automobile" vs "car" scores 0 despite identical meaning
- Correctness — a fluent wrong answer can score higher than an awkward right one
- Whether the retrieved chunk was actually complete (it only sees the final output)

### When to use
Best as a quick, cheap baseline metric. Always pair with BERTScore or human evaluation for real assessment.

---

## BERTScore (Generation)

**Paper:** Zhang et al. 2020 — ICLR 2020  
**How it works:** Embed every token in both the generated and reference text using a pretrained BERT model. Match each reference token to its most similar generated token. Average the similarity scores.

```
Reference tokens: ["add", "salt", "then", "stir"]
Generated tokens: ["stir", "after", "adding", "salt"]

Each reference token finds its closest match in generated:
  "add"  ↔ "adding"  → similarity 0.91
  "salt" ↔ "salt"    → similarity 1.00
  "then" ↔ "after"   → similarity 0.78
  "stir" ↔ "stir"    → similarity 1.00

BERTScore F1 = average of matched similarities
```

### What it catches
- Semantic similarity — synonyms and paraphrases score high
- Meaning-preserving rewrites that ROUGE-L misses

### What it misses
- Factual correctness — semantically similar but factually wrong answers score high
- Still operates at generation output; does not see chunk structure

### When to use
Whenever ROUGE-L is used — always run both together. BERTScore catches what ROUGE-L misses (paraphrases); ROUGE-L catches what BERTScore sometimes misses (exact phrase matching).

```python
from bert_score import score

references = ["add salt then stir for two minutes"]
generated  = ["stir for two minutes after adding salt"]

P, R, F1 = score(generated, references, lang="en")
print(f"BERTScore F1: {F1.mean():.3f}")
```

---

## Retrieval Metrics

These measure Layer 1 — whether the right chunks were retrieved — before any generation happens.

### Precision@k
Of the k chunks retrieved, what fraction are actually relevant?
$$Precision@k = \frac{\text{relevant chunks in top-}k}{k}$$

### Recall@k
Of all relevant chunks in the collection, what fraction did the retriever find in its top-k?
$$Recall@k = \frac{\text{relevant chunks in top-}k}{\text{total relevant chunks}}$$

### Hit Rate (Hit@k)
Binary: did at least one relevant chunk appear in the top-k results?
$$Hit@k = \begin{cases} 1 & \text{if any relevant chunk in top-}k \\ 0 & \text{otherwise} \end{cases}$$
Simple and widely used. Wang et al. 2024 use Hit@10 as a primary retrieval metric.

### MRR — Mean Reciprocal Rank
Average of `1/rank` where `rank` is the position of the first relevant chunk.
$$MRR = \frac{1}{|Q|} \sum_{q=1}^{|Q|} \frac{1}{\text{rank of first relevant chunk}}$$
If the first relevant chunk is always rank 1 → MRR = 1.0 (perfect). If always rank 3 → MRR = 0.33.

---

## RAGAS — Framework for End-to-End RAG Evaluation

**Paper:** Es et al. 2024 — EACL 2024  
**What it is:** A framework that evaluates a RAG system on four dimensions simultaneously, using an LLM as the judge.

| Metric | Question it answers | Layer |
|--------|-------------------|-------|
| **Faithfulness** | Is the answer grounded in the retrieved context? Does not hallucinate? | Generation |
| **Answer Relevance** | Does the answer actually address the question asked? | Generation |
| **Context Precision** | Are the retrieved chunks relevant to the question? | Retrieval |
| **Context Recall** | Does the retrieved context contain everything needed to answer? | Retrieval |

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall
from datasets import Dataset

data = {
    "question":  ["What is the dosage for Drug X?"],
    "answer":    ["500mg every 8 hours"],
    "contexts":  [["The patient must take 500mg every 8 hours."]],
    "ground_truth": ["500mg every 8 hours"]
}
dataset = Dataset.from_dict(data)
result = evaluate(dataset, metrics=[faithfulness, answer_relevancy, context_precision, context_recall])
print(result)
```

**Why RAGAS matters:** It gives you a score for each of the four failure modes of RAG. Without it, you don't know *why* your RAG system is wrong.

---

## Choosing the Right Metric

| You want to measure... | Use |
|----------------------|-----|
| Did retriever find right chunks? | Hit@k, Precision@k, MRR |
| Did generated answer match reference? | ROUGE-L + BERTScore |
| Did LLM stay faithful to context? | RAGAS Faithfulness |
| Is the answer actually relevant? | RAGAS Answer Relevance |
| Quick end-to-end benchmark | ROUGE-L (fast, cheap) |
| Thorough end-to-end evaluation | RAGAS (all four dimensions) |

---

## The Evaluation Gap in Practice

The most common mistake: **measuring only ROUGE-L and declaring success.**

ROUGE-L can be high even when:
- The LLM hallucinated but happened to use similar words to the reference
- The retrieved chunk was complete but the answer was wrong
- The answer is fluent but missing a critical step

**Minimum responsible evaluation for RAG:**
1. Hit@k — confirms retrieval is working
2. Faithfulness (RAGAS) — confirms LLM is not hallucinating
3. ROUGE-L + BERTScore — confirms answer quality vs reference

---

## Key Numbers from Wang et al. 2024

| Configuration | ROUGE-L | BERTScore F1 | Hit@10 |
|--------------|---------|-------------|--------|
| No retrieval (LLM only) | 0.213 | 0.821 | — |
| RAG with BM25 | 0.271 | 0.847 | 0.71 |
| RAG with dense retrieval | 0.318 | 0.863 | 0.84 |
| RAG with hybrid retrieval | **0.334** | **0.871** | **0.89** |

Dense + hybrid retrieval consistently outperforms keyword search across all metrics.

## Related
[[rag]] · [[embeddings-and-vector-search]] · [[chunking]]
