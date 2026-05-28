---
title: Self-RAG — Learning to Retrieve, Generate, and Critique through Self-Reflection
type: source
tags: self-rag, agentic-rag, reflection, hallucination, fine-tuning
updated: 2026-05-28
---

**Authors:** Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, Hannaneh Hajishirzi
**Affiliation:** University of Washington + IBM Research
**Venue:** ICLR 2024 (top ML conference)
**arXiv:** 2310.11511
**Citation count:** 1,500+

## What It Introduced
Self-RAG fine-tunes an LLM to generate special **reflection tokens** that control its own retrieval and output quality — eliminating the need for a separate judge model or hardcoded retrieval schedule.

## The Reflection Tokens
| Token | Meaning |
|-------|---------|
| `[Retrieve]` | I need to look something up |
| `[No Retrieve]` | I can answer from memory |
| `[Relevant]` | Retrieved passage is useful |
| `[Irrelevant]` | Retrieved passage is not useful |
| `[Supported]` | My answer is grounded in context |
| `[Partially Supported]` | My answer is mostly grounded |
| `[Contradicts]` | My answer contradicts the context |
| `[Utility: 1–5]` | How useful is my answer overall? |

## Key Findings
| Finding | Detail |
|---------|--------|
| Outperforms standard RAG on factual tasks | Because it skips retrieval when unnecessary and critiques output quality |
| Reduces hallucination | `[Supported]` token forces the model to verify grounding |
| Adaptive retrieval saves cost | Does not retrieve on simple questions where parametric knowledge suffices |
| Better than ChatGPT on several benchmarks | Despite being a smaller fine-tuned model |

## Important Distinction
Self-RAG requires **fine-tuning** the base LLM to generate reflection tokens. It is not a prompting trick — it is a trained behaviour. For practitioners without fine-tuning resources, the reflection gates can be approximated using LLM-as-judge calls (as in Project P10).

## Used In
[[agentic-rag]]
