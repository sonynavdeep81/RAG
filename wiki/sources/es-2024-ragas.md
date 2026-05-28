---
title: RAGAS: Automated Evaluation of Retrieval Augmented Generation
type: source
tags: evaluation, ragas, rag, faithfulness, relevance
updated: 2026-05-28
---

**Authors:** Shahul Es, Jithin James, Luis Espinosa-Anke, Steven Schockaert  
**Venue:** EACL 2024 (European Chapter of ACL)  
**DOI:** 10.18653/v1/2024.eacl-demo.16  
**GitHub:** github.com/explodinggradients/ragas (7k+ stars)

## What It Introduced
RAGAS — a framework for reference-free evaluation of RAG systems using an LLM as judge across four complementary dimensions.

## The Four Metrics
| Metric | Layer | Question |
|--------|-------|----------|
| **Faithfulness** | Generation | Is the answer grounded in the retrieved context? (no hallucination) |
| **Answer Relevance** | Generation | Does the answer address the actual question? |
| **Context Precision** | Retrieval | Are retrieved chunks relevant to the question? |
| **Context Recall** | Retrieval | Does the context contain all information needed to answer? |

## Why It Matters
Before RAGAS, RAG evaluation required human annotation or reference answers for every query — expensive and slow. RAGAS uses an LLM to judge quality, making continuous automated evaluation practical.

## Key Limitation
LLM-as-judge inherits LLM biases. Scores can be inflated when the judge model is the same as the generation model. Best used comparatively (A vs B) rather than as absolute scores.

## Used In
[[evaluation-metrics]]
