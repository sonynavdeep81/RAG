# P8 — ReAct RAG Agent

**Tagline:** "Your RAG system learns to think before it retrieves."
**Difficulty:** Advanced
**Concepts:** `[[agentic-rag]]` `[[agent-frameworks]]` `[[rag]]`
**Builds on:** P6 (production RAG pipeline)
**Estimated time:** 4–5 hours

---

## What You Will Build
A ReAct agent that uses your P6 RAG system as one of its tools — alongside a calculator and a date tool. The agent decides which tool to call, in what order, and stops when it has a confident answer. It handles multi-step questions that standard RAG cannot.

---

## What You Will Learn
- How the Thought → Action → Observation loop works in practice
- How tool descriptions control agent behaviour
- Why agents sometimes loop incorrectly and how to fix it
- The difference between function-calling and text-based ReAct

---

## Requirements

### R1 — Define Your Tools
Build three tools:
- [ ] **`search_documents(query)`** — wraps your P6 hybrid retriever (dense + BM25 + reranker)
- [ ] **`calculate(expression)`** — evaluates arithmetic safely using `numexpr` or `ast.literal_eval`
- [ ] **`get_current_date()`** — returns today's date (useful for "this year" type questions)

Each tool must have a clear description explaining:
- What it does
- When to use it
- What it returns

### R2 — Build the ReAct Agent
- [ ] Use `LangChain AgentExecutor` with the ReAct prompt
- [ ] Set `verbose=True` so you can see every Thought/Action/Observation
- [ ] Set `max_iterations=6` to prevent infinite loops
- [ ] Use `handle_parsing_errors=True`

### R3 — Single-Hop vs Multi-Hop Test
Test these question types and record what happens:

**Single-hop** (one retrieval needed):
- "What is the PhD admission fee?"
- "Who is the head of the Computer Science department?"

**Multi-hop** (two retrievals needed):
- "What is the difference between the PhD fee in 2024 and 2025?"
- "Which department has more students — CS or EE — and by how many?"

**Arithmetic** (retrieval + calculation):
- "If the semester fee is ₹45,000, what is the annual fee for a 2-year Masters programme?"

- [ ] Record: for each question — how many iterations, which tools were called, was the answer correct?

### R4 — Failure Analysis
- [ ] Identify one question where the agent failed or looped unnecessarily
- [ ] Diagnose why: wrong tool description? ambiguous question? irrelevant chunks retrieved?
- [ ] Fix it — either improve the tool description or rewrite the question, and show the improved result

### R5 — Conversation Memory
- [ ] Add `ConversationBufferMemory` to the agent
- [ ] Run a 3-turn conversation where each question builds on the previous answer:
  ```
  Turn 1: "What is the CS department budget?"
  Turn 2: "And what about EE?"
  Turn 3: "Which is larger and by what percentage?"
  ```
- [ ] Verify the agent correctly uses context from previous turns

---

## Key Experiment
Run the same 5 questions on:
1. Your P6 standard RAG (one fixed retrieval)
2. Your P8 ReAct agent (dynamic retrieval)

Fill this table:

```
Question              | P6 Correct? | P8 Correct? | P8 Iterations
──────────────────────|─────────────|─────────────|--------------
Single-hop Q1         |             |             |
Single-hop Q2         |             |             |
Multi-hop Q1          |             |             |
Multi-hop Q2          |             |             |
Arithmetic Q1         |             |             |
```

Expected finding: P6 and P8 tie on single-hop; P8 wins on multi-hop and arithmetic.

---

## Bonus Challenges
- [ ] Add a `web_search(query)` tool using DuckDuckGo or Tavily API — use it when document search fails
- [ ] Add a `list_available_documents()` tool so the agent can see what's indexed before searching
- [ ] Switch from ReAct to function calling — compare reliability

---

## Deliverable
`p8_react_agent.py` — runnable with an interactive question loop.
```
python p8_react_agent.py
> Ask a question: What is the difference in fees between 2024 and 2025?
[Thinking...]
[Action: search_documents("PhD fee 2025")]
[Action: search_documents("PhD fee 2024")]
[Action: calculate("45000 - 42000")]
Answer: The fee increased by ₹3,000 (7.1%) from 2024 to 2025.
```
