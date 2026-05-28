---
title: Chunking Strategies — Deep Dive
type: concept
tags: chunking, text-splitting, langchain, strategies
sources:
  - Greg Kamradt — "Five Levels of Text Splitting" GitHub notebook (2023)
  - LangChain Text Splitters documentation
  - Pinecone — "Chunking Strategies for LLM Applications" (2023)
updated: 2026-05-28
---

## Summary
A practical deep dive into each chunking strategy — how it works mechanically, what its failure modes are, and runnable code for each. Strategy choice matters more than chunk size tuning.

---

## Strategy 1 — Fixed Character Splitting

### How it works
Split every N characters. No awareness of words, sentences, or meaning.

```python
text = "The patient must take 500mg every 8 hours. If symptoms worsen, call a doctor."

chunk_size = 40
chunks = [text[i:i+chunk_size] for i in range(0, len(text), chunk_size)]

# Result:
# "The patient must take 500mg every 8 hou"   ← cuts mid-word
# "rs. If symptoms worsen, call a doctor."
```

### Failure mode
Cuts mid-word and mid-sentence consistently. Produces incoherent chunks.

### When to use
Never in production. Useful only for understanding why smarter methods exist.

---

## Strategy 2 — Recursive Character Splitting

### How it works
Tries a hierarchy of separators in order. If a chunk is still too large after splitting on `\n\n`, it tries `\n`, then `.`, then ` `.

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=200,        # max characters per chunk
    chunk_overlap=20,      # characters shared between adjacent chunks
    separators=["\n\n", "\n", ". ", " ", ""]  # tried in this order
)

chunks = splitter.split_text(your_text)
```

### What it produces
```
Input:
  "Section 1\n\nThis is the first paragraph about setup.\n\nSection 2\n\nThis is about configuration."

Output (chunk_size=50):
  Chunk 1: "Section 1"
  Chunk 2: "This is the first paragraph about setup."
  Chunk 3: "Section 2"
  Chunk 4: "This is about configuration."
```
Paragraph breaks are respected first. Only falls back to sentence/word splits if needed.

### Why this is the industry default
- Fast (no embeddings needed)
- Respects natural language boundaries
- Works on any text format
- Consistent and reproducible

### Failure mode
Cannot understand *meaning* — splits at `\n\n` even if two paragraphs are deeply connected ideas. A step that spans two paragraphs will be split.

---

## Strategy 3 — Document Structure Splitting

### How it works
Use the document's own markup as split signals. Different parsers for different formats.

```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

headers_to_split_on = [
    ("#",  "heading_1"),
    ("##", "heading_2"),
    ("###","heading_3"),
]
splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers_to_split_on)
chunks = splitter.split_text(markdown_text)
# Each chunk carries metadata: {"heading_1": "Installation", "heading_2": "Prerequisites"}
```

For HTML:
```python
from langchain.text_splitter import HTMLHeaderTextSplitter
# Splits on <h1>, <h2>, <h3> etc.
```

### What it produces
Each chunk represents one complete section. Metadata tells you exactly where in the document the chunk came from — useful for citations.

### When to use
- Markdown documentation
- HTML web pages
- PDFs with clear heading structure
- Legal documents with numbered sections
- Technical manuals (Chapter → Section → Subsection)

### Failure mode
Sections vary wildly in length. A 5-word section header becomes a chunk; a 3000-word section also becomes a chunk. Usually combined with a secondary size-based splitter to cap maximum chunk length.

---

## Strategy 4 — Semantic Splitting

### How it works
1. Split text into sentences
2. Embed each sentence using a sentence transformer model
3. Compute cosine similarity between adjacent sentence embeddings
4. Insert a chunk boundary wherever similarity drops below a threshold

```python
from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai.embeddings import OpenAIEmbeddings

splitter = SemanticChunker(
    embeddings=OpenAIEmbeddings(),
    breakpoint_threshold_type="percentile",  # or "standard_deviation"
    breakpoint_threshold_amount=95           # split at bottom 5% similarity drops
)
chunks = splitter.split_text(your_text)
```

### What it produces
```
Paragraph about diet     ─► high similarity ─► same chunk
Paragraph about exercise ─► high similarity ─► same chunk
Paragraph about surgery  ─► LOW similarity  ─► NEW CHUNK starts here
Paragraph about recovery ─► high similarity ─► same chunk
```

Chunks align with topic shifts rather than arbitrary boundaries.

### When to use
- Long research papers
- News articles
- Books / long narrative documents
- Any document where topic changes matter more than length

### Failure mode
- **Expensive** — must embed every sentence at index time
- **Threshold is fragile** — too sensitive: too many tiny chunks; too lenient: chunks too large
- Slow for large corpora
- Can produce very uneven chunk sizes

### Threshold types (LangChain)
| Type | How it decides to split |
|------|------------------------|
| `percentile` | Split at the bottom X% of all similarity scores |
| `standard_deviation` | Split when similarity drops more than X std devs below mean |
| `interquartile` | Split when similarity is below Q1 - 1.5×IQR |

---

## Strategy 5 — Agentic Splitting

### How it works
Send the document (or sections of it) to an LLM with a prompt asking it to identify logical chunk boundaries and return them.

```python
# Conceptual — not a standard library call
prompt = """
Read this document and identify where each self-contained unit of information ends.
Return a list of split positions.

Document: {text}
"""
# LLM returns: [position_1, position_2, ...]
# You then split at those positions
```

### When to use
- Complex, irregular documents that defeat all other strategies
- Documents mixing tables, prose, code, and figures
- When the cost of a wrong chunk is very high (legal, medical)

### Failure mode
- Very expensive (LLM call per document)
- Slow
- Non-deterministic (same document may produce different splits on different runs)
- Overkill for most use cases

---

---

## Handling Special Content Types

Regular text chunking breaks down completely for tables, images, and mixed-format documents. Each content type needs its own strategy.

### Tables

**The problem:** A recursive splitter sees a table as plain text and cuts it mid-row:
```
| Drug | Dose  | Frequency |
|------|-------|-----------|
| A    | 500mg | Every 8h  |    ← chunk boundary here
| B    | 200mg | Once      |
```
The second chunk has no header row — completely meaningless in isolation.

**Solutions by use case:**

| Situation | Best approach |
|-----------|--------------|
| Table fits in one chunk | Keep it whole — increase chunk size temporarily |
| Table is too large | Convert to row-per-chunk with header repeated |
| Need to query table values | Extract to structured format (CSV/JSON), store separately |
| Table has rich context | Summarise the table with an LLM; store summary as the chunk |

**Practical tool:** `pdfplumber` extracts tables from PDFs as Python lists. `pandas` converts them to markdown strings that embed well.

```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    for page in pdf.pages:
        tables = page.extract_tables()
        for table in tables:
            # Convert to markdown string for embedding
            header = " | ".join(table[0])
            rows = [" | ".join(row) for row in table[1:]]
            table_text = header + "\n" + "\n".join(rows)
            # Treat each table as its own chunk
```

---

### Images, Charts, and Diagrams

**The problem:** Standard text chunkers completely ignore images — they either skip them or extract garbled text from around them.

**Three strategies depending on your needs:**

**Strategy A — Skip images (simplest)**
Ignore images entirely. Works when images are decorative (logos, borders) and not information-bearing.

**Strategy B — Use surrounding captions and alt-text**
Extract the text immediately before/after the image (usually the caption or figure description). This text describes what the image contains.
```
"Figure 3 shows the relationship between chunk size and retrieval precision..."
```
Embed the caption text as the chunk representing that image.

**Strategy C — Multimodal embeddings (best for information-dense images)**
Use a vision-language model to either:
- Generate a text description of the image → embed the description
- Embed the image directly using a multimodal model (CLIP, GPT-4V)

```python
# Option: Generate description using a vision LLM, then embed as text
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Describe this chart for a search index. Be specific about values."},
            {"type": "image_url", "image_url": {"url": f"data:image/png;base64,{image_b64}"}}
        ]
    }]
)
image_description = response.choices[0].message.content
# Now embed image_description as a regular text chunk
```

**When to use each:**

| Image type | Strategy |
|-----------|---------|
| Decorative (logos, borders) | A — Skip |
| Charts with caption | B — Caption text |
| Diagrams with labels | B — Caption + label extraction |
| Information-dense charts (clinical, technical) | C — Multimodal |
| Scanned tables in image format | C — OCR first, then treat as table |

---

### Scanned PDFs (Images of Text)

**The problem:** Many real-world documents are scanned — they're images of pages, not text. `pypdf` and `pdfplumber` return empty strings.

**Detection:**
```python
import pypdf

reader = pypdf.PdfReader("document.pdf")
text = reader.pages[0].extract_text()
if not text or len(text.strip()) < 50:
    print("Likely scanned — needs OCR")
```

**OCR solutions:**

| Tool | Quality | Cost | Best for |
|------|---------|------|---------|
| **Tesseract** (open source) | Good | Free | English text, clean scans |
| **AWS Textract** | Excellent | Paid | Tables, forms, mixed content |
| **Google Document AI** | Excellent | Paid | High-volume production |
| **LlamaParse** | Excellent | Paid | Complex PDFs with mixed content |

```python
# Tesseract example
from pdf2image import convert_from_path
import pytesseract

pages = convert_from_path("scanned.pdf")
full_text = ""
for page in pages:
    full_text += pytesseract.image_to_string(page)
# Now chunk full_text normally
```

---

### Mixed-Format Documents (PDF with text + tables + images)

**The recommended pipeline for complex PDFs:**

```
Complex PDF
    │
    ▼
Document Parser (LlamaParse / Unstructured / PDFMiner)
    │
    ├──► Text sections  ──► Recursive character splitter
    ├──► Tables         ──► Table-aware chunker (whole table or row-per-chunk)
    └──► Images         ──► Caption extraction or multimodal description
    │
    ▼
All chunks pooled into one vector index
(each chunk carries metadata: type=text|table|image, page=N, section="...")
    │
    ▼
Retrieval returns mixed chunk types
    │
    ▼
LLM synthesises answer from text + table + image description chunks
```

**Best tool for complex PDFs:** `Unstructured` (open source) or `LlamaParse` (paid, higher quality). Both categorise document elements — titles, narratives, tables, images — before any chunking happens.

```python
from unstructured.partition.pdf import partition_pdf

elements = partition_pdf("complex_document.pdf")
for element in elements:
    print(type(element).__name__, ":", str(element)[:100])
# NarrativeText : The patient must take 500mg every 8 hours...
# Table         : Drug | Dose | Frequency...
# Image         : [Figure 3]
# Title         : Section 2: Configuration
```

---

## Side-by-Side Comparison

| Strategy | Speed | Cost | Respects meaning | Handles structure | Recommended for |
|----------|-------|------|-----------------|------------------|----------------|
| Fixed character | ⚡ Fastest | Free | ❌ No | ❌ No | Nothing |
| Recursive char | ⚡ Fast | Free | Partial | Partial | **General default** |
| Structure-based | ⚡ Fast | Free | Partial | ✅ Yes | Markdown/HTML/PDF |
| Semantic | 🐢 Slow | Embedding cost | ✅ Yes | ❌ No | Long narrative |
| Agentic | 🐢🐢 Slowest | LLM cost | ✅ Best | ✅ Yes | Complex/critical docs |

---

## The Practical Recommendation (Kamradt + Wang et al.)

Start with **Recursive Character Splitting at 512 tokens with 10% overlap**. This is the best general-purpose starting point backed by Wang et al. 2024's experiments. Switch to structure-based if your documents have clear headers. Move to semantic only if retrieval quality is still poor after tuning chunk size and overlap.

## Runnable Code
See `wiki/code/chunking_strategies_demo.py` for working examples of all five strategies.

## Related
[[rag]] · [[chunking]] · [[chunk-overlap]] · [[embeddings]]
