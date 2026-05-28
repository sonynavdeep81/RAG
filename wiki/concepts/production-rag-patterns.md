---
title: Production RAG Patterns
type: concept
tags: production, rag, reranking, hierarchical, sentence-window, hyde, advanced
sources:
  - Jerry Liu 2024 — "Building Production RAG Over Complex Documents" (Databricks Summit)
  - Dr. Raj Dandekar 2024/2025 — Production Level RAG Workshop Parts 1 & 2
  - Gao et al. 2023 — "Retrieval-Augmented Generation for Large Language Models: A Survey" (IEEE TNNLS)
  - LlamaIndex documentation — Advanced Retrieval patterns
updated: 2026-05-28
---

## Summary
Basic RAG (chunk → embed → retrieve → generate) works for prototypes. Production RAG requires additional patterns to handle retrieval failures, improve precision, and work on complex documents. This file covers the patterns used in real production systems.

---

## Why Basic RAG Fails in Production

Basic RAG has three failure modes:

| Failure | Symptom | Root cause |
|---------|---------|-----------|
| **Wrong chunk retrieved** | Answer is about a different topic | Embedding similarity ≠ relevance |
| **Right chunk but incomplete** | Answer is cut off or missing steps | Chunk boundary splits critical content |
| **Right content but bad answer** | LLM ignores retrieved context | Too much noise in the retrieved chunks |

The patterns below address each failure mode specifically.

---

## Pattern 1 — Sentence-Window Retrieval

**Problem it solves:** Small chunks retrieve precisely but lack context. Large chunks have context but retrieve imprecisely.

**The insight:** Use small chunks for retrieval, but pass a larger window of surrounding text to the LLM.

```
Index:    [chunk1] [chunk2] [chunk3*] [chunk4] [chunk5]
                              ↑ retrieved

Pass to LLM: [chunk2] [chunk3] [chunk4]  ← window of 3 around the retrieved chunk
```

**How it works:**
1. Index small chunks (e.g. 1–2 sentences each)
2. When a chunk is retrieved, fetch its neighbours (±1 or ±2 chunks on each side)
3. Pass the expanded window to the LLM, not just the retrieved chunk

**Result:** Retrieval precision of small chunks + context richness of large chunks.

```python
# LlamaIndex implementation
from llama_index.core.node_parser import SentenceWindowNodeParser
from llama_index.core.postprocessor import MetadataReplacementPostProcessor

# At index time: parse into sentence-level nodes with window metadata
parser = SentenceWindowNodeParser.from_defaults(window_size=3)

# At query time: replace retrieved sentence with its surrounding window
postprocessor = MetadataReplacementPostProcessor(target_metadata_key="window")
```

**When to use:** Documents where individual sentences lack context (technical manuals, protocols). Especially useful when users ask follow-up questions that require surrounding context.

---

## Pattern 2 — Hierarchical / Parent-Child Chunking

**Problem it solves:** A question might need paragraph-level context but only a sentence matches the query.

**The insight:** Index at multiple granularities simultaneously. Retrieve small (leaf) chunks for precision; return their parent (larger chunk) to the LLM for context.

```
Document
├── Section 1 (parent chunk — 512 tokens)
│   ├── Paragraph 1.1 (leaf chunk — 128 tokens)  ← retrieval target
│   ├── Paragraph 1.2 (leaf chunk — 128 tokens)
│   └── Paragraph 1.3 (leaf chunk — 128 tokens)
└── Section 2 (parent chunk — 512 tokens)
    ├── Paragraph 2.1 (leaf chunk — 128 tokens)
    └── ...
```

When Paragraph 1.1 is retrieved → pass Section 1 (the parent) to the LLM.

```python
from llama_index.core.node_parser import HierarchicalNodeParser, get_leaf_nodes
from llama_index.core.retrievers import AutoMergingRetriever

# Build hierarchy: 2048 → 512 → 128 tokens
parser = HierarchicalNodeParser.from_defaults(chunk_sizes=[2048, 512, 128])
nodes = parser.get_nodes_from_documents(documents)
leaf_nodes = get_leaf_nodes(nodes)

# AutoMergingRetriever: if enough sibling leaf nodes match, return their parent instead
retriever = AutoMergingRetriever(vector_index, storage_context, verbose=True)
```

**When to use:** Long structured documents (technical manuals, legal contracts, research papers) where section-level context is needed to interpret a specific sentence.

---

## Pattern 3 — Reranking

**Problem it solves:** The top-k chunks from vector search are ranked by embedding similarity, which is an imperfect relevance signal. A chunk with similar words may not actually answer the question.

**The insight:** Use a two-stage retrieval process:
1. **Stage 1 (recall):** Retrieve top-50 candidates using fast vector search
2. **Stage 2 (precision):** Rerank using a slower but more accurate cross-encoder model → keep top-3

```
Query ──► Vector search (fast) ──► top-50 candidates
                                         │
                                         ▼
                                   Cross-encoder reranker
                                   (scores each query-chunk pair jointly)
                                         │
                                         ▼
                                      top-3 chunks ──► LLM
```

**Why cross-encoders are more accurate:** Bi-encoders (SBERT) encode query and chunk independently — they miss interaction between query terms and chunk content. Cross-encoders read both together, producing a much more accurate relevance score.

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

# After vector search returns 50 candidates:
query = "What is the medication dosage?"
pairs = [[query, chunk.text] for chunk in candidates]
scores = reranker.predict(pairs)

# Sort by score, keep top-3
ranked = sorted(zip(scores, candidates), reverse=True)
top_chunks = [chunk for _, chunk in ranked[:3]]
```

**Popular reranker models:**
| Model | Speed | Quality |
|-------|-------|---------|
| `cross-encoder/ms-marco-MiniLM-L-6-v2` | Fast | Good |
| `cross-encoder/ms-marco-electra-base` | Medium | Better |
| Cohere Rerank API | Fast (API) | Excellent |
| `BAAI/bge-reranker-large` | Slow | Best open source |

**When to use:** Whenever retrieval precision matters more than speed. Almost always worth adding in production.

---

## Pattern 4 — Hybrid Retrieval (Dense + Sparse)

**Problem it solves:** Dense retrieval (embeddings) finds semantic matches but misses exact keyword matches. BM25 keyword search finds exact matches but misses paraphrases.

**The insight:** Run both in parallel and merge results.

```
Query
├──► Dense retrieval (FAISS) ──► ranked list A
└──► Sparse retrieval (BM25) ──► ranked list B
         │
         ▼
   Reciprocal Rank Fusion (RRF)
   score(chunk) = Σ 1/(k + rank_in_list)
         │
         ▼
   Merged ranked list ──► LLM
```

Wang et al. 2024 found hybrid retrieval consistently outperforms either method alone across all tested domains.

```python
from langchain.retrievers import BM25Retriever, EnsembleRetriever

bm25_retriever = BM25Retriever.from_documents(docs, k=10)
dense_retriever = vector_store.as_retriever(search_kwargs={"k": 10})

hybrid_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, dense_retriever],
    weights=[0.4, 0.6]   # tune based on your domain
)
```

**When to use:** Always in production. Especially critical for technical domains with precise terminology (drug names, error codes, product IDs) where exact matching matters.

---

## Pattern 5 — HyDE (Hypothetical Document Embeddings)

**Problem it solves:** Queries are short and sparse ("what is the dosage?"). Documents are long and dense. The query embedding and the document embedding live in different parts of the vector space.

**The insight:** Instead of embedding the query directly, ask the LLM to *hallucinate* a hypothetical answer first, then embed that hypothetical answer. The hallucinated answer is stylistically similar to real document chunks — it embeds closer to them.

```
Query: "What is the maximum safe temperature for the reactor?"
    │
    ▼
LLM generates hypothetical answer (may be factually wrong, that's OK):
"The maximum safe operating temperature for a nuclear reactor is typically
 350°C for the coolant and 450°C for the fuel cladding..."
    │
    ▼
Embed the hypothetical answer (not the query)
    │
    ▼
Find chunks closest to the hypothetical answer vector
    │
    ▼
Pass real retrieved chunks + original query to LLM for final answer
```

```python
from langchain.chains import HypotheticalDocumentEmbedder
from langchain_openai import ChatOpenAI, OpenAIEmbeddings

llm = ChatOpenAI(model="gpt-4o-mini")
embeddings = OpenAIEmbeddings()

hyde_embeddings = HypotheticalDocumentEmbedder.from_llm(
    llm=llm,
    embeddings=embeddings,
    prompt_key="web_search"  # or "sci_fact", "fiqa", etc.
)
```

**When to use:** Short, vague queries where direct embedding retrieval fails. Especially useful for question-answering over technical documents.

**Warning:** Adds one LLM call per query — increases latency and cost.

---

## Pattern 6 — Multi-Query Retrieval

**Problem it solves:** A single query phrasing may miss relevant chunks that would be found by alternative phrasings.

**The insight:** Generate multiple rephrasings of the query, retrieve for each, deduplicate.

```
Original query: "side effects of Drug X"
    │
    ▼
LLM generates rephrasings:
  1. "adverse reactions to Drug X"
  2. "Drug X contraindications"
  3. "what are the risks of taking Drug X"
    │
    ▼
Retrieve for all 3 queries
    │
    ▼
Deduplicate + merge results ──► LLM
```

```python
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
retriever = MultiQueryRetriever.from_llm(
    retriever=vector_store.as_retriever(),
    llm=llm
)
```

**When to use:** When users ask ambiguous or jargon-heavy questions. When recall matters more than latency.

---

## Choosing the Right Pattern(s)

```
Is retrieval quality poor?
├── Wrong chunks returned → add Reranking (Pattern 3)
├── Missing exact keyword matches → add Hybrid Retrieval (Pattern 4)
├── Chunks lack surrounding context → use Sentence-Window (Pattern 1)
├── Query too short/vague → use HyDE (Pattern 5)
└── Single query misses variants → use Multi-Query (Pattern 6)

Is document structure complex (sections, subsections)?
└── Use Hierarchical/Parent-Child (Pattern 2)

Production baseline recommendation:
    Recursive chunking (512 tokens, 10% overlap)
    + Hybrid retrieval (dense + BM25)
    + Reranking (cross-encoder)
    This combination handles 80% of production cases well.
```

---

## Production Quality Improvement Loop

```
Deploy RAG system
    │
    ▼
Collect user queries + answers
    │
    ▼
Evaluate with RAGAS (faithfulness, relevance, precision, recall)
    │
    ├── Low faithfulness → LLM hallucinating → improve retrieval precision (reranker)
    ├── Low context precision → wrong chunks retrieved → tune chunk size or add hybrid
    ├── Low context recall → missing chunks → increase k, check chunking boundaries
    └── Low answer relevance → improve prompt template
    │
    ▼
Fix the weakest link, re-evaluate
    │
    ▼
Repeat
```

## Related
[[rag]] · [[chunking]] · [[chunking-strategies]] · [[embeddings-and-vector-search]] · [[evaluation-metrics]] · [[document-parsing]]
