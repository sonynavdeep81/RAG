# P6 — Production RAG

**Tagline:** "The same system used by real companies — built by you."  
**Difficulty:** Advanced  
**Concepts:** `[[production-rag-patterns]]` `[[evaluation-metrics]]` `[[embeddings-and-vector-search]]`  
**Builds on:** P5 (complex document processing)  
**Estimated time:** 6–8 hours

---

## What You Will Build
Upgrade your P5 RAG system with four production-grade patterns: hybrid retrieval (dense + BM25), a cross-encoder reranker, sentence-window retrieval, and RAGAS-based evaluation. This is the architecture used in real enterprise RAG deployments.

---

## What You Will Learn
- Why dense retrieval alone is not enough in production
- How reranking improves precision without rebuilding the index
- How sentence-window retrieval solves the context vs precision tradeoff
- How to measure a RAG system's quality across all four RAGAS dimensions
- How to identify and fix the weakest link in a RAG pipeline

---

## Requirements

### R1 — Hybrid Retrieval (Dense + BM25)
- [ ] Build a BM25 retriever over your chunks using `rank_bm25`
- [ ] Keep your existing FAISS dense retriever
- [ ] Run both retrievers for each query → retrieve top-10 from each
- [ ] Merge using Reciprocal Rank Fusion (RRF):
  ```python
  def rrf_score(rank, k=60):
      return 1 / (k + rank)
  # Final score = sum of RRF scores from both lists
  ```
- [ ] Compare: for 10 test questions, how many does dense-only get right vs hybrid?

### R2 — Cross-Encoder Reranker
- [ ] Load `cross-encoder/ms-marco-MiniLM-L-6-v2` from sentence-transformers
- [ ] After hybrid retrieval returns top-20 candidates, rerank all 20 with the cross-encoder
- [ ] Keep top-3 after reranking
- [ ] Compare Hit@3: hybrid-only vs hybrid+reranker — which is better?

### R3 — Sentence-Window Retrieval
- [ ] When indexing, store each chunk with its neighbours:
  ```python
  chunk["metadata"]["prev_chunk"] = chunks[i-1]["text"] if i > 0 else ""
  chunk["metadata"]["next_chunk"] = chunks[i+1]["text"] if i < len(chunks)-1 else ""
  ```
- [ ] At retrieval time, expand each retrieved chunk:
  ```
  context_for_llm = prev_chunk + "\n" + retrieved_chunk + "\n" + next_chunk
  ```
- [ ] Compare answer quality: retrieved chunk only vs sentence-window expanded

### R4 — RAGAS Evaluation
- [ ] Install `ragas`
- [ ] Create a test set of 10 question-answer pairs with ground truth answers
- [ ] Run RAGAS evaluation on your system:
  ```python
  from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall
  ```
- [ ] Print all four scores
- [ ] Identify the lowest score → this is your weakest link

### R5 — Weakest Link Fix
Based on your RAGAS scores:
- [ ] If **faithfulness** is low → your LLM is hallucinating → improve your prompt (instruct it to say "I don't know" when not in context)
- [ ] If **context precision** is low → wrong chunks retrieved → tune chunk size or add reranker
- [ ] If **context recall** is low → missing chunks → increase k or improve chunking
- [ ] If **answer relevance** is low → answer doesn't match question → improve prompt template
- [ ] Apply one fix, re-run RAGAS, show the before/after improvement

---

## Performance Comparison Table
At the end, fill this table for your document collection:

```
System Version       | Hit@3 | Faithfulness | Ctx Precision | Ctx Recall | Latency
─────────────────────|───────|──────────────|─────────────--|────────────|---------
P2 Basic RAG         |       |              |               |            |
P5 + Complex Docs    |       |              |               |            |
P6 + Hybrid          |       |              |               |            |
P6 + Hybrid + Rerank |       |              |               |            |
P6 + Sentence Window |       |              |               |            |
```

This table is the most important output of this project — it shows you concretely what each improvement bought.

---

## Bonus Challenges
- [ ] Add HyDE: for vague queries, generate a hypothetical answer first, embed it, retrieve with that
- [ ] Add multi-query retrieval: generate 3 rephrasings of each query, retrieve for all, deduplicate
- [ ] Add response streaming so the user sees the answer token by token

---

## Deliverable
`p6_production_rag.py` — a single runnable system with all four upgrades and a `--evaluate` flag that runs RAGAS.
```
python p6_production_rag.py --docs ./documents --evaluate
```
