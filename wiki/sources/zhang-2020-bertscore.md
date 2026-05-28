---
title: BERTScore: Evaluating Text Generation with BERT
type: source
tags: evaluation, bertscore, metrics, bert
updated: 2026-05-28
---

**Authors:** Tianyi Zhang, Varsha Kishore, Felix Wu, Kilian Q. Weinberger, Yoav Artzi  
**Venue:** ICLR 2020 (top ML conference)  
**Citation count:** 3,000+

## What It Introduced
BERTScore — an evaluation metric that uses contextual embeddings from BERT to measure semantic similarity between generated and reference text, rather than surface word overlap.

## How It Works
1. Embed every token in both generated and reference text using a pretrained model (RoBERTa-large by default)
2. For each reference token, find the most similar generated token (greedy matching)
3. Compute Precision, Recall, F1 from the similarity scores

## Key Advantage Over ROUGE
Captures paraphrases and synonyms. "The car broke down" vs "The automobile malfunctioned" → high BERTScore, near-zero ROUGE.

## Key Limitation
Still operates at generation output — does not measure whether the retrieved context was complete or correct. A fluent hallucination can score high BERTScore if it happens to be semantically similar to the reference.

## Used In
[[evaluation-metrics]]
