---
title: LlamaIndex Agents Documentation
type: source
tags: llamaindex, agents, sub-question, query-engine, tools
updated: 2026-05-28
---

**Maintainer:** LlamaIndex (Jerry Liu, Simon Suo et al.)
**Type:** Official framework documentation
**URL:** docs.llamaindex.ai/en/stable/module_guides/deploying/agents
**GitHub:** github.com/run-llama/llama_index (37k+ stars)
**Framework focus:** Document-centric AI applications

## What LlamaIndex Agents Are
LlamaIndex provides agent infrastructure specifically designed for document-heavy RAG applications. Unlike LangChain which is general-purpose, LlamaIndex agents have native understanding of document indices, query engines, and retrieval pipelines as first-class tools.

## Key Agent Types
| Agent | Description | Best for |
|-------|-------------|---------|
| `ReActAgent` | Standard ReAct loop over any tools | General purpose |
| `OpenAIAgent` | Uses OpenAI function calling | Reliability, structured outputs |
| `SubQuestionQueryEngine` | Decomposes complex questions into sub-questions | Multi-document Q&A |

## SubQuestionQueryEngine — The Standout Feature
Automatically decomposes a complex question into sub-questions, routes each to the most appropriate tool (document index), retrieves answers, and synthesises a final response. This is what makes LlamaIndex uniquely strong for multi-document RAG.

```
Complex question → [Sub-Q1, Sub-Q2, Sub-Q3]
                        │        │        │
                   Tool A   Tool B   Tool A
                        │        │        │
                   Answer1  Answer2  Answer3
                        └────────┴────────┘
                              │
                        Final answer
```

## Key Advantage Over LangChain for Document RAG
- Every document index can be wrapped as a tool with minimal code
- Query engines handle retrieval, reranking, and synthesis internally
- Built-in support for multi-document routing without custom code

## Used In
[[agent-frameworks]]
