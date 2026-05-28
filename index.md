# Chunking & RAG Wiki — Index

**Goal:** Learn chunking and RAG deeply from authoritative sources.

---

## Concepts
| Page | Summary |
|------|---------|
| [[wiki/concepts/rag]] | What RAG is, the 3-stage pipeline, why it exists, what it cannot fix — Lewis et al. 2020 |
| [[wiki/concepts/chunking]] | Five levels of chunking, key parameters, chunk size tradeoff, practical decision guide — Wang et al. 2024, Kamradt 2023 |
| [[wiki/concepts/document-parsing]] | PDF/DOCX/HTML/scanned parsing, table extraction, image handling, OCR, metadata — Unstructured, LlamaParse, pdfplumber |
| [[wiki/concepts/chunking-strategies]] | Deep dive into all 5 strategies with code + tables, images, scanned PDFs, mixed-format handling — Kamradt 2023, LangChain |
| [[wiki/concepts/chunk-overlap]] | What overlap is, how computed, storage cost table, Wang et al. findings, when overlap is not enough |
| [[wiki/concepts/embeddings-and-vector-search]] | What embeddings are, SBERT, cosine similarity, FAISS, dense vs sparse retrieval — Reimers 2019, Johnson 2021 |
| [[wiki/concepts/evaluation-metrics]] | ROUGE-L, BERTScore, RAGAS (faithfulness/relevance/precision/recall), retrieval metrics — Lin 2004, Zhang 2020, Es 2024 |
| [[wiki/concepts/production-rag-patterns]] | Sentence-window, hierarchical chunking, reranking, hybrid retrieval, HyDE, multi-query, quality improvement loop |
| [[wiki/concepts/agentic-rag]] | What agentic RAG is, ReAct, Self-RAG, CRAG, FLARE — Yao 2022, Asai 2023, Jiang 2023, Yan 2024 |
| [[wiki/concepts/agent-frameworks]] | LangChain agents, LangGraph, LlamaIndex agents, tool definition, memory, function calling |

## Sources
| Page | Used in |
|------|---------|
| [[wiki/sources/lewis-2020-rag]] | [[rag]] — original RAG paper, NeurIPS 2020, 5000+ citations |
| [[wiki/sources/wang-2024-best-practices-rag]] | [[chunking]] · [[chunking-strategies]] · [[chunk-overlap]] · [[evaluation-metrics]] — EMNLP 2024 |
| [[wiki/sources/kamradt-2023-five-levels]] | [[chunking]] · [[chunking-strategies]] — Five Levels framework, industry standard practitioner guide |
| [[wiki/sources/reimers-2019-sbert]] | [[embeddings-and-vector-search]] — Sentence-BERT paper, EMNLP 2019, 6000+ citations |
| [[wiki/sources/johnson-2021-faiss]] | [[embeddings-and-vector-search]] — FAISS library paper, IEEE 2021 |
| [[wiki/sources/lin-2004-rouge]] | [[evaluation-metrics]] — ROUGE paper, ACL 2004, 15000+ citations |
| [[wiki/sources/zhang-2020-bertscore]] | [[evaluation-metrics]] — BERTScore paper, ICLR 2020, 3000+ citations |
| [[wiki/sources/es-2024-ragas]] | [[evaluation-metrics]] — RAGAS framework paper, EACL 2024 |
| [[wiki/sources/dandekar-2024-production-rag-workshop]] | [[rag]] · [[chunking]] · [[chunking-strategies]] — Dr. Raj Dandekar (MIT), 7.5hr production RAG workshop |
| [[wiki/sources/databricks-2024-production-rag-complex-docs]] | [[rag]] · [[chunking]] · [[chunking-strategies]] — Jerry Liu (LlamaIndex CEO), Databricks Summit 2024 |
| [[wiki/sources/dandekar-2025-production-rag-workshop-part2]] | [[embeddings-and-vector-search]] · [[evaluation-metrics]] — Dr. Raj Dandekar (MIT), Part 2: embeddings, vector DBs, reranking |
| [[wiki/sources/yao-2022-react]] | [[agentic-rag]] · [[agent-frameworks]] — ReAct paper, ICLR 2023, Princeton + Google Brain, 3000+ citations |
| [[wiki/sources/asai-2023-self-rag]] | [[agentic-rag]] — Self-RAG paper, ICLR 2024, UW + IBM, reflection tokens, hallucination reduction |
| [[wiki/sources/jiang-2023-active-rag-flare]] | [[agentic-rag]] — FLARE paper, EMNLP 2023, CMU + Meta, mid-generation retrieval |
| [[wiki/sources/yan-2024-corrective-rag]] | [[agentic-rag]] — CRAG paper, EMNLP Findings 2024, relevance evaluator + web fallback |
| [[wiki/sources/langgraph-docs]] | [[agent-frameworks]] — LangGraph official docs, stateful graph-based agents |
| [[wiki/sources/llamaindex-agents-docs]] | [[agent-frameworks]] — LlamaIndex Agents docs, SubQuestionQueryEngine, document-centric agents |
| [[wiki/sources/langchain-agents-docs]] | [[agent-frameworks]] — LangChain Agents docs, AgentExecutor, ReAct, most widely used framework |

## Queries
| Page | Summary |
|------|---------|
| *(empty)* | |

## Code
| File | Purpose |
|------|---------|
| *(empty)* | |

## Projects
| Project | Skill | Builds on |
|---------|-------|-----------|
| [P1 — Chunking Playground](projects/p1-chunking-playground.md) | Chunking mechanics from scratch | — |
| [P2 — My First RAG](projects/p2-first-rag.md) | End-to-end RAG pipeline | P1 |
| [P3 — PDF RAG](projects/p3-pdf-rag.md) | PDF parsing, noise cleaning, scanned detection | P2 |
| [P4 — Strategy Battle](projects/p4-strategy-battle.md) | Compare strategies with ROUGE-L + Hit@3 | P3 |
| [P5 — Complex Document RAG](projects/p5-complex-documents.md) | Tables, images, OCR, mixed content | P4 |
| [P6 — Production RAG](projects/p6-production-rag.md) | Hybrid retrieval, reranking, sentence-window, RAGAS | P5 |
| [P7 — University Assistant (Capstone)](projects/p7-capstone-university-assistant.md) | Full production system with Streamlit UI | P1–P6 |
| [P8 — ReAct RAG Agent](projects/p8-react-rag-agent.md) | ReAct loop, multi-tool agent, conversation memory | P6 |
| [P9 — Multi-Hop Agent](projects/p9-multi-hop-agent.md) | Sub-question decomposition, multi-document routing | P8 |
| [P10 — Self-RAG](projects/p10-self-rag.md) | Relevance gate, hallucination detection, self-correction loop | P9 |
