---
title: Searching for Best Practices in Retrieval-Augmented Generation
type: source
tags: rag, chunking, best-practices, benchmarking
updated: 2026-05-28
---

**Authors:** Xiaohua Wang et al.  
**Venue:** EMNLP 2024 (top NLP conference)  
**DOI:** 10.18653/v1/2024.emnlp-main.981

## What It Did
Systematic empirical study testing every major RAG design choice across multiple datasets: chunking strategy, chunk size, overlap, embedding model, retriever type, reranking, and prompt format.

## Key Findings Relevant to This Wiki
| Finding | Detail |
|---------|--------|
| Chunk size is the most impactful parameter | 512 tokens is the strongest general default |
| Recursive splitting outperforms fixed-size | Respecting natural boundaries improves answer quality |
| Overlap sweet spot: 10–20% | Beyond 20% gains plateau; storage cost grows |
| Hybrid retrieval (dense + BM25) is best | Consistently outperforms either alone |
| Hit@10 is the primary retrieval benchmark metric | Used across all their experiments |

## Used In
[[chunking]] · [[chunking-strategies]] · [[chunk-overlap]] · [[evaluation-metrics]]
