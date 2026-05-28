# P7 — University Document Assistant (Capstone)

**Tagline:** "A real system that real employees at your university will use."  
**Difficulty:** Advanced  
**Concepts:** All concepts from P1–P6  
**Builds on:** P1–P6 (every component you built)  
**Estimated time:** 2–3 weeks

---

## Project Goal
Build a production-grade document Q&A system for university employees. Employees upload or point the system at university documents — HR policies, academic regulations, fee structures, examination schedules, departmental reports — and ask questions in plain English. The system finds the right answer and cites the exact source document and page.

---

## Real-World Requirements (from your use case)

University documents include:
- **Digital PDFs** — policy documents, examination forms, academic calendars
- **Scanned PDFs** — older circulars, signed letters, printed forms that were scanned
- **Word Documents (.docx)** — HR policies, job descriptions, meeting minutes
- **Spreadsheets (.xlsx)** — fee structures, budget tables, employee records
- **Tables within PDFs** — department-wise data, grade distributions, budget summaries
- **Images in PDFs** — org charts, campus maps, signed scans

---

## System Architecture

```
                    ┌─────────────────────────────┐
                    │     DOCUMENT INGESTION       │
                    │                              │
  PDF/DOCX/XLSX ──► │  Parser (Unstructured/       │
  Scanned PDFs  ──► │  pdfplumber/python-docx/     │
  Images        ──► │  pandas + OCR)               │
                    │         │                    │
                    │         ▼                    │
                    │  Chunker (recursive +         │
                    │  table-aware + caption)       │
                    │         │                    │
                    │         ▼                    │
                    │  Embedder (all-MiniLM-L6-v2) │
                    │         │                    │
                    │         ▼                    │
                    │  Vector Store (FAISS/Chroma)  │
                    │  + BM25 Index                │
                    └─────────────────────────────┘
                                  │
                    ┌─────────────────────────────┐
                    │        QUERY ENGINE          │
                    │                              │
  Employee query ──► │  Hybrid Retrieval            │
                    │  (Dense + BM25 + RRF)        │
                    │         │                    │
                    │         ▼                    │
                    │  Cross-Encoder Reranker       │
                    │         │                    │
                    │         ▼                    │
                    │  Sentence-Window Expansion   │
                    │         │                    │
                    │         ▼                    │
                    │  LLM Answer Generation       │
                    │  + Source Citation           │
                    └─────────────────────────────┘
                                  │
                    ┌─────────────────────────────┐
                    │       USER INTERFACE         │
                    │  (Streamlit web app)         │
                    └─────────────────────────────┘
```

---

## Requirements

### Module 1 — Document Ingestion Pipeline

**M1.1 — Universal Parser**
- [ ] Support file types: `.pdf` (digital), `.pdf` (scanned), `.docx`, `.xlsx`, `.txt`
- [ ] Auto-detect scanned PDFs and apply OCR
- [ ] Extract tables from PDFs and DOCX as structured chunks (headers + rows)
- [ ] Extract tables from XLSX — each sheet becomes a separate document
- [ ] Attach full metadata to every chunk: `source_file`, `page`, `section`, `element_type`, `date_ingested`

**M1.2 — Smart Chunker**
- [ ] Text content → Recursive character splitter (512 tokens, 15% overlap)
- [ ] Table content → Whole table as one chunk if < 300 tokens; else row-per-chunk with repeated header
- [ ] Scanned OCR content → Recursive splitter with larger overlap (20%) to compensate for OCR errors
- [ ] Store each chunk with `prev_chunk` and `next_chunk` for sentence-window expansion

**M1.3 — Incremental Indexing**
- [ ] Track which files have been indexed (store hash of each file)
- [ ] On re-run, only process new or modified files
- [ ] Support deleting a document: remove its chunks from the index

---

### Module 2 — Query Engine

**M2.1 — Hybrid Retrieval**
- [ ] FAISS dense retriever (top-20 candidates)
- [ ] BM25 sparse retriever (top-20 candidates)
- [ ] Merge with Reciprocal Rank Fusion

**M2.2 — Reranker**
- [ ] Cross-encoder reranker on merged top-40 candidates
- [ ] Return top-5 after reranking

**M2.3 — Sentence-Window Expansion**
- [ ] Expand each of the top-5 chunks with ±1 neighbouring chunks
- [ ] Pass expanded context to LLM

**M2.4 — Answer Generation**
- [ ] Prompt template:
  ```
  You are a helpful assistant for university employees.
  Answer the question using ONLY the context provided below.
  If the answer is not in the context, say "I could not find this information in the provided documents."
  Always cite the source document and page number at the end of your answer.

  Context:
  {context}

  Question: {question}

  Answer:
  ```
- [ ] Never hallucinate — if answer is not in context, say so explicitly

**M2.5 — Source Citation**
- [ ] Every answer must end with: `Source: [filename], Page [N]`
- [ ] If multiple sources contributed, list all of them

---

### Module 3 — Web Interface (Streamlit)

**M3.1 — Chat Interface**
- [ ] Text input for employee questions
- [ ] Display answer in a chat bubble
- [ ] Display source citations below each answer
- [ ] Show which chunks were used (expandable section)

**M3.2 — Document Upload**
- [ ] Drag-and-drop file upload
- [ ] Accepted types: PDF, DOCX, XLSX, TXT
- [ ] Show processing progress: `"Parsing... Chunking... Embedding... Done (47 chunks indexed)"`
- [ ] Show current document inventory: list of all indexed files with chunk counts

**M3.3 — Filters**
- [ ] Filter by document type (HR / Academic / Financial / General)
- [ ] Filter by date range (documents ingested after date X)
- [ ] These filters narrow the retrieval to specific document subsets

---

### Module 4 — Evaluation & Quality

**M4.1 — Test Set**
- [ ] Create 20 test questions from your university documents with reference answers
- [ ] At least 5 questions answered by table data
- [ ] At least 3 questions from scanned documents

**M4.2 — RAGAS Evaluation**
- [ ] Run RAGAS on all 20 questions
- [ ] Target scores:
  - Faithfulness ≥ 0.85
  - Answer Relevance ≥ 0.80
  - Context Precision ≥ 0.75
  - Context Recall ≥ 0.70
- [ ] If any score is below target, apply the fix from P6-R5

**M4.3 — Failure Analysis**
- [ ] Identify the 3 worst-performing questions
- [ ] For each: diagnose why it failed (wrong chunk retrieved? chunk fragmented? LLM ignored context?)
- [ ] Apply a fix for at least 2 of them

---

### Module 5 — Documentation

**M5.1 — README**
- [ ] How to install dependencies
- [ ] How to add documents
- [ ] How to run the system
- [ ] Screenshot of the interface

**M5.2 — System Report (for university submission)**
- [ ] Architecture diagram (use the one above as base)
- [ ] Document types supported
- [ ] RAGAS evaluation results table
- [ ] 3 example questions + answers + sources
- [ ] Limitations and future improvements

---

## Technology Stack

| Component | Tool | Why |
|-----------|------|-----|
| PDF parsing | `pdfplumber` + `Unstructured` | Table extraction + mixed content |
| OCR | `pytesseract` + `pdf2image` | Scanned PDFs |
| DOCX parsing | `python-docx` | Word documents |
| XLSX parsing | `pandas` | Spreadsheets |
| Chunking | `LangChain RecursiveCharacterTextSplitter` | Reliable, well-tested |
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` | Fast, free, good quality |
| Vector store | `FAISS` or `Chroma` | Local, no cloud required |
| BM25 | `rank_bm25` | Keyword matching |
| Reranker | `cross-encoder/ms-marco-MiniLM-L-6-v2` | Free, good quality |
| LLM | Groq API (free) or Ollama (local) | Free to run |
| UI | `Streamlit` | Fast to build, looks professional |
| Evaluation | `ragas` | Standard RAG evaluation framework |

---

## Folder Structure
```
university-assistant/
├── app/
│   ├── main.py              ← Streamlit app entry point
│   ├── ingestion/
│   │   ├── parser.py        ← Universal document parser
│   │   ├── chunker.py       ← Smart chunker
│   │   └── indexer.py       ← Embedding + FAISS index builder
│   ├── retrieval/
│   │   ├── hybrid.py        ← Dense + BM25 + RRF
│   │   ├── reranker.py      ← Cross-encoder reranker
│   │   └── expander.py      ← Sentence-window expansion
│   └── generation/
│       └── generator.py     ← LLM answer + citation
├── data/
│   ├── documents/           ← University documents go here
│   └── index/               ← Saved FAISS index + chunk store
├── evaluation/
│   ├── test_questions.json  ← 20 test Q&A pairs
│   └── evaluate.py          ← RAGAS evaluation runner
├── requirements.txt
└── README.md
```

---

## Milestone Checklist

- [ ] **Week 1:** Module 1 complete — all document types parse and index correctly
- [ ] **Week 2:** Modules 2 & 3 complete — working Q&A interface with citations
- [ ] **Week 3:** Module 4 complete — RAGAS scores meet targets; report written

---

## What Makes This Capstone Strong for Submission
1. **Real problem** — not a toy demo; solves a genuine need at your institution
2. **Real documents** — handles messy real-world files, not clean Wikipedia text
3. **Measured quality** — RAGAS scores prove the system works, not just looks like it works
4. **Production architecture** — hybrid retrieval + reranking + sentence-window are industry-standard patterns
5. **Every component explained** — you built each piece yourself across P1–P6, so you can explain every design decision
