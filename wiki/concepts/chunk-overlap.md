---
title: Chunk Overlap
type: concept
tags: chunking, overlap, context-preservation, rag
sources:
  - Wang et al. 2024 — "Searching for Best Practices in RAG" (EMNLP 2024)
  - LangChain Text Splitters documentation
  - Pinecone — "Chunking Strategies for LLM Applications" (2023)
updated: 2026-05-28
---

## Summary
Overlap is the number of tokens shared between the end of one chunk and the start of the next. It prevents important content from being lost at chunk boundaries — at the cost of increased storage.

---

## The Problem Overlap Solves

Without overlap, a document is cut into non-overlapping windows:

```
Document: "...Step 2: Check the valve pressure. If it exceeds 80 PSI, immediately
           release the safety pin. Step 3: Wait 30 seconds before..."

Chunk 1:  "...Step 2: Check the valve pressure. If it exceeds 80 PSI, immediately"
Chunk 2:  "release the safety pin. Step 3: Wait 30 seconds before..."
```

A user asks: *"What do I do if pressure exceeds 80 PSI?"*  
The retriever finds Chunk 1. But Chunk 1 ends with "immediately" — the action is missing.  
The LLM receives an incomplete instruction and either truncates the answer or guesses.

**Overlap makes both chunks contain the critical boundary text:**

```
Chunk 1:  "...Step 2: Check the valve pressure. If it exceeds 80 PSI, immediately
           release the safety pin."                           ← boundary text included
Chunk 2:  "If it exceeds 80 PSI, immediately release         ← boundary text repeated
           the safety pin. Step 3: Wait 30 seconds before..."
```

Now whichever chunk the retriever finds, the complete instruction is present.

---

## How Overlap Is Computed

```
chunk_size    = 512 tokens
overlap_pct   = 10%
overlap_tokens = floor(512 × 0.10) = 51 tokens
step_size     = 512 - 51 = 461 tokens

Chunk 1: tokens [0   : 512]
Chunk 2: tokens [461 : 973]    ← starts 51 tokens before Chunk 1 ends
Chunk 3: tokens [922 : 1434]   ← starts 51 tokens before Chunk 2 ends
```

The step size controls how far the window moves forward each time.  
Smaller step = more overlap = more chunks = higher storage cost.

---

## The Storage Cost

More overlap means more chunks. More chunks means more vectors to store, more memory, and slower retrieval.

| Overlap % | Step size (at 512 tokens) | Overhead vs 0% |
|-----------|--------------------------|----------------|
| 0% | 512 | baseline |
| 5% | 487 | +5.1% |
| 10% | 461 | +11.1% |
| 15% | 435 | +17.7% |
| 20% | 410 | +24.9% |
| 25% | 384 | +33.3% |

These are the theoretical (asymptotic) values for long documents. For short documents the actual overhead is lower because short documents produce 1 chunk regardless of overlap setting.

---

## What Wang et al. 2024 Found

Wang et al. (EMNLP 2024) tested overlap across multiple domains and chunk sizes:

| Finding | Detail |
|---------|--------|
| **10–20% overlap is the sweet spot** | Consistent improvement in retrieval quality; beyond 20% the gain plateaus |
| **Overlap matters more at smaller chunk sizes** | At 512 tokens most ideas fit inside one chunk; at 128 tokens boundary cuts are frequent |
| **Overlap does not hurt generation quality** | The repeated text in adjacent chunks does not confuse the LLM |
| **Too much overlap creates noise** | At 50%+ overlap, retrieved chunks become near-duplicates, reducing diversity |

**Practical recommendation from Wang et al.:** Use 10–20% overlap as a default. Start at 10%, increase to 20% if you observe retrieval failures at boundaries.

---

## The Core Tradeoff

```
LOW overlap (0–5%)                    HIGH overlap (20–30%)
──────────────────                    ─────────────────────
✅ Less storage                       ✅ Less fragmentation risk
✅ Faster retrieval                   ✅ Better boundary coverage
❌ Boundary cuts lose context         ❌ More chunks = more storage
❌ Retriever may miss split answers   ❌ Near-duplicate chunks at very high %
```

**The decision:** If storage is cheap and documents are procedural or technical → use 15–20%.  
If storage is constrained or documents are narrative → use 10%.

---

## Overlap vs Chunk Size: Which Matters More?

Wang et al. 2024 is clear: **chunk size has a larger effect on quality than overlap percentage.**

Get the chunk size right first (512 tokens is the strongest general default). Then tune overlap as a secondary adjustment. Overlap is a safety net — it catches what chunk size misses at boundaries.

```
Priority 1: Choose the right chunk size (512 tokens general default)
Priority 2: Choose the right split strategy (recursive for most cases)
Priority 3: Set overlap (10–20% as safety net)
```

---

## When Overlap Is Not Enough

Overlap works well when the critical information spans only a few sentences across a boundary. It fails when:

1. **The important content spans many paragraphs** — overlap only covers the immediate boundary, not cross-section dependencies
2. **Deeply nested conditionals** — "If A, and B, and C, then D" where A, B, C, D are spread across many tokens
3. **Tables and figures** — overlap cannot reassemble a table split across two chunks

For these cases, structure-based chunking (splitting at section boundaries) or larger chunk sizes are better solutions than higher overlap.

---

## In Code (LangChain)

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,       # tokens (approx — LangChain uses characters by default)
    chunk_overlap=51,     # ~10% of 512
    length_function=len,
)

chunks = splitter.split_text(document_text)
print(f"Total chunks: {len(chunks)}")
print(f"Chunk 1 end:   ...{chunks[0][-100:]}")
print(f"Chunk 2 start: {chunks[1][:100]}...")
# You will see ~51 characters of overlap between them
```

## Related
[[chunking]] · [[chunking-strategies]] · [[rag]] · [[embeddings]]
