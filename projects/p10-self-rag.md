# P10 — Self-RAG: The Agent That Critiques Itself

**Tagline:** "Build a RAG system that knows when it's wrong and fixes itself."
**Difficulty:** Advanced–Expert
**Concepts:** `[[agentic-rag]]` `[[agent-frameworks]]` `[[evaluation-metrics]]`
**Builds on:** P9 (multi-hop agent)
**Estimated time:** 6–8 hours

---

## What You Will Build
A RAG system that evaluates its own retrieval quality and generation quality at runtime, and self-corrects when either is poor. It decides whether to retrieve, whether retrieved content is relevant, and whether its own answer is grounded — all without human intervention.

---

## What You Will Learn
- How to implement Self-RAG's reflection tokens using LLM-as-judge
- How to build a corrective feedback loop in LangGraph
- How to detect hallucination at generation time
- Why self-evaluation is the key to production reliability

---

## The Self-Evaluation Loop

```
Query
  │
  ▼
Does this need retrieval?
  ├── No (simple factual) → answer directly
  └── Yes → retrieve
              │
              ▼
        Is retrieved content relevant?
          ├── Irrelevant → reformulate query → retrieve again
          └── Relevant → generate answer
                            │
                            ▼
                    Is answer grounded in context?
                      ├── Hallucinated → regenerate with stricter prompt
                      └── Grounded → is answer useful/complete?
                                        ├── Incomplete → retrieve more
                                        └── Complete → return answer ✅
```

---

## Requirements

### R1 — Retrieval Decision Gate
Build a function that decides whether a query needs retrieval:
- [ ] Prompt an LLM: *"Does answering this question require looking up a document, or can it be answered from general knowledge? Answer YES or NO with one sentence of reasoning."*
- [ ] If YES → proceed to retrieval
- [ ] If NO → answer directly (skip retrieval, save cost and latency)
- [ ] Test on 10 questions — 5 that need retrieval, 5 that don't
  - Needs retrieval: "What is the PhD fee at Punjabi University?"
  - Does not need: "What is 15% of 45,000?" or "What does 'semester' mean?"

### R2 — Relevance Evaluator
After retrieval, score each chunk for relevance:
- [ ] Prompt: *"On a scale of 1-5, how relevant is this passage to the query? 1=completely irrelevant, 5=directly answers it. Return only the number."*
- [ ] If average score < 3 → reformulate query and retrieve again
- [ ] If score ≥ 3 → proceed to generation
- [ ] Maximum 2 reformulation attempts before falling back to "not found"

```python
def evaluate_relevance(query: str, chunk: str, llm) -> int:
    prompt = f"""Query: {query}
Passage: {chunk}
Rate relevance 1-5 (1=irrelevant, 5=directly answers). Return only the number."""
    response = llm.invoke(prompt)
    return int(response.content.strip())
```

### R3 — Faithfulness Checker (Hallucination Detector)
After generating an answer, check if it's grounded:
- [ ] Prompt: *"Does the following answer contain information NOT present in the context? Answer YES (hallucinated) or NO (grounded)."*
- [ ] If hallucinated → regenerate with a stricter prompt: *"Answer using ONLY exact information from the context. Do not add anything not stated."*
- [ ] Maximum 2 regeneration attempts

```python
def check_faithfulness(answer: str, context: str, llm) -> bool:
    prompt = f"""Context: {context}

Answer: {answer}

Does the answer contain ANY information not present in the context?
Answer YES if hallucinated, NO if fully grounded."""
    response = llm.invoke(prompt)
    return "YES" in response.content.upper()
```

### R4 — Completeness Checker
Check if the answer actually addresses the question:
- [ ] Prompt: *"Does this answer fully address the question? If partial, what is missing?"*
- [ ] If incomplete → identify what's missing → retrieve specifically for the gap → regenerate

### R5 — Full Self-RAG Pipeline with LangGraph
Wire everything into a LangGraph:

```
Nodes:
  A: retrieval_decision    (R1)
  B: retrieve
  C: relevance_eval        (R2)
  D: query_reformulator
  E: generate
  F: faithfulness_check    (R3)
  G: completeness_check    (R4)
  H: return_answer

Edges:
  A → B (if needs retrieval) or E (if no retrieval needed)
  B → C
  C → D (if irrelevant) or E (if relevant)
  D → B (reformulated retrieval)
  E → F
  F → E (if hallucinated, regenerate) or G (if grounded)
  G → B (if incomplete) or H (if complete)
```

- [ ] Implement all nodes and edges
- [ ] Set a global `max_iterations=8` counter to prevent runaway loops
- [ ] Log every decision with reason: "Relevance: 2/5 — reformulating query"

### R6 — Evaluation: Self-RAG vs Standard RAG
- [ ] Run both systems on 15 questions
- [ ] Measure:
  - Faithfulness score (RAGAS)
  - Hallucination rate (count of hallucinated answers / total)
  - Answer correctness
  - Average latency per question
  - Average cost (number of LLM calls)
- [ ] Fill the table:

```
Metric              | Standard RAG (P6) | Self-RAG (P10)
────────────────────|─────────────────--|────────────────
Faithfulness        |                   |
Hallucination rate  |                   |
Answer correctness  |                   |
Avg latency (sec)   |                   |
Avg LLM calls/query |                   |
```

Expected finding: Self-RAG has higher faithfulness and lower hallucination rate, at the cost of higher latency and more LLM calls.

---

## Corrective RAG Extension (Bonus)
Add the CRAG pattern on top:
- [ ] If relevance score is very low (< 2/5) for ALL retrieved chunks → fall back to web search
- [ ] Use DuckDuckGo or Tavily API for web fallback
- [ ] Mark answers that came from web search vs document search in the output

---

## What Makes P10 Different from P8/P9
| Feature | P8 ReAct | P9 Multi-hop | P10 Self-RAG |
|---------|----------|-------------|-------------|
| Dynamic retrieval | ✅ | ✅ | ✅ |
| Multi-step reasoning | Partial | ✅ | ✅ |
| Relevance evaluation | ❌ | ❌ | ✅ |
| Hallucination detection | ❌ | ❌ | ✅ |
| Self-correction | ❌ | ❌ | ✅ |
| Production reliability | Medium | Medium | High |

---

## Bonus Challenges
- [ ] Replace LLM-as-judge with a fine-tuned small classifier for relevance/faithfulness (faster, cheaper)
- [ ] Add a confidence score to every answer: "Confidence: HIGH / MEDIUM / LOW — based on relevance score and faithfulness check"
- [ ] Integrate P10 into the P7 university assistant as its query engine

---

## Deliverable
`p10_self_rag.py` — the full LangGraph pipeline with all 4 self-evaluation gates.
```
python p10_self_rag.py
> Question: What is the research grant policy for associate professors?
[Gate 1] Needs retrieval: YES
[Retrieved 3 chunks] Relevance: 4/5, 3/5, 2/5 → proceeding
[Generated answer]
[Gate 3] Faithfulness check: GROUNDED ✅
[Gate 4] Completeness check: COMPLETE ✅
Answer: Associate professors are eligible for research grants up to ₹5 lakh...
Source: hr_policy_2025.pdf, Page 12
```
