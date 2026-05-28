---
title: Building Production RAG Over Complex Documents
type: source
tags: rag, production, complex-documents, llamaindex, chunking, video
updated: 2026-05-28
---

**Presenter:** Jerry Liu (CEO, LlamaIndex)  
**Venue:** Databricks Data + AI Summit 2024 (major industry conference)  
**Type:** Conference talk video  
**URL:** https://www.youtube.com/watch?v=dI_TmTW9S4c  
**Published:** July 23, 2024

## Why It's Trustworthy
Jerry Liu is the co-founder and CEO of LlamaIndex — the framework used by thousands of production RAG systems globally. The Databricks Data + AI Summit is one of the largest data/AI industry conferences. This talk reflects real production experience at scale, not academic theory.

## Topics Covered
- Why simple chunking fails on complex documents (PDFs with tables, figures, embedded objects)
- Parsing strategies for structurally complex documents
- Indexing strategies beyond simple chunking — hierarchical indexing, document summaries
- The key insight: **chunks used for retrieval ≠ chunks used for LLM synthesis**
- Sentence-window retrieval: store small chunks, but pass larger surrounding context to the LLM
- Auto-merging retrieval: retrieve leaf chunks, merge to parent if enough children match
- Evaluation framework for production RAG quality

## Key Insights for This Wiki

**Insight 1 — Retrieval chunk ≠ synthesis chunk**
> "The chunk you retrieve with should not necessarily be the same chunk you pass to the LLM."
Small chunks = better retrieval precision. Large chunks = better LLM context. Use small chunks to find, large chunks to answer.

**Insight 2 — Complex document handling**
Standard recursive splitting completely fails on PDFs with tables, multi-column layouts, and figures. You need document-aware parsing (PDFMiner, Unstructured, LlamaParse) before chunking.

**Insight 3 — Hierarchical chunking**
Index at multiple granularities simultaneously. Retrieve at the fine-grained level; synthesise at the coarse level.

## Concepts This Extends
Goes beyond `[[chunking-strategies]]` into production-grade patterns not covered by academic papers:
- Sentence-window retrieval
- Auto-merging retrieval
- Hierarchical node parsing

## Used In
[[rag]] · [[chunking]] · [[chunking-strategies]] · [[embeddings-and-vector-search]]
