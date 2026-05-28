---
title: LangChain Agents Documentation
type: source
tags: langchain, agents, tools, react, function-calling
updated: 2026-05-28
---

**Maintainer:** LangChain Inc. (Harrison Chase et al.)
**Type:** Official framework documentation
**URL:** python.langchain.com/docs/modules/agents
**GitHub:** github.com/langchain-ai/langchain (90k+ stars)
**Most widely used LLM framework globally**

## What LangChain Agents Provide
The infrastructure for building ReAct-style agents: tool definition, prompt templates, the Thought-Action-Observation loop, and result parsing — all pre-built so you focus on defining tools and logic.

## Core Components
| Component | What it does |
|-----------|-------------|
| `@tool` decorator | Wraps any Python function into a tool with name + description |
| `create_react_agent` | Builds a ReAct agent from LLM + tools + prompt |
| `AgentExecutor` | Runs the agent loop, handles errors, enforces max iterations |
| `ConversationBufferMemory` | Gives the agent memory of previous turns |
| LangChain Hub | Pre-built prompts — `hwchase17/react` is the standard ReAct prompt |

## Why It's the Best Starting Point
- Largest community (most Stack Overflow answers, tutorials, examples)
- `verbose=True` prints every Thought/Action/Observation — makes learning transparent
- Extensive integrations: 200+ tools pre-built (web search, Wikipedia, calculator, SQL)
- Easy to upgrade to LangGraph when you need more control

## Key Limitation
`AgentExecutor` is a black box — the loop logic is hidden. For production systems needing fine-grained control, migrate to LangGraph (which is also by LangChain Inc.).

## Used In
[[agent-frameworks]]
