---
title: Agentic RAG
type: concept
tags: agentic-rag, agents, self-rag, multi-hop, reasoning, tool-use
sources:
  - Yao et al. 2022 — "ReAct: Synergizing Reasoning and Acting in Language Models" (ICLR 2023)
  - Asai et al. 2023 — "Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection" (ICLR 2024)
  - Jiang et al. 2023 — "Active Retrieval Augmented Generation" (EMNLP 2023)
  - Yan et al. 2024 — "Corrective Retrieval Augmented Generation" (EMNLP Findings 2024)
updated: 2026-05-28
---

## Summary
Agentic RAG gives the LLM control over the retrieval process itself. Instead of retrieving once per query on a fixed schedule, an agent decides **when** to retrieve, **what** to retrieve, **how many times** to retrieve, and whether the retrieved result is good enough — repeating the loop if not.

---

## The Core Difference: Fixed vs Agentic

**Standard RAG (fixed pipeline):**
```
Query → Retrieve once → Generate answer
```
The pipeline is hardcoded. Retrieval always happens exactly once, before generation.

**Agentic RAG (dynamic loop):**
```
Query
  │
  ▼
Agent THINKS: "Do I need to retrieve? What should I search for?"
  │
  ├── Retrieve if needed → Evaluate result → "Is this enough?"
  │     ├── Yes → Generate answer
  │     └── No  → Reformulate query → Retrieve again → ...
  │
  └── Generate final answer when confident
```

The agent decides the retrieval strategy at runtime based on the question and what it finds.

---

## Why Standard RAG Is Not Enough for Complex Questions

Standard RAG fails on:

| Question type | Why standard RAG fails |
|--------------|----------------------|
| Multi-hop: "Who founded the company that makes the drug approved for X?" | Requires 2–3 retrievals; standard RAG retrieves once |
| Ambiguous: "Tell me about the policy" | Unclear what to retrieve; agent needs to ask clarifying questions |
| Comparative: "Which department has the highest budget vs last year?" | Needs multiple documents; one retrieval is insufficient |
| Iterative: "Summarise all safety incidents in the last 5 years" | Needs to retrieve across many documents systematically |

---

## The Four Core Agentic RAG Patterns

### Pattern 1 — ReAct (Reasoning + Acting)
**Paper:** Yao et al. 2022 — ICLR 2023

The LLM alternates between **Thought** (reasoning about what to do) and **Action** (calling a tool like search), observing the result before deciding the next step.

```
Thought: "The question asks about the drug approval date.
          I need to search for 'Drug X FDA approval'."
Action:  search("Drug X FDA approval")
Observation: "Drug X was approved by FDA on March 15, 2021."

Thought: "Now I need the company that makes Drug X."
Action:  search("Drug X manufacturer")
Observation: "Drug X is manufactured by PharmaCorp."

Thought: "Now I can answer the original question."
Action:  finish("PharmaCorp's Drug X was approved by the FDA on March 15, 2021.")
```

Each Thought-Action-Observation cycle is one retrieval loop. The agent decides when to stop.

**Key property:** The agent can use multiple tools — not just RAG search. It can call calculators, APIs, databases, or code executors alongside document retrieval.

---

### Pattern 2 — Self-RAG
**Paper:** Asai et al. 2023 — ICLR 2024

The LLM is trained (fine-tuned) to generate special reflection tokens that control its own retrieval and output quality:

| Token | Meaning | When generated |
|-------|---------|---------------|
| `[Retrieve]` | "I need to retrieve" | When parametric knowledge is insufficient |
| `[No Retrieve]` | "I can answer from memory" | For simple factual questions |
| `[Relevant]` | "Retrieved passage is relevant" | After seeing retrieved chunk |
| `[Irrelevant]` | "Retrieved passage is not useful" | After seeing retrieved chunk |
| `[Supported]` | "My answer is grounded in context" | Self-critique of own output |
| `[Contradicts]` | "My answer contradicts context" | Self-critique triggers correction |

```
Query: "What is the side effect of Drug X?"

[Retrieve] → retrieves chunk about Drug X
[Relevant] → chunk is about Drug X side effects ✅
Generates answer
[Supported] → answer is grounded in chunk ✅
Returns answer
```

vs.

```
Query: "What is the capital of France?"
[No Retrieve] → answers from parametric knowledge directly
Returns "Paris"
```

**Key insight:** Not every question needs retrieval. Self-RAG avoids unnecessary retrieval on simple questions, saving time and cost.

---

### Pattern 3 — Corrective RAG (CRAG)
**Paper:** Yan et al. 2024 — EMNLP Findings 2024

Adds a lightweight **relevance evaluator** after retrieval. If retrieved documents score low on relevance, the system automatically falls back to web search or broader retrieval before generating.

```
Query → Retrieve → Relevance Score
                        │
              ┌─────────┴──────────┐
           High                  Low
              │                   │
         Refine chunks        Web search
              │               (broader retrieval)
              └─────────┬──────────┘
                        │
                   Generate answer
```

**Three relevance outcomes:**
- **Correct** (score ≥ 0.5): use retrieved docs directly
- **Incorrect** (score < 0.1): discard, use web search instead
- **Ambiguous** (0.1–0.5): combine retrieved docs + web search

---

### Pattern 4 — Active RAG (FLARE)
**Paper:** Jiang et al. 2023 — EMNLP 2023

**Forward-Looking Active REtrieval** — the LLM generates a tentative answer token by token, and triggers retrieval whenever it is about to generate a low-confidence token.

```
Generating: "The patient should take..."
Confidence check: next token "500mg" → confidence LOW (drug-specific fact)
→ Trigger retrieval: search("Drug X dosage")
→ Retrieved: "Drug X: 500mg every 8 hours"
→ Continue generating with retrieved context: "500mg every 8 hours."
```

**Key difference from standard RAG:** Retrieval is triggered mid-generation, not before it. The LLM itself identifies where it needs help.

---

## Agentic RAG vs Standard RAG — Full Comparison

| Dimension | Standard RAG | Agentic RAG |
|-----------|-------------|-------------|
| Retrieval timing | Once, before generation | Dynamic — when needed |
| Number of retrievals | Always 1 | 1 to N |
| Query reformulation | None | Agent can rewrite queries |
| Self-evaluation | None | Agent critiques own output |
| Multi-hop questions | Fails | Handles naturally |
| Latency | Low (one retrieval) | Higher (multiple loops) |
| Cost | Low | Higher (multiple LLM calls) |
| Complexity | Simple | Requires agent framework |
| Best for | Simple factual Q&A | Complex, multi-step questions |

---

## When to Use Agentic RAG

**Use standard RAG when:**
- Questions are simple and single-hop ("What is the leave policy?")
- Latency is critical (< 2 seconds response required)
- Cost is constrained

**Use agentic RAG when:**
- Questions require multiple retrievals ("Compare department budgets for 2023 and 2024")
- Questions are ambiguous and need clarification
- Answers require reasoning across several documents
- You need the system to self-correct when retrieval quality is poor

---

## Key Risks of Agentic RAG

| Risk | Description | Mitigation |
|------|-------------|-----------|
| **Infinite loops** | Agent keeps retrieving without stopping | Set max iterations (e.g. 5) |
| **Higher cost** | Multiple LLM calls per query | Use smaller model for planning, larger for final answer |
| **Hallucinated tool calls** | Agent calls tools with wrong parameters | Validate tool inputs strictly |
| **Slow responses** | Multiple round trips add latency | Show streaming progress to user |
| **Unpredictable behaviour** | Hard to debug multi-step reasoning | Log every Thought-Action-Observation step |

## Related
[[rag]] · [[agent-frameworks]] · [[production-rag-patterns]] · [[evaluation-metrics]]
