---
title: Embeddings & Vector Search
type: concept
tags: embeddings, vector-search, faiss, sentence-transformers, retrieval
sources:
  - Reimers & Gurevych 2019 — "Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks" (EMNLP 2019)
  - Karpukhin et al. 2020 — "Dense Passage Retrieval for Open-Domain QA" (EMNLP 2020)
  - Johnson et al. 2021 — "Billion-Scale Similarity Search with GPUs" (FAISS, IEEE Trans. Big Data)
  - HuggingFace Sentence Transformers documentation
updated: 2026-05-28
---

## Summary
Embeddings convert text into lists of numbers (vectors) that capture meaning. Vector search finds the most semantically similar chunks to a query by comparing these numbers — without needing exact keyword matches.

---

## What Is an Embedding?

An embedding is a translation of text into a list of numbers — called a **vector** — where the numbers encode the *meaning* of the text, not just its words.

**Simple analogy:** Imagine a map of the world. Every city has coordinates (latitude, longitude). Two cities that are geographically close have similar coordinates. Embeddings do the same thing for meaning — texts with similar meaning end up with similar coordinates in a high-dimensional space.

```
"The patient has a fever"     → [0.21, -0.54, 0.83, 0.12, ...]  (384 numbers)
"The person has high temperature" → [0.19, -0.51, 0.80, 0.14, ...]  (similar numbers!)
"How to install Python"       → [-0.43, 0.72, -0.31, 0.67, ...]  (very different numbers)
```

Two sentences about the same concept end up near each other in this number-space, even if they use completely different words. This is what allows semantic search — finding *meaning* matches, not just keyword matches.

---

## Why Embeddings Are Better Than Keyword Search

**Keyword search** (old approach):
- Query: "car engine repair"
- Finds: documents containing the words "car", "engine", "repair"
- Misses: "automobile motor maintenance" — same meaning, different words

**Embedding search** (modern approach):
- Query: "car engine repair" → convert to vector
- Finds: chunks whose vectors are closest to the query vector
- Finds: "automobile motor maintenance" — same meaning, close vectors ✅

For RAG, this matters enormously. Users ask questions in their own words. Documents are written in formal language. Keyword search fails where embedding search succeeds.

---

## Sentence-BERT: The Breakthrough (Reimers & Gurevych 2019)

Before 2019, using BERT to compare two sentences required feeding both sentences together into the model — extremely slow for search (comparing a query to millions of chunks one by one).

**Sentence-BERT (SBERT)** solved this with a **Siamese network** architecture:

```
Sentence A ──► BERT encoder ──► pooling ──► vector A ─┐
                                                        ├── cosine similarity
Sentence B ──► BERT encoder ──► pooling ──► vector B ─┘
```

Both sentences are encoded independently. This means:
- **Index time:** Encode all chunks once, store the vectors
- **Query time:** Encode only the query (fast), compare to stored vectors

This made semantic search practical at scale. SBERT is why modern RAG works.

**The model used most in RAG practice:**
`sentence-transformers/all-MiniLM-L6-v2`
- 384-dimensional vectors
- Very fast (6 layers, small model)
- Strong benchmark performance
- Free, runs locally

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')

chunks = [
    "The patient must take 500mg every 8 hours.",
    "Install Python 3.10 using the package manager.",
    "Mix flour and eggs until smooth."
]

# Encode all chunks at index time (done once)
chunk_vectors = model.encode(chunks)
print(chunk_vectors.shape)  # (3, 384) — 3 chunks, 384 numbers each

# Encode query at search time
query = "What is the medication dosage?"
query_vector = model.encode([query])  # shape: (1, 384)
```

---

## Vector Similarity — How "Closeness" Is Measured

Once you have vectors, you need a way to measure how close two vectors are.

### Cosine Similarity (most common in RAG)
Measures the **angle** between two vectors. Ignores magnitude; only cares about direction.

```
cosine_similarity(A, B) = (A · B) / (|A| × |B|)

Range: -1 (opposite meaning) to +1 (identical meaning)
```

```python
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

similarity = cosine_similarity(query_vector, chunk_vectors)
# Returns: [[0.87, 0.12, 0.08]]
# Chunk 0 (medication dosage) is closest to the query ✅
```

**Why cosine for text?** Text vectors vary in magnitude depending on sentence length. Cosine similarity normalises this away — only the direction (meaning) matters.

### Other distance metrics

| Metric | Measures | Use case |
|--------|----------|----------|
| **Cosine similarity** | Angle between vectors | Text search (most common) |
| **Dot product** | Magnitude × angle | When vectors are normalised |
| **Euclidean (L2)** | Straight-line distance | Image embeddings, lower-dim spaces |

---

## Vector Databases — Storing and Searching at Scale

A vector database is a specialised system that stores vectors and finds the nearest ones efficiently. Without it, finding the closest chunk among 1 million chunks would require 1 million individual comparisons — too slow.

### FAISS (Facebook AI Similarity Search)
**Paper:** Johnson et al. 2021 — *Billion-Scale Similarity Search with GPUs*  
**By:** Facebook AI Research  
**What it is:** A library (not a full database) for fast similarity search. Powers many production RAG systems.

```python
import faiss
import numpy as np

dimension = 384  # must match embedding model output

# Build index
index = faiss.IndexFlatL2(dimension)   # exact L2 search
index.add(chunk_vectors.astype('float32'))
print(f"Index contains {index.ntotal} vectors")

# Search
k = 3  # return top-3 most similar chunks
distances, indices = index.search(query_vector.astype('float32'), k)

print(f"Top-3 chunk indices: {indices[0]}")    # [0, 2, 1]
print(f"Distances:           {distances[0]}")  # smaller = more similar
```

FAISS types:
| Index type | Speed | Accuracy | Memory | Use when |
|------------|-------|----------|--------|----------|
| `IndexFlatL2` | Slow | 100% exact | High | Small corpora, prototyping |
| `IndexIVFFlat` | Fast | ~99% | Medium | Medium corpora |
| `IndexIVFPQ` | Fastest | ~95% | Low | Large corpora, memory constrained |

### Other popular vector stores

| Tool | Type | Best for |
|------|------|---------|
| **FAISS** | Library | Local, fast, production |
| **Chroma** | Embedded DB | Prototyping, local dev |
| **Pinecone** | Cloud service | Managed, scalable |
| **Weaviate** | Full DB | Hybrid search + metadata filtering |
| **Qdrant** | Full DB | High performance, open source |

---

## The Full Retrieval Flow

```
INDEX TIME (run once)
─────────────────────
Documents
   └─► Chunking (512 tokens, 10% overlap)
          └─► Embedding model (all-MiniLM-L6-v2)
                 └─► Vectors stored in FAISS index


QUERY TIME (run per question)
──────────────────────────────
User question: "What is the dosage for Drug X?"
   └─► Embedding model (same model as index time)
          └─► Query vector
                 └─► FAISS search → top-k=3 most similar chunk indices
                        └─► Retrieve actual chunk text
                               └─► Send question + chunks to LLM
                                      └─► Grounded answer
```

**Critical rule:** The embedding model at query time MUST be the same as at index time. If you embed chunks with Model A and queries with Model B, the vector spaces are incompatible and retrieval will fail.

---

## Dense vs Sparse Retrieval

| | Sparse (BM25, TF-IDF) | Dense (SBERT, DPR) |
|--|----------------------|-------------------|
| **How it works** | Keyword frequency matching | Vector similarity |
| **Finds synonyms** | ❌ No | ✅ Yes |
| **Finds paraphrases** | ❌ No | ✅ Yes |
| **Exact keyword match** | ✅ Strong | Weaker |
| **Speed** | Very fast | Fast with FAISS |
| **Needs training data** | No | Pre-trained models available |

**Hybrid retrieval** (used in advanced RAG): combine both. BM25 catches exact keyword matches; dense retrieval catches semantic matches. Results are merged using Reciprocal Rank Fusion (RRF).

---

## Key Numbers to Remember

| Parameter | Value | Why |
|-----------|-------|-----|
| `all-MiniLM-L6-v2` dimensions | 384 | Standard lightweight model |
| `all-mpnet-base-v2` dimensions | 768 | Higher quality, slower |
| Top-k for retrieval | 3–5 | Balances coverage vs LLM context limit |
| Cosine similarity range | -1 to +1 | 0.8+ = very similar |

---

## What Can Go Wrong

| Problem | Symptom | Fix |
|---------|---------|-----|
| Different models at index vs query time | Retrieval always wrong | Use same model both times |
| Chunk too short (< 10 tokens) | Poor embedding quality | Minimum chunk size ~20 tokens |
| Chunk too long | Embedding averages out meaning | Max ~512 tokens |
| Wrong similarity metric | Suboptimal results | Cosine for text, L2 for images |
| k too small | Misses relevant chunks | Increase k; use reranker |

## Related
[[rag]] · [[chunking]] · [[chunk-overlap]] · [[evaluation-metrics]]
