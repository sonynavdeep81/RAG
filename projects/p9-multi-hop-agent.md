# P9 — Multi-Hop Reasoning Agent

**Tagline:** "Answer questions that require connecting dots across multiple documents."
**Difficulty:** Advanced
**Concepts:** `[[agentic-rag]]` `[[agent-frameworks]]` `[[production-rag-patterns]]`
**Builds on:** P8 (ReAct agent)
**Estimated time:** 5–6 hours

---

## What You Will Build
An agent that automatically decomposes complex questions into sub-questions, retrieves answers for each sub-question independently, and synthesises a final answer by combining the partial results. This is how enterprise RAG handles research-style questions.

---

## What You Will Learn
- How to decompose complex questions into atomic sub-questions
- How LlamaIndex's sub-question query engine works
- How to route different sub-questions to different document collections
- How to synthesise multiple partial answers into one coherent response

---

## The Core Idea: Sub-Question Decomposition

```
Complex question:
"Compare the research output and budget allocation of the CS and Physics departments,
 and identify which department gets more funding per publication."

Decomposed into:
  Sub-Q1: "What is the CS department research output (publications)?"
  Sub-Q2: "What is the Physics department research output (publications)?"
  Sub-Q3: "What is the CS department budget allocation?"
  Sub-Q4: "What is the Physics department budget allocation?"

Each sub-question retrieved independently → answers combined:
  CS: 45 publications, ₹2.1 crore budget → ₹4.67 lakh per publication
  Physics: 38 publications, ₹1.8 crore budget → ₹4.74 lakh per publication
  → Physics gets slightly more funding per publication
```

---

## Requirements

### R1 — Multi-Document Index Setup
- [ ] Create **separate FAISS indices** for at least 3 different document collections:
  - `academic_docs/` — course catalogues, examination rules, academic calendar
  - `finance_docs/` — fee structures, budget reports, scholarship information
  - `hr_docs/` — HR policies, leave rules, employee handbook
- [ ] Each collection gets its own LlamaIndex `VectorStoreIndex`
- [ ] Wrap each as a separate `QueryEngineTool` with a clear description of what it contains

### R2 — Sub-Question Query Engine
- [ ] Use `LlamaIndex SubQuestionQueryEngine` which automatically:
  - Breaks the complex question into sub-questions
  - Routes each sub-question to the most relevant tool
  - Gathers all sub-answers
  - Synthesises a final answer

```python
from llama_index.core.query_engine import SubQuestionQueryEngine
from llama_index.core.tools import QueryEngineTool, ToolMetadata

tools = [
    QueryEngineTool(query_engine=academic_engine,
        metadata=ToolMetadata(name="academic", description="Academic rules, courses, exams")),
    QueryEngineTool(query_engine=finance_engine,
        metadata=ToolMetadata(name="finance", description="Fees, budgets, scholarships")),
    QueryEngineTool(query_engine=hr_engine,
        metadata=ToolMetadata(name="hr", description="HR policies, leave, employment")),
]

engine = SubQuestionQueryEngine.from_defaults(query_engine_tools=tools, verbose=True)
response = engine.query(
    "What is the fee waiver policy for PhD students and how does it interact with the leave policy?"
)
```

### R3 — Routing Evaluation
- [ ] Write 10 complex questions that span at least 2 document collections
- [ ] For each, note which sub-questions were generated and which tools were called
- [ ] Verify routing was correct — did finance questions go to finance? academic to academic?
- [ ] Identify any misrouting and fix it by improving tool descriptions

### R4 — Manual Multi-Hop with LangGraph
Build a custom multi-hop pipeline using LangGraph (for full control):
- [ ] **Node 1 — Planner:** LLM decomposes the question into a list of sub-questions
- [ ] **Node 2 — Retriever:** For each sub-question, retrieve from the appropriate index
- [ ] **Node 3 — Synthesiser:** LLM combines all sub-answers into one final answer
- [ ] **Edge condition:** If synthesiser says "insufficient information", loop back to Planner for refined sub-questions

```python
# Graph structure
Planner → Retriever → Synthesiser
              ↑              │
              └──────────────┘ (if insufficient)
```

### R5 — Comparison: Standard RAG vs Multi-Hop Agent
- [ ] Take 5 cross-document questions
- [ ] Run on: (a) standard P6 RAG, (b) P8 ReAct agent, (c) P9 multi-hop agent
- [ ] Score each answer: correct / partial / wrong
- [ ] Fill the comparison table and write a conclusion

---

## Test Questions Designed for Multi-Hop
Use these (adapt to your actual documents):
1. "What scholarship is available for PhD students and what GPA is required to maintain it?"
2. "Can a faculty member on medical leave still supervise PhD students per university policy?"
3. "What is the total cost of a 3-year PhD including tuition, hostel, and exam fees?"
4. "Which department offers the most elective courses and what is their average class size?"
5. "How does the fee structure for international students differ from domestic students?"

---

## Bonus Challenges
- [ ] Add a `web_search` tool as a fallback when no document collection has the answer
- [ ] Add a confidence score to each sub-answer — if confidence < 0.5, flag it in the final answer
- [ ] Build a "citation map" showing exactly which document + page each piece of the final answer came from

---

## Deliverable
`p9_multihop_agent.py` with both the LlamaIndex SubQuestion engine and the LangGraph manual implementation.
