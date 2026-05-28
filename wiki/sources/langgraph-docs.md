---
title: LangGraph Documentation
type: source
tags: langgraph, agents, graph, state-machine, production
updated: 2026-05-28
---

**Maintainer:** LangChain Inc.
**Type:** Official framework documentation
**URL:** langchain-ai.github.io/langgraph
**GitHub:** github.com/langchain-ai/langgraph (8k+ stars)
**First released:** 2024

## What LangGraph Is
A library for building stateful, multi-actor applications with LLMs, built on top of LangChain. Models agent workflows as **directed graphs** where nodes are computation steps (LLM calls, tool calls, conditions) and edges are transitions between them.

## Why It Exists
Standard LangChain `AgentExecutor` is a black box — it runs the Thought-Action-Observation loop internally with little control. LangGraph makes the loop explicit and modifiable:
- You define exactly when the loop continues or stops
- You can add conditional branches (if relevance low → try web search)
- You can add human-in-the-loop approval before certain actions
- You can inspect and modify state at any node

## Core Concepts
| Concept | Description |
|---------|-------------|
| `StateGraph` | The graph that holds all nodes and edges |
| `State` | A TypedDict that flows through every node and accumulates results |
| `Node` | A Python function that reads state and returns updated state |
| `Edge` | A transition between nodes; can be conditional |
| `Checkpoint` | Saves graph state for resumption (human-in-the-loop, crash recovery) |
| `compile()` | Locks the graph into a runnable app |

## When to Use LangGraph vs LangChain AgentExecutor
| Use LangChain AgentExecutor | Use LangGraph |
|-----------------------------|--------------|
| Simple single-agent Q&A | Complex branching logic |
| Prototyping | Production deployment |
| Standard ReAct loop is sufficient | Need custom loop control |
| | Human approval steps needed |
| | Multi-agent coordination |

## Used In
[[agent-frameworks]]
