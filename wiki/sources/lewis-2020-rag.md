---
title: Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks
type: source
tags: rag, retrieval, generation, foundational
updated: 2026-05-28
---

**Authors:** Patrick Lewis, Ethan Perez, Aleksandra Piktus et al.  
**Affiliation:** Facebook AI Research (FAIR)  
**Venue:** NeurIPS 2020 (top AI conference)  
**Citation count:** 5,000+ (one of the most cited NLP papers of the decade)

## What It Introduced
The original RAG architecture: a non-parametric retriever (DPR — Dense Passage Retrieval) combined with a parametric sequence-to-sequence generator (BART). The retriever and generator are trained jointly end-to-end.

## Key Contributions
1. Defined the RAG paradigm — grounding LLM generation in retrieved external documents
2. Introduced two RAG modes: RAG-Sequence (one retrieval per query) and RAG-Token (retrieve per generated token)
3. Showed RAG substantially outperforms both pure retrieval and pure generation on open-domain QA

## Key Finding
RAG achieves state-of-the-art on knowledge-intensive tasks (TriviaQA, NaturalQuestions, WebQ) without storing all facts in model weights — the knowledge is in the document index, not the model.

## Used In
[[rag]] — the entire concept page is grounded in this paper
