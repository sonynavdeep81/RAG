---
title: Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks
type: source
tags: embeddings, sentence-transformers, sbert, retrieval
updated: 2026-05-28
---

**Authors:** Nils Reimers, Iryna Gurevych  
**Affiliation:** Technische Universität Darmstadt  
**Venue:** EMNLP 2019 (top NLP conference)  
**Citation count:** 6,000+  
**GitHub:** github.com/UKPLab/sentence-transformers (20k+ stars)

## What It Introduced
Sentence-BERT (SBERT) — a modification of BERT using a Siamese network to produce fixed-size sentence embeddings that can be compared with cosine similarity. This made semantic search practical at scale for the first time.

## Why It Was a Breakthrough
Before SBERT: comparing two sentences with BERT required feeding both through the model together — O(n²) complexity for n sentences. Completely impractical for search.  
After SBERT: encode each sentence independently once → store vectors → compare any query to all stored vectors in milliseconds.

## Key Numbers
- `all-MiniLM-L6-v2`: 384 dimensions, ~22MB, fastest, standard RAG default
- `all-mpnet-base-v2`: 768 dimensions, ~420MB, highest quality
- Training uses contrastive learning on NLI + STS datasets

## Used In
[[embeddings-and-vector-search]]
