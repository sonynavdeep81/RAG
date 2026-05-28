---
title: ReAct — Synergizing Reasoning and Acting in Language Models
type: source
tags: react, agents, reasoning, tool-use, agentic-rag
updated: 2026-05-28
---

**Authors:** Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, Yuan Cao
**Affiliation:** Princeton University + Google Brain
**Venue:** ICLR 2023 (top ML conference)
**arXiv:** 2210.03629
**Citation count:** 3,000+

## What It Introduced
ReAct — a prompting framework that interleaves **Reasoning** (Thought) and **Acting** (Action + Observation) traces in a single LLM prompt. The LLM alternates between thinking about what to do and actually doing it (calling a tool), observing the result, and repeating.

## The Core Loop
```
Thought: reasoning about the current state and what to do next
Action:  calling a tool (search, lookup, calculate)
Observation: the tool's output
... repeat until ...
Final Answer: the concluded response
```

## Key Findings
| Finding | Detail |
|---------|--------|
| ReAct outperforms chain-of-thought alone | Acting grounds reasoning in real retrieved facts |
| ReAct outperforms acting alone | Reasoning helps the agent recover from wrong tool calls |
| Works on diverse tasks | HotpotQA (multi-hop QA), FEVER (fact verification), ALFWorld (interactive tasks) |
| Human interpretable | The Thought trace explains every decision the agent makes |

## Why It Became the Standard
ReAct is the foundation that LangChain's `AgentExecutor`, LlamaIndex's `ReActAgent`, and most production agent frameworks are built on. Understanding ReAct means understanding how every major agent framework works internally.

## Key Limitation
Text-based action parsing is fragile — if the LLM outputs `Action: Search` instead of `Action: search`, parsing breaks. Function calling (structured JSON) was developed to address this.

## Used In
[[agentic-rag]] · [[agent-frameworks]]
