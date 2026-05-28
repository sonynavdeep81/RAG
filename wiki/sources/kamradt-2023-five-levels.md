---
title: Five Levels of Text Splitting
type: source
tags: chunking, text-splitting, practical, langchain
updated: 2026-05-28
---

**Author:** Greg Kamradt  
**Type:** GitHub notebook + YouTube video (2023)  
**URL:** github.com/FullStackRetrieval-com/RetrievalTutorials  
**Status:** Widely cited practitioner reference — used in LangChain docs, Pinecone guides, and hundreds of RAG tutorials

## What It Is
A hands-on framework organising chunking strategies from simplest (Level 1) to most intelligent (Level 5). Not a peer-reviewed paper — a practitioner's guide that became the industry standard reference for chunking strategy selection.

## The Five Levels
| Level | Strategy | Key point |
|-------|----------|-----------|
| 1 | Fixed character | Never use in production |
| 2 | Recursive character | Industry default — fast, free, generally good |
| 3 | Document structure | Best for docs with clear headers/sections |
| 4 | Semantic | Best for narrative where topic shifts matter |
| 5 | Agentic / LLM-based | Best quality; highest cost |

## Why It's Trusted
Kamradt is a prominent RAG educator cited by LangChain, Pinecone, and LlamaIndex official documentation. The framework is simple enough to be actionable and detailed enough to be correct.

## Used In
[[chunking]] · [[chunking-strategies]]
