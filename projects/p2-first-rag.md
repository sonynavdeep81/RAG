# P2 — My First RAG

**Tagline:** "Ask your documents a question and get a real answer."  
**Difficulty:** Beginner–Intermediate  
**Concepts:** `[[rag]]` `[[embeddings-and-vector-search]]` `[[chunking]]`  
**Builds on:** P1 (chunking logic)  
**Estimated time:** 3–4 hours

---

## What You Will Build
A simple question-answering system over a small collection of plain text files.
You type a question, the system finds the most relevant chunks, and an LLM answers using only those chunks.

---

## What You Will Learn
- How the full RAG pipeline connects chunking → embedding → retrieval → generation
- How FAISS stores and searches vectors
- How to use sentence-transformers to create embeddings
- How the quality of the retrieved chunks directly affects the answer

---

## Requirements

### R1 — Document Loader
- [ ] Write a function that reads all `.txt` files from a folder
- [ ] Each file becomes a list of chunks (reuse your P1 chunker)
- [ ] Each chunk must carry metadata: `{"source": filename, "chunk_id": N}`

### R2 — Embedding Pipeline
- [ ] Use `sentence-transformers/all-MiniLM-L6-v2` to embed all chunks
- [ ] Store embeddings as a numpy array
- [ ] Print: number of chunks embedded, embedding dimensions, time taken

### R3 — FAISS Index
- [ ] Build a `faiss.IndexFlatL2` index from the embeddings
- [ ] Save the index to disk (`faiss.write_index`)
- [ ] Save the chunk texts and metadata to disk (use `json` or `pickle`)
- [ ] Write a load function that restores the index without re-embedding

### R4 — Retriever
- [ ] Write a function `retrieve(query, index, chunks, k=3)` that:
  - Embeds the query
  - Searches FAISS for top-k nearest chunks
  - Returns the chunk texts + their metadata

### R5 — Generator
- [ ] Use any free LLM API (Groq API with LLaMA-3 is free) or a local model
- [ ] Build a prompt: `"Answer the question using only the context below.\nContext: {chunks}\nQuestion: {query}"`
- [ ] Print the answer and the source filenames it came from

### R6 — Interactive Loop
- [ ] Build a simple loop: user types a question → system answers → asks for next question
- [ ] Type `quit` to exit
- [ ] Show which chunks were used for each answer

---

## Folder Structure
```
p2_first_rag/
├── documents/          ← put your .txt files here
│   ├── doc1.txt
│   └── doc2.txt
├── index/              ← saved FAISS index + chunks
├── p2_indexer.py       ← build the index (run once)
└── p2_query.py         ← interactive Q&A loop
```

---

## Suggested Test Documents
Create 3 simple text files about topics you know well (your university courses, a hobby, a book summary). Ask questions whose answers are only in those files — verify the system finds the right one.

---

## Experiment to Run
1. Ask a question that has a clear answer in one document → observe which chunk was retrieved
2. Ask the same question but deliberately word it differently → does retrieval still work?
3. Remove the relevant document → observe what the LLM does with irrelevant chunks (hallucination)

---

## Bonus Challenges
- [ ] Add cosine similarity score to each retrieved chunk display
- [ ] Try `k=1`, `k=3`, `k=5` — how does answer quality change?
- [ ] Try `all-mpnet-base-v2` (higher quality embedding) vs `all-MiniLM-L6-v2` — which answers better?

---

## Deliverable
Two scripts: `p2_indexer.py` (index documents) and `p2_query.py` (interactive Q&A).
