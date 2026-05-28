---
title: Production Level RAG Workshop — Part 2
type: source
tags: rag, embeddings, vector-databases, reranking, evaluation, production, workshop, video
updated: 2026-05-28
---

**Author:** Dr. Raj Dandekar (MIT Ph.D.)  
**Type:** YouTube workshop video (continuation of Part 1)  
**URL:** https://www.youtube.com/watch?v=gwAJi4d9QaE  
**Published:** September 20, 2025  
**Part 1:** [[dandekar-2024-production-rag-workshop]]

## Why It's Trustworthy
Direct continuation of Part 1 by the same author. Part 2 moves from document ingestion and chunking into the retrieval and evaluation layers of production RAG — the stages that come after chunking in the pipeline.

## Topics Covered
- **Embedding strategies in depth** — PyTorch-based embedding pipelines, model selection criteria
- **Vector databases compared** — Pinecone, Milvus, Chroma, Qdrant — when to use each
- **FAISS deep dive** — index types, approximate nearest neighbour search, scaling considerations
- **Advanced retrieval** — dense retrieval, sparse retrieval (BM25), hybrid retrieval
- **Reranking** — cross-encoder rerankers for precision after initial retrieval
- **Evaluation** — measuring RAG quality end-to-end in production settings

## How It Relates to Stage 1 Concepts
| Workshop topic | Wiki concept it extends |
|---------------|------------------------|
| Embedding strategies | [[embeddings-and-vector-search]] |
| Vector database comparison | [[embeddings-and-vector-search]] |
| Reranking | Beyond Stage 1 — covers Stage 2 territory |
| Production evaluation | [[evaluation-metrics]] |

## Key Insight
Retrieval in production is a two-stage process:
1. **Recall stage** — retrieve a large candidate set (top-50) using fast approximate search
2. **Precision stage** — rerank the candidates using a slower but more accurate cross-encoder model

This is why a single embedding similarity score is not enough for production quality.

## Used In
[[embeddings-and-vector-search]] · [[evaluation-metrics]]
