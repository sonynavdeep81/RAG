# Chunking & RAG Wiki — Rules

## Session start
Read `STATE.md` first. Keep it current (~5 lines).

## Directory structure
- `raw/` → immutable sources. Never modify.
- `wiki/concepts/` → one file per concept, dense notes with source citations
- `wiki/sources/` → one file per ingested source
- `wiki/queries/` → full readable answers the user reads directly
- `wiki/code/` → runnable implementations
- `wiki/lint/` → contradiction logs only
- `index.md` → master catalog (SSOT)
- `log.md` → append-only: `## [YYYY-MM-DD] action | title`
- `learning-path.md` → reading and execution order

## On Ingest ("ingest <source>")
1. Read source. 2. Discuss takeaways. 3. Write `wiki/sources/[slug].md`. 4. Create/update `wiki/concepts/`. 5. Check contradictions — report before writing. 6. Update `index.md` + append `log.md`.

## On Query (a question)
1. Read `index.md` → relevant pages → synthesize. 2. Ask: "Save as wiki/queries/[slug].md?" 3. If yes → write full readable article. 4. Add to `learning-path.md`.

## On Contradictions
Flag immediately. Preserve old claim in `wiki/lint/contradictions.md`. Update wiki to better claim.

## Source quality rule
Only use sources that are: peer-reviewed papers from top venues (NeurIPS, EMNLP, ACL, ICLR), widely-cited surveys, or official documentation from major frameworks (LangChain, LlamaIndex, HuggingFace).

## Formatting
- Dense tables/key-value for concepts/sources; readable prose for queries
- Math: `$...$` inline LaTeX; backticks for code only
- `[[wikilinks]]` for cross-references between concept pages
