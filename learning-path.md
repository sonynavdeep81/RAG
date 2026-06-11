# Learning Path

**How to use:** Do stages in order. For each step: read the primary resource → discuss/ingest → write/read the concept page → only then move on. Start the paired project as soon as its "ready after" step is done — don't finish all reading before building.

## Stage 1 — RAG Foundations (build these first, in order)

| Step | File | What you will understand after | Primary resources (in reading order) |
|------|------|-------------------------------|--------------------------------------|
| 1 | `wiki/concepts/rag.md` | What RAG is, the 3-stage pipeline, what it cannot fix | ① LangChain RAG tutorial (python.langchain.com/docs/tutorials/rag) — gentle, runnable. ② Lewis et al. 2020, *RAG for Knowledge-Intensive NLP* (arXiv:2005.11401) — the origin paper. ③ Gao et al. 2023 RAG survey (arXiv:2312.10997) — skim now, reference later |
| 2 | `wiki/concepts/chunking.md` | What chunking is, the 5 levels overview, chunk size tradeoff | ① LangChain *Text splitters* concept docs. ② LlamaIndex *Node Parser / Text Splitters* docs. ③ Kamradt, *5 Levels of Text Splitting* notebook (github.com/FullStackRetrieval-com) — supplementary, source of the 5-levels framing |
| 3 | `wiki/concepts/document-parsing.md` | How to handle PDFs, tables, images, scanned docs, mixed formats | ① Unstructured docs (docs.unstructured.io) — core concepts + partitioning. ② LlamaIndex *Loading Data* docs. ③ PyMuPDF docs (RAG section) for low-level PDF control |
| 4 | `wiki/concepts/chunking-strategies.md` | Each strategy with code + tables, images, scanned PDFs | ① LangChain how-to: recursive, token, markdown/HTML header splitters. ② LlamaIndex `SemanticSplitterNodeParser` docs. ③ Chroma technical report, *Evaluating Chunking Strategies for Retrieval* — supplementary, has real numbers |
| 5 | `wiki/concepts/chunk-overlap.md` | What overlap is, storage costs, when it helps | ① LangChain `RecursiveCharacterTextSplitter` API docs (`chunk_overlap` param). ② Revisit Chroma chunking report — overlap ablations |
| 6 | `wiki/concepts/embeddings-and-vector-search.md` | Vectors, SBERT, FAISS, dense vs sparse retrieval | ① sbert.net *Semantic Search* docs — easiest entry. ② Reimers & Gurevych 2019, *Sentence-BERT* (arXiv:1908.10084, EMNLP). ③ FAISS wiki (github.com/facebookresearch/faiss/wiki) — Flat vs IVF vs HNSW. ④ HuggingFace MTEB leaderboard — how to pick an embedding model |
| 7 | `wiki/concepts/evaluation-metrics.md` | ROUGE-L, BERTScore, RAGAS — what each measures | ① RAGAS docs (docs.ragas.io) — metrics pages. ② Es et al. 2023, *RAGAS* (arXiv:2309.15217, EACL). ③ Zhang et al. 2019, *BERTScore* (arXiv:1904.09675, ICLR). ④ Lin 2004, *ROUGE* (ACL) — skim |
| 8 | `wiki/concepts/production-rag-patterns.md` | Sentence-window, reranking, hybrid retrieval, HyDE | ① LlamaIndex *Sentence Window Retrieval* + *Auto-merging* docs. ② sbert.net *Retrieve & Re-Rank* docs (cross-encoders). ③ Gao et al. 2022, *HyDE* (arXiv:2212.10496, ACL). ④ Qdrant/Weaviate hybrid-search docs — RRF fusion |

## Stage 2 — Agentic RAG (after Stage 1 is solid)

| Step | File | What you will understand after | Primary resources (in reading order) |
|------|------|-------------------------------|--------------------------------------|
| 9  | `wiki/concepts/agentic-rag.md` | ReAct, Self-RAG, CRAG, FLARE — when agents beat standard RAG | ① Yao et al. 2022, *ReAct* (arXiv:2210.03629, ICLR) — read first, everything builds on it. ② Asai et al. 2023, *Self-RAG* (arXiv:2310.11511, ICLR). ③ Yan et al. 2024, *CRAG* (arXiv:2401.15884). ④ Jiang et al. 2023, *FLARE* (arXiv:2305.06983, EMNLP). ⑤ Singh et al. 2025, *Agentic RAG survey* (arXiv:2501.09136) — ties it together |
| 10 | `wiki/concepts/agent-frameworks.md` | LangChain, LangGraph, LlamaIndex — tools, memory, function calling | ① LangGraph docs — *Quickstart* + *Agentic RAG tutorial* (langchain-ai.github.io/langgraph). ② LangChain *Tool calling* concept docs. ③ LlamaIndex *Agents* module docs — compare the two mental models |

## Stage 3 — Production & Advanced (after P8; expert level)

| Step | File | What you will understand after | Primary resources (in reading order) |
|------|------|-------------------------------|--------------------------------------|
| 11 | `wiki/concepts/graph-rag.md` | Knowledge-graph retrieval, when structure beats similarity | ① Edge et al. 2024, *GraphRAG* (arXiv:2404.16130). ② Microsoft GraphRAG docs (microsoft.github.io/graphrag) |
| 12 | `wiki/concepts/multimodal-rag.md` | Retrieving over images/tables/figures, vision-language embeddings | ① LlamaIndex *Multi-modal RAG* docs. ② Faysse et al. 2024, *ColPali* (arXiv:2407.01449, ICLR) |
| 13 | `wiki/concepts/rag-in-production.md` | Observability, eval-in-CI, caching, cost/latency, failure modes | ① LangSmith docs — tracing + evaluation. ② RAGAS docs — CI integration. ③ Revisit Gao et al. survey §production challenges |

---

## Projects — Standard RAG Track (P1–P7)

Every project reuses the previous one's code. Keep one repo; each project is a module/branch that extends the last.

| Project | Key skill | Builds on | Ready after step |
|---------|-----------|-----------|------------------|
| P1 — Chunking Playground | Chunking mechanics from scratch (no frameworks) | — (from scratch) | 2 |
| P2 — My First RAG | End-to-end pipeline: embed → FAISS → retrieve → generate | P1's chunker feeds the index | 6 |
| P3 — PDF RAG | Document parsing (Unstructured/PyMuPDF) | P2's pipeline, swap text loader → PDF parser | 6 (+3) |
| P4 — Strategy Battle | Compare chunking strategies with metrics | P1's strategies + P2's pipeline + RAGAS harness | 7 |
| P5 — Complex Document RAG | Tables, images, OCR | P3's parser, hardened with P4's winning strategy | 7 |
| P6 — Production RAG | Hybrid retrieval, reranking, RAGAS regression suite | P5's ingest + P4's eval harness as CI gate | 8 |
| P7 — University Assistant | Full capstone with Streamlit UI | P6 wrapped in a UI; multi-doc, citations, chat memory | 8 |

## Projects — Agentic RAG Track (P8–P11)
*(start after completing P6 or P7)*

| Project | Key skill | Builds on | Ready after step |
|---------|-----------|-----------|------------------|
| P8 — ReAct RAG Agent | ReAct loop, tools, memory (LangGraph) | P7's retriever exposed as a tool the agent calls | 9–10 |
| P9 — Multi-Hop Agent | Sub-question decomposition, multi-doc routing | P8's agent + a router over multiple P6 indexes | 10 |
| P10 — Self-RAG | Relevance gate, hallucination detection, self-correction | P9's graph + grading/retry nodes (CRAG-style) | 10 |
| P11 — Agentic Capstone | Production agentic RAG: tracing, eval-in-CI, deployment | P10 + LangSmith tracing + RAGAS CI + deployed UI | 13 |

---

## Query Reading Order
*(populated as questions are answered and saved)*
