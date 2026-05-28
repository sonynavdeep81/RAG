---
title: ROUGE: A Package for Automatic Evaluation of Summaries
type: source
tags: evaluation, rouge, metrics, summarisation
updated: 2026-05-28
---

**Author:** Chin-Yew Lin  
**Affiliation:** Information Sciences Institute, USC  
**Venue:** ACL Workshop on Text Summarisation, 2004  
**Citation count:** 15,000+ (one of the most cited NLP papers ever)

## What It Introduced
ROUGE (Recall-Oriented Understudy for Gisting Evaluation) — a set of metrics for comparing automatically generated text against human reference text.

## Variants
| Variant | Measures |
|---------|----------|
| ROUGE-1 | Unigram (single word) overlap |
| ROUGE-2 | Bigram (two-word sequence) overlap |
| **ROUGE-L** | Longest Common Subsequence — most used in RAG |
| ROUGE-S | Skip-bigram overlap |

## Why ROUGE-L Is Used in RAG
ROUGE-L does not require exact word-order matching — it finds the longest sequence of words that appear in both texts in the same order, with other words allowed in between. This makes it more lenient than ROUGE-1/2 for paraphrase.

## Key Limitation
Purely lexical — "car" and "automobile" score 0 overlap. Cannot detect meaning, only surface word patterns.

## Used In
[[evaluation-metrics]]
