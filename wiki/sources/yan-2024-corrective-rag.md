---
title: Corrective Retrieval Augmented Generation (CRAG)
type: source
tags: corrective-rag, crag, agentic-rag, relevance-evaluation, web-search
updated: 2026-05-28
---

**Authors:** Shi-Qi Yan, Jia-Chen Gu, Yun Zhu, Zhen-Hua Ling
**Affiliation:** University of Science and Technology of China
**Venue:** EMNLP Findings 2024
**arXiv:** 2401.15884
**Citation count:** 700+

## What It Introduced
**CRAG** adds a lightweight **retrieval evaluator** after standard RAG retrieval. If retrieved documents are judged irrelevant, the system automatically falls back to web search before generation. This corrects the most common RAG failure mode: the retriever returns something but it's not actually useful.

## The Three Outcomes
| Relevance score | Decision | Action |
|----------------|----------|--------|
| High (≥ 0.5) | Correct | Refine retrieved chunks, use directly |
| Low (< 0.1) | Incorrect | Discard, use web search instead |
| Middle (0.1–0.5) | Ambiguous | Combine retrieved chunks + web search results |

## The Knowledge Refinement Step
Even when retrieval is deemed correct, CRAG applies a **decompose-then-recompose** step:
1. Split retrieved documents into knowledge strips (sentence-level)
2. Score each strip for relevance to the query
3. Filter out irrelevant strips
4. Reassemble only the relevant strips for the generator

This prevents irrelevant surrounding text from confusing the LLM.

## Key Findings
| Finding | Detail |
|---------|--------|
| Consistently improves over standard RAG | Across 4 QA benchmarks |
| Plug-and-play | Works with any existing RAG pipeline — just adds the evaluator layer |
| Web fallback is effective | When document retrieval fails, web search usually finds the answer |
| Small evaluator model is sufficient | A T5-large fine-tuned classifier works as the retrieval evaluator |

## Why It Matters Practically
Most production RAG failures happen because retrieval returns a plausible-looking but irrelevant chunk, and the LLM generates a confident wrong answer from it. CRAG directly intercepts this failure mode before generation.

## Used In
[[agentic-rag]]
