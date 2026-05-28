---
title: Document Parsing — Preprocessing Before Chunking
type: concept
tags: parsing, pdf, ocr, tables, images, preprocessing, unstructured
sources:
  - Unstructured.io documentation
  - LlamaParse documentation (LlamaIndex)
  - Jerry Liu 2024 — "Building Production RAG Over Complex Documents" (Databricks Summit)
updated: 2026-05-28
---

## Summary
Parsing happens before chunking. It extracts clean, structured text from raw files. Poor parsing means dirty input to your chunker — no chunking strategy can fix garbled or incomplete text. This step is where most production RAG systems fail on real-world documents.

---

## Why Parsing Is a Separate Step

A RAG pipeline has two distinct preprocessing stages that are often confused:

```
Raw file (PDF/DOCX/HTML/image)
    │
    ▼
PARSING  ← "What is in this file? What type is each element?"
    │       Extracts: text blocks, tables, images, headers, captions
    ▼
CHUNKING ← "How do I split this content into retrievable pieces?"
    │       Splits: by size, structure, or semantics
    ▼
Embedding + Indexing
```

Skipping parsing and going straight to chunking works only for simple plain-text files. For everything else — PDFs, Word documents, HTML pages, scanned documents — you need proper parsing first.

---

## File Format Decision Guide

### Plain Text / Markdown (.txt, .md)
**Complexity:** Low  
**Parser needed:** None — read directly  
**Chunking:** Recursive character splitter or markdown header splitter

```python
with open("document.txt", "r") as f:
    text = f.read()
# Chunk directly
```

---

### Word Documents (.docx)
**Complexity:** Low-Medium  
**Parser:** `python-docx`

```python
from docx import Document

doc = Document("document.docx")
text = "\n".join([para.text for para in doc.paragraphs])
# For tables:
for table in doc.tables:
    for row in table.rows:
        print(" | ".join(cell.text for cell in row.cells))
```

**Watch out for:** Embedded images (ignored by python-docx), footnotes, headers/footers.

---

### Digital PDFs (.pdf — text-based, not scanned)
**Complexity:** Medium  
**Three parser options:**

| Parser | Speed | Table support | Image support | Best for |
|--------|-------|--------------|---------------|---------|
| `pypdf` | Fast | Poor | None | Simple text-only PDFs |
| `pdfplumber` | Medium | Good | None | PDFs with tables |
| `PyMuPDF (fitz)` | Fast | Medium | Image extraction | General purpose |
| `pdfminer.six` | Slow | Medium | None | Layout-accurate extraction |
| `LlamaParse` (paid) | Slow | Excellent | Excellent | Complex, mixed-content PDFs |
| `Unstructured` | Medium | Good | Good | Production, open source |

**Recommended default for PDFs with tables:**
```python
import pdfplumber

full_content = []
with pdfplumber.open("document.pdf") as pdf:
    for page in pdf.pages:
        # Extract text
        text = page.extract_text()
        if text:
            full_content.append({"type": "text", "content": text})
        # Extract tables
        for table in page.extract_tables():
            rows = [" | ".join(str(c) for c in row) for row in table]
            full_content.append({"type": "table", "content": "\n".join(rows)})
```

---

### Scanned PDFs (.pdf — images of pages)
**Complexity:** High  
**Requires OCR (Optical Character Recognition)**

**Detection first:**
```python
import pypdf

reader = pypdf.PdfReader("document.pdf")
text = "".join(page.extract_text() or "" for page in reader.pages)
is_scanned = len(text.strip()) < 100
```

**OCR pipeline:**
```python
from pdf2image import convert_from_path
import pytesseract

# Step 1: Convert PDF pages to images
pages = convert_from_path("scanned.pdf", dpi=300)

# Step 2: OCR each page
full_text = ""
for i, page_img in enumerate(pages):
    page_text = pytesseract.image_to_string(page_img, lang="eng")
    full_text += f"\n--- Page {i+1} ---\n{page_text}"

# Step 3: Chunk the extracted text normally
```

**When Tesseract is not enough:**
- Tables in scanned PDFs → AWS Textract (preserves table structure)
- Handwritten text → Google Document AI or Azure Form Recognizer
- Multi-language documents → Tesseract with language packs

---

### HTML / Web Pages (.html)
**Complexity:** Low-Medium  
**Parser:** `BeautifulSoup` + `LangChain HTMLHeaderTextSplitter`

```python
from bs4 import BeautifulSoup
import requests

html = requests.get(url).text
soup = BeautifulSoup(html, "html.parser")

# Remove navigation, footers, ads
for tag in soup(["nav", "footer", "script", "style", "header"]):
    tag.decompose()

clean_text = soup.get_text(separator="\n", strip=True)
```

---

### Spreadsheets (.xlsx, .csv)
**Complexity:** Low  
**Parser:** `pandas`

```python
import pandas as pd

df = pd.read_excel("data.xlsx", sheet_name="Sheet1")

# Option A: Each row becomes a chunk (for row-level queries)
chunks = []
for _, row in df.iterrows():
    chunk_text = " | ".join(f"{col}: {val}" for col, val in row.items())
    chunks.append(chunk_text)

# Option B: Whole table as one chunk (for overview queries)
table_text = df.to_markdown(index=False)
```

**Decision:** Use row-per-chunk when users ask about specific records. Use whole-table when users ask about patterns or aggregates.

---

## The Metadata Rule (Critical for Production)

Every chunk must carry metadata about where it came from. Without metadata:
- You cannot cite sources in answers
- You cannot filter retrieval by document type, date, or section
- You cannot debug why a bad chunk was retrieved

```python
chunk = {
    "text": "The patient must take 500mg every 8 hours.",
    "metadata": {
        "source_file": "clinical_protocol_v3.pdf",
        "page": 4,
        "section": "Dosage Instructions",
        "element_type": "text",   # text | table | image_description
        "doc_type": "medical",
        "date_ingested": "2026-05-28"
    }
}
```

LangChain preserves metadata automatically when using document loaders:
```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("document.pdf")
pages = loader.load()  # Each page is a Document with metadata {"source": ..., "page": N}
```

---

## Production Parser Recommendation (Decision Tree)

```
What type of file?
│
├── Plain text / Markdown
│   └── Read directly → chunk
│
├── Word document (.docx)
│   └── python-docx → chunk text + tables separately
│
├── Simple PDF (text only, no tables/images)
│   └── pypdf or pdfplumber → recursive chunk
│
├── PDF with tables
│   └── pdfplumber → text chunks + table chunks (keep tables whole)
│
├── PDF with images and tables (complex)
│   └── Unstructured (free) or LlamaParse (paid) → categorise elements → chunk each type
│
├── Scanned PDF
│   └── pdf2image + Tesseract (free) or AWS Textract (tables) → OCR text → chunk
│
├── HTML / web page
│   └── BeautifulSoup → clean HTML → HTMLHeaderTextSplitter
│
└── Spreadsheet
    └── pandas → row-per-chunk or whole-table chunk depending on query type
```

---

## Common Mistakes in Parsing

| Mistake | Consequence | Fix |
|---------|------------|-----|
| Using `pypdf` on scanned PDFs | Empty chunks | Detect + use OCR |
| Ignoring tables | Table data not retrievable | Extract tables separately |
| No metadata attached | Cannot cite sources, no filtering | Always attach source + page |
| Stripping all whitespace | Destroys structure (numbered lists, tables) | Preserve meaningful whitespace |
| Parsing headers/footers as content | Noise in every chunk | Filter page numbers, headers, footers |
| Treating every newline as a paragraph | Over-fragmented chunks | Collapse single newlines, keep double newlines |

## Related
[[chunking]] · [[chunking-strategies]] · [[rag]]
