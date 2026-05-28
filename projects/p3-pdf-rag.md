# P3 — PDF RAG

**Tagline:** "Your RAG system grows up — now it reads real files."  
**Difficulty:** Intermediate  
**Concepts:** `[[document-parsing]]` `[[chunking]]` `[[rag]]`  
**Builds on:** P2 (full RAG pipeline)  
**Estimated time:** 3–4 hours

---

## What You Will Build
Extend your P2 RAG system to handle PDF files — the most common real-world document format. By the end, you will have a RAG system that can answer questions from any PDF you drop into a folder.

---

## What You Will Learn
- How to extract text from digital PDFs
- How PDF parsing can go wrong (garbled text, missing content, page headers as noise)
- How to detect and skip scanned PDFs gracefully
- How to attach page numbers as metadata

---

## Requirements

### R1 — Digital PDF Parser
- [ ] Use `pdfplumber` to extract text from a digital PDF
- [ ] Each page becomes its own text block before chunking
- [ ] Attach metadata: `{"source": filename, "page": N, "type": "pdf"}`
- [ ] Print a warning if any page returns less than 50 characters (likely scanned)

### R2 — Noise Cleaning
Real PDFs have noise. Clean your extracted text:
- [ ] Remove repeated page numbers (e.g. lines that are just `"4"` or `"Page 4 of 20"`)
- [ ] Remove lines shorter than 20 characters that appear at the top/bottom of every page (headers/footers)
- [ ] Collapse 3+ consecutive blank lines into one blank line

### R3 — Scanned PDF Detection
- [ ] Before parsing, check if the PDF is scanned: extract page 1 text, if < 100 characters → mark as scanned
- [ ] For scanned PDFs, print a clear message: `"[SKIPPED] filename.pdf — scanned document, OCR not yet supported"`
- [ ] Do not crash — continue processing other files

### R4 — Mixed Document Loader
- [ ] Extend your loader to handle both `.txt` and `.pdf` files from the same folder
- [ ] `.txt` files → direct read
- [ ] Digital `.pdf` → pdfplumber extraction + noise cleaning
- [ ] Scanned `.pdf` → skip with warning
- [ ] All chunks carry consistent metadata regardless of source type

### R5 — Rebuild Index
- [ ] Re-run the full pipeline from P2 on your PDF documents
- [ ] Ask 5 questions whose answers are in the PDFs
- [ ] Record: question, retrieved chunks (with page numbers), answer quality (good/partial/wrong)

---

## Experiment to Run
Take any PDF textbook chapter or research paper you have. Index it. Ask:
1. A question whose answer is on a specific page → verify the correct page is retrieved
2. A question that spans two pages → observe what happens (the chunk likely only has half the answer)
3. The same question after increasing overlap from 10% to 25% → does it improve?

---

## Bonus Challenges
- [ ] Add OCR support for scanned PDFs using `pytesseract`
- [ ] Handle multi-column PDFs (hint: `pdfplumber` has `extract_words()` with position data)
- [ ] Display the page number in the final answer: "Source: document.pdf, Page 7"

---

## Deliverable
Updated `p3_indexer.py` and `p3_query.py` that handle both `.txt` and `.pdf` inputs.
