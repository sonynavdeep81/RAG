# P5 — Complex Document RAG

**Tagline:** "Tables, images, scanned pages — your RAG handles it all."  
**Difficulty:** Intermediate–Advanced  
**Concepts:** `[[document-parsing]]` `[[chunking-strategies]]` `[[rag]]`  
**Builds on:** P4 (evaluation harness + multi-strategy RAG)  
**Estimated time:** 5–6 hours

---

## What You Will Build
A document processing pipeline that correctly handles PDFs containing tables, images, mixed layouts, and scanned pages — not just clean text. Each content type is parsed, chunked, and indexed appropriately.

---

## What You Will Learn
- Why standard text chunking fails on real-world PDFs
- How to extract tables as structured data
- How to handle images using captions and descriptions
- How to run OCR on scanned pages
- How to build a unified index from mixed content types

---

## Requirements

### R1 — Table Extractor
- [ ] Use `pdfplumber` to detect and extract tables from a PDF
- [ ] For each table, create a chunk where: first line = column headers, each subsequent line = one row
- [ ] Attach metadata: `{"type": "table", "source": filename, "page": N, "table_id": M}`
- [ ] Test: ask a question whose answer is a specific value in the table → verify it is retrieved

### R2 — Scanned Page Handler (OCR)
- [ ] Use `pdf2image` + `pytesseract` to OCR any page that has < 100 characters of extractable text
- [ ] Clean OCR output: remove lines that are just numbers or single characters
- [ ] Attach metadata: `{"type": "ocr_text", "source": filename, "page": N}`
- [ ] Print a processing log: `"Page 3: digital text (847 chars)"` or `"Page 4: OCR applied (1203 chars)"`

### R3 — Image Caption Extractor
- [ ] Detect image captions: lines that start with "Figure", "Fig.", "Table", "Chart", "Image", or "Diagram"
- [ ] Create a chunk from the caption text
- [ ] Attach metadata: `{"type": "image_caption", "source": filename, "page": N}`

### R4 — Unified Document Processor
- [ ] Build a single function `process_document(filepath)` that:
  - Detects file type (PDF, DOCX, TXT)
  - For PDFs: runs R1 (tables) + R2 (OCR for scanned) + R3 (captions) + text chunking
  - For DOCX: extracts text and tables using `python-docx`
  - For TXT: reads directly
  - Returns a flat list of chunks, each with type + metadata
- [ ] Print a summary: `"Processed: 15 text chunks, 4 table chunks, 2 caption chunks"`

### R5 — Mixed-Content RAG
- [ ] Index all chunk types together in one FAISS index
- [ ] In retrieval output, show the chunk type alongside the text:
  ```
  [TEXT, page 3] The policy states that employees must...
  [TABLE, page 7] Department | Budget | Headcount
  [OCR, page 12]  Scanned content: Annual report figures...
  ```
- [ ] Ask 5 questions that require answers from different chunk types:
  - 2 questions answered by text
  - 2 questions answered by table data
  - 1 question answered by a scanned page

### R6 — Quality Check
- [ ] Compute Hit@3 for each question
- [ ] Note which chunk type was retrieved for the correct answer
- [ ] Write a brief observation: did table chunks retrieve correctly? OCR chunks?

---

## Test Document Ideas
- A university annual report (has tables: enrollment numbers, budget)
- A research paper with figures and tables
- A form or policy document with some scanned pages

---

## Bonus Challenges
- [ ] Add multimodal image description using an LLM API (send image, get text description, embed description)
- [ ] Use `Unstructured` library instead of manual parsing — compare output quality
- [ ] Handle Word documents (.docx) with embedded tables using `python-docx`

---

## Deliverable
`p5_processor.py` (document processing pipeline) + `p5_rag.py` (unified RAG with mixed content).
