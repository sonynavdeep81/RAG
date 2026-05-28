---
title: Active Retrieval Augmented Generation (FLARE)
type: source
tags: active-rag, flare, agentic-rag, mid-generation-retrieval
updated: 2026-05-28
---

**Authors:** Zhengbao Jiang, Frank F. Xu, Luyu Gao, Zhiqing Sun, Qian Liu, Jane Dwivedi-Yu, Pengfei Liu, Graham Neubig, Chunting Zhou
**Affiliation:** Carnegie Mellon University + Meta AI
**Venue:** EMNLP 2023 (top NLP conference)
**arXiv:** 2305.06983
**Citation count:** 800+

## What It Introduced
**FLARE (Forward-Looking Active REtrieval)** — triggers retrieval *during* generation, not before it. The LLM generates a tentative next sentence, evaluates its own token-level confidence, and retrieves when about to generate a low-confidence (uncertain) token.

## How It Works
```
Generate tentatively: "The maximum operating temperature is [LOW CONFIDENCE]"
                                                                    │
                                                                    ▼
                                                        Trigger retrieval
                                                        Query: "maximum operating temperature"
                                                                    │
                                                                    ▼
                                                        Retrieved: "Operating temp: 350°C max"
                                                                    │
                                                                    ▼
                                        Regenerate with context: "...is 350°C"
```

## Key Findings
| Finding | Detail |
|---------|--------|
| Outperforms standard RAG on long-form generation | Because retrieval happens where it's needed, not uniformly |
| Especially strong on multi-sentence answers | Mid-generation retrieval catches facts that span multiple sentences |
| Confidence threshold matters | Too sensitive = too many retrievals; too lenient = misses needed facts |

## Key Difference from Standard RAG
Standard RAG: retrieve once before generation (may retrieve irrelevant content).
FLARE: retrieve exactly when the LLM is uncertain mid-generation (targeted retrieval).

## Practical Limitation
Requires access to token-level probability scores — available with open models (HuggingFace) but not always with API-only models (OpenAI API does not expose token probabilities by default).

## Used In
[[agentic-rag]]
