---
title: Text Chunking
type: concept
tags: chunking, rag, segmentation, text-splitting
sources:
  - Wang et al. 2024 — "Searching for Best Practices in RAG" (EMNLP 2024)
  - Greg Kamradt — "Five Levels of Text Splitting" (2023, widely cited practitioner guide)
  - LangChain Text Splitters documentation
updated: 2026-05-28
---

## Summary
Chunking splits documents into smaller segments before indexing in a RAG system. It is the first step in the pipeline and the one that most directly determines retrieval quality — a poor chunking decision cannot be recovered downstream.

## Why Chunking Is Necessary

LLMs have a **context window** — a hard limit on how many tokens they can process at once. A typical document (a manual, a legal contract, a research paper) far exceeds this limit. More importantly, even if you could fit the whole document in, you do not want to — you want the LLM to focus on the specific passage that answers the question, not wade through irrelevant pages.

Chunking solves both problems:
1. Breaks documents into units small enough to embed and store
2. Makes retrieval precise — the system returns only the relevant piece

## The Key Parameters

| Parameter | What it controls | Typical values |
|-----------|-----------------|---------------|
| **Chunk size** | How many tokens (or characters) per chunk | 256 – 1024 tokens |
| **Overlap** | Tokens shared between adjacent chunks | 10 – 20% of chunk size |
| **Split method** | The rule for deciding where to cut | See strategies below |

These three decisions interact. There is no universally optimal setting — the right values depend on document type, query type, and the embedding model used.

## The Five Levels of Chunking (Kamradt 2023)

Greg Kamradt's framework, widely adopted by practitioners, organises strategies from simplest to most intelligent:

### Level 1 — Fixed Character Splitting (naive)
Cut every N characters regardless of content.
```
"The patient must take 500mg every 8 hours. If symptoms" | "worsen, contact a doctor immediately."
```
**Problem:** Cuts mid-sentence, mid-word, mid-step. No awareness of content at all.
**Use case:** Never recommended; only for understanding why smarter methods exist.

---

### Level 2 — Recursive Character Splitting (industry default)
Try to split at the most natural boundary first. LangChain implements this as `RecursiveCharacterTextSplitter`.

The hierarchy of separators tried in order:
1. `\n\n` — paragraph break (most preferred)
2. `\n` — line break
3. `. ` — sentence end
4. ` ` — word boundary (last resort)

The splitter tries the first separator. If the resulting chunk is still too large, it tries the next separator down. This keeps semantic units together as much as possible.

**This is the most widely used strategy in production RAG systems.**

---

### Level 3 — Document Structure Splitting
Use the document's own structure as split points — headings (`#`, `##`), section breaks, numbered lists, HTML tags, Markdown headers.

```
# Section 1: Installation
... (all content becomes one chunk) ...

# Section 2: Configuration
... (all content becomes one chunk) ...
```

**Best for:** Well-structured documents — technical manuals, Wikipedia articles, API docs, legal contracts with clear sections.  
**Problem:** Sections can be very long (too large) or very short (too small); inconsistent across documents.

---

### Level 4 — Semantic Splitting
Embed every sentence. Measure cosine similarity between adjacent sentences. Insert a chunk boundary where similarity drops significantly — meaning the topic has changed.

```
Sentence 1 ──► vector  ┐
Sentence 2 ──► vector  ├── similarity HIGH → same chunk
Sentence 3 ──► vector  ┘
Sentence 4 ──► vector  ┐── similarity LOW → new chunk starts here
Sentence 5 ──► vector  ┘
```

**Best for:** Narrative documents, research papers, long-form content where topics shift gradually.  
**Problem:** Computationally expensive (embed every sentence); threshold is hard to tune.

---

### Level 5 — Agentic / LLM-based Splitting
Use an LLM itself to decide where chunk boundaries should be. The LLM reads the document and identifies logical units — "this paragraph introduces a concept", "this section is a complete procedure".

**Best for:** Complex, unstructured documents where all other methods fail.  
**Problem:** Very expensive; slow; overkill for most use cases.

---

## What Wang et al. 2024 Found (EMNLP)

Wang et al. ran systematic experiments across strategies, chunk sizes, and document types. Key findings:

| Finding | Detail |
|---------|--------|
| **Chunk size is the most impactful parameter** | Too small → chunks lack context; too large → retrieval precision drops |
| **512 tokens is a strong general default** | Balances context richness and retrieval precision across most domains |
| **Recursive splitting outperforms fixed-size** | Respecting natural boundaries consistently improves downstream answer quality |
| **Overlap helps for long documents** | 10–20% overlap reduces the risk of cutting at a critical boundary |
| **No single strategy wins everywhere** | Document type drives the optimal choice more than any universal rule |

## The Chunk Size Tradeoff

This is the central tension in chunking:

```
SMALL chunks (e.g. 128 tokens)          LARGE chunks (e.g. 1024 tokens)
──────────────────────────────          ────────────────────────────────
+ Precise retrieval                     + Rich context per chunk
+ Retriever returns exactly             + Less risk of cutting mid-idea
  what's needed
- May cut ideas in half                 - Retriever returns too much
- Missing context for short answers       surrounding text
- More chunks to store and search       - Answer buried in noise
```

The right size depends on your documents. Short, dense documents (software docs, recipes) → smaller chunks. Long, narrative documents (manuals, reports) → larger chunks.

## A Practical Decision Guide

```
What type of document?
│
├── Well-structured (headers, numbered sections)
│   └── Use Document Structure Splitting
│
├── General prose / mixed content
│   └── Use Recursive Character Splitting (512 tokens, 10% overlap)
│
├── Long narrative where topics shift gradually
│   └── Use Semantic Splitting
│
└── Very complex / no consistent structure
    └── Use Agentic Splitting (if cost allows)
```

## What Chunking Cannot Fix
- If a complete answer spans multiple chunks with no overlap, retrieval may miss it
- Chunking cannot add information that isn't in the document
- A chunk containing the right words but the wrong relationship between them still misleads the LLM

## Related
[[rag]] · [[chunk-overlap]] · [[embeddings]] · [[evaluation-metrics]]
