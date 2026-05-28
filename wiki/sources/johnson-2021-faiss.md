---
title: Billion-Scale Similarity Search with GPUs
type: source
tags: faiss, vector-search, similarity-search, indexing
updated: 2026-05-28
---

**Authors:** Jeff Johnson, Matthijs Douze, Hervé Jégou  
**Affiliation:** Facebook AI Research  
**Venue:** IEEE Transactions on Big Data, 2021  
**Library:** github.com/facebookresearch/faiss (30k+ stars)

## What It Is
FAISS (Facebook AI Similarity Search) — a library for efficient similarity search and clustering of dense vectors. Enables finding the nearest neighbours among billions of vectors in milliseconds.

## Key Index Types
| Index | Accuracy | Speed | Memory | Use case |
|-------|----------|-------|--------|----------|
| `IndexFlatL2` | 100% exact | Slowest | High | Prototyping, small corpora |
| `IndexIVFFlat` | ~99% | Fast | Medium | Production, medium scale |
| `IndexIVFPQ` | ~95% | Fastest | Low | Large scale, memory-constrained |

## Why It Matters for RAG
Every production RAG system doing dense retrieval uses FAISS or a wrapper around it (Chroma, Weaviate, Pinecone all use FAISS-style indexing internally). It is the standard infrastructure for vector search.

## Used In
[[embeddings-and-vector-search]]
