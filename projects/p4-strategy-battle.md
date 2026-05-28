# P4 — Strategy Battle

**Tagline:** "Which chunking strategy wins? Let the numbers decide."  
**Difficulty:** Intermediate  
**Concepts:** `[[chunking-strategies]]` `[[evaluation-metrics]]` `[[chunk-overlap]]`  
**Builds on:** P3 (PDF parsing + RAG pipeline)  
**Estimated time:** 4–5 hours

---

## What You Will Build
An evaluation harness that runs multiple chunking strategies on the same document, asks the same set of questions to each, and scores the results — so you can empirically decide which strategy works best for your content type.

---

## What You Will Learn
- How to measure chunking quality objectively
- That no single strategy wins on all document types
- How ROUGE-L and Hit@k work in practice
- How to build an evaluation-driven development workflow

---

## Requirements

### R1 — Strategy Runner
Implement all 4 practical strategies (skip Level 1 fixed-character — it's never useful):
- [ ] **Strategy A:** Fixed-size token chunker (from P1) — 512 tokens, 10% overlap
- [ ] **Strategy B:** `RecursiveCharacterTextSplitter` — 512 tokens, 10% overlap
- [ ] **Strategy C:** `MarkdownHeaderTextSplitter` (if document has headers) or structure-based
- [ ] **Strategy D:** `SemanticChunker` (LangChain experimental)

For each strategy:
- [ ] Chunk the same input document
- [ ] Build a separate FAISS index
- [ ] Record: total chunks, min/max/avg chunk size

### R2 — Question Set
- [ ] Write 10 questions about your test document
- [ ] For each question, write the reference answer (what the correct answer should be)
- [ ] Store as a JSON file: `[{"question": "...", "reference_answer": "..."}]`

### R3 — Retrieval Evaluation
For each strategy × each question:
- [ ] Retrieve top-3 chunks
- [ ] Check Hit@3: is the answer present in any of the 3 chunks? (1/0)
- [ ] Compute Extractive ROUGE-L: ROUGE-L between concatenated chunks and reference answer

### R4 — Generation Evaluation
For each strategy × each question:
- [ ] Generate an answer using the retrieved chunks
- [ ] Compute ROUGE-L between generated answer and reference answer
- [ ] Compute BERTScore F1

### R5 — Results Table
Print a final comparison table:
```
Strategy          | Chunks | Hit@3 | Ext-ROUGE-L | Gen-ROUGE-L | BERTScore
──────────────────|────────|───────|─────────────|─────────────|──────────
Fixed-size        |   47   |  0.70 |    0.31     |    0.28     |  0.84
Recursive         |   39   |  0.80 |    0.38     |    0.34     |  0.87
Structure-based   |   21   |  0.75 |    0.35     |    0.31     |  0.85
Semantic          |   33   |  0.85 |    0.42     |    0.38     |  0.89
```
- [ ] Highlight the winner in each column
- [ ] Write a 3-sentence conclusion: which strategy won and why

---

## Suggested Test Documents
Use two different document types to see how strategies behave differently:
1. A Wikipedia article (narrative, no headers)
2. A technical tutorial or README (structured, with headers and code)

---

## Experiment to Run
Run the battle on both document types. Notice:
- On the narrative document → semantic chunking likely wins
- On the structured document → structure-based chunking likely wins
This is the empirical proof that "no strategy wins everywhere."

---

## Bonus Challenges
- [ ] Add chunk size as a variable: run each strategy at 256, 512, and 1024 tokens
- [ ] Plot Hit@3 vs chunk size as a line chart
- [ ] Add overlap as a variable: 0%, 10%, 20%, 30% — find the optimal for your document

---

## Deliverable
`p4_strategy_battle.py` — takes a document file and a question set JSON, outputs the comparison table.
```
python p4_strategy_battle.py --doc document.pdf --questions questions.json
```
