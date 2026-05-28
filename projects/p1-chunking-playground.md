# P1 — Chunking Playground

**Tagline:** "Take a document apart and understand every piece."  
**Difficulty:** Beginner  
**Concepts:** `[[chunking]]` `[[chunk-overlap]]` `[[chunking-strategies]]`  
**Builds on:** Nothing — this is the starting point  
**Estimated time:** 2–3 hours

---

## What You Will Build
A command-line tool that takes any text file, applies different chunking strategies, and shows you the chunks side by side — so you can *see* exactly what each strategy does to real content.

---

## What You Will Learn
- How fixed-size, recursive, and sentence-based chunking differ in practice
- How overlap changes chunk content
- How chunk size affects the number of chunks and the amount of context per chunk
- Why some chunks make sense and others cut ideas in half

---

## Requirements

### R1 — Fixed-Size Chunker
- [ ] Write a function `fixed_chunk(text, chunk_size, overlap)` from scratch (no libraries)
- [ ] It must accept chunk size in tokens (split on whitespace for simplicity)
- [ ] It must apply overlap correctly — the last `overlap` tokens of chunk N appear at the start of chunk N+1
- [ ] Print: total chunks, average chunk length, first 3 chunks

### R2 — Recursive Character Chunker
- [ ] Use `LangChain RecursiveCharacterTextSplitter`
- [ ] Chunk the same text with chunk_size=200, overlap=20
- [ ] Print: total chunks, average chunk length, first 3 chunks

### R3 — Side-by-Side Comparison
- [ ] Run both chunkers on the same input text
- [ ] Print a table showing: chunk number, first 50 chars, last 50 chars, length
- [ ] Highlight any chunk that ends mid-sentence (ends without `.`, `?`, `!`)

### R4 — Overlap Visualiser
- [ ] For a given text, show the overlapping tokens between chunk N and chunk N+1
- [ ] Highlight the shared text in a visible way (e.g. `>>> shared text <<<`)

### R5 — Statistics Report
- [ ] For each chunking configuration, print:
  - Total chunks
  - Min / max / average chunk length
  - Number of chunks ending mid-sentence
  - Estimated storage overhead vs 0% overlap

---

## Sample Input to Use
Use this paragraph as your test text (or any Wikipedia article):
```
The Retrieval-Augmented Generation model was introduced by Lewis et al. in 2020.
It combines a non-parametric retriever over a document index with a parametric
sequence-to-sequence generator. The retriever finds the most relevant passages
from a large corpus, and the generator conditions on these passages to produce
an answer. This approach significantly reduces hallucination compared to
generation-only models, because the answer is grounded in retrieved text.
Subsequent work has extended RAG to handle multi-hop reasoning, long documents,
and domain-specific corpora including medical, legal, and technical text.
```

---

## Bonus Challenges
- [ ] Add a sentence-based chunker that splits only at `.`, `?`, `!`
- [ ] Try chunk sizes of 50, 100, 200, 500 tokens and plot how many chunks ending mid-sentence changes
- [ ] Load a real Wikipedia article (use `wikipedia` Python library) and chunk it

---

## Deliverable
A single Python script `p1_chunking_playground.py` that runs all 5 requirements when executed.

```
python p1_chunking_playground.py --input sample.txt --chunk-size 200 --overlap 20
```
