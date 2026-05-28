---
title: Agent Frameworks for RAG
type: concept
tags: langchain, langgraph, llamaindex, agents, tools, function-calling
sources:
  - LangGraph documentation — langchain-ai.github.io/langgraph
  - LlamaIndex Agents documentation — docs.llamaindex.ai
  - LangChain Agents documentation — python.langchain.com
  - OpenAI Function Calling documentation
updated: 2026-05-28
---

## Summary
Agent frameworks provide the infrastructure to build agentic RAG systems — they handle tool definition, the Thought-Action-Observation loop, state management, and multi-step execution so you focus on logic rather than plumbing.

---

## What an Agent Needs (Infrastructure)

Any agentic RAG system needs four things:

```
1. TOOLS          — functions the agent can call (search, calculate, lookup)
2. LLM BACKBONE   — the model that reasons and decides which tool to call
3. LOOP MANAGER   — runs Thought → Action → Observation → repeat
4. STATE STORE    — remembers what happened in previous steps of this conversation
```

Frameworks provide 3 and 4 out of the box. You provide 1 and 2.

---

## Tool Definition — The Foundation

A tool is any function wrapped with a name and description. The LLM reads the description to decide when to call it.

```python
from langchain.tools import tool

@tool
def search_university_documents(query: str) -> str:
    """Search university policy and administrative documents.
    Use this when the question is about university rules, fees,
    procedures, or departmental information.
    Args:
        query: The search query string
    Returns:
        Relevant text passages from university documents
    """
    # Your FAISS retrieval logic here
    chunks = retrieve(query, index, k=3)
    return "\n\n".join(chunk.text for chunk in chunks)

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression.
    Use this for arithmetic, percentages, or comparisons.
    Args:
        expression: A valid Python math expression e.g. '(150 + 200) / 2'
    """
    return str(eval(expression))  # use safely in production
```

**Critical rule:** The description is what the LLM uses to decide when to call the tool. Write it as if explaining to a person what the tool does and when to use it. Vague descriptions → wrong tool calls.

---

## Framework 1 — LangChain Agents

**Best for:** Getting started, simple single-agent systems, standard ReAct loop.

```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain_openai import ChatOpenAI
from langchain import hub

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
tools = [search_university_documents, calculate]

# Pull the standard ReAct prompt from LangChain hub
prompt = hub.pull("hwchase17/react")

agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,       # prints every Thought/Action/Observation
    max_iterations=5,   # prevent infinite loops
    handle_parsing_errors=True
)

result = agent_executor.invoke({
    "input": "What is the fee for a PhD student and how does it compare to last year?"
})
print(result["output"])
```

**What `verbose=True` prints:**
```
Thought: I need to find the current PhD fee and last year's fee.
Action: search_university_documents
Action Input: "PhD student fee structure 2025"
Observation: PhD programme fee for 2025-26: ₹45,000 per semester...

Thought: Now I need last year's fee.
Action: search_university_documents
Action Input: "PhD student fee structure 2024"
Observation: PhD programme fee for 2024-25: ₹42,000 per semester...

Thought: I can now calculate the difference.
Action: calculate
Action Input: "(45000 - 42000) / 42000 * 100"
Observation: 7.142857142857142

Thought: I have all the information needed.
Final Answer: The PhD fee increased from ₹42,000 to ₹45,000 per semester — a 7.1% increase.
```

---

## Framework 2 — LangGraph

**Best for:** Complex multi-agent systems, conditional branching, human-in-the-loop, production-grade control flow.

LangGraph models your agent as a **graph** — nodes are actions (LLM call, tool call, condition check) and edges are transitions between them. This gives you precise control over when and how the agent loops.

```python
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from langchain_openai import ChatOpenAI
from typing import TypedDict, Annotated
import operator

# Define state that flows through the graph
class AgentState(TypedDict):
    messages: Annotated[list, operator.add]

# Nodes
llm = ChatOpenAI(model="gpt-4o-mini").bind_tools(tools)

def call_llm(state: AgentState):
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

def should_continue(state: AgentState):
    last_message = state["messages"][-1]
    if last_message.tool_calls:
        return "tools"     # LLM wants to call a tool → go to tool node
    return END             # LLM gave final answer → stop

# Build graph
graph = StateGraph(AgentState)
graph.add_node("llm", call_llm)
graph.add_node("tools", ToolNode(tools))
graph.set_entry_point("llm")
graph.add_conditional_edges("llm", should_continue)
graph.add_edge("tools", "llm")   # after tool → back to LLM

app = graph.compile()
result = app.invoke({"messages": [("user", "What is the total budget for the CS department?")]})
```

**Why LangGraph over basic LangChain agents:**
- You see and control exactly when the loop continues or stops
- You can add human approval steps before certain tool calls
- You can branch based on retrieved content (e.g. if relevance is low, try web search)
- Easier to debug because the graph is explicit

---

## Framework 3 — LlamaIndex Agents

**Best for:** Document-heavy agentic RAG, when your primary tools are document indices.

```python
from llama_index.core.agent import ReActAgent
from llama_index.core.tools import QueryEngineTool, ToolMetadata
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader

# Build a query engine over your documents
documents = SimpleDirectoryReader("./university_docs").load_data()
index = VectorStoreIndex.from_documents(documents)
query_engine = index.as_query_engine(similarity_top_k=3)

# Wrap as a tool
rag_tool = QueryEngineTool(
    query_engine=query_engine,
    metadata=ToolMetadata(
        name="university_documents",
        description="Search university policies, fees, regulations, and reports."
    )
)

# Build ReAct agent
agent = ReActAgent.from_tools(
    tools=[rag_tool],
    llm=llm,
    verbose=True,
    max_iterations=5
)

response = agent.chat("Compare the CS and EE department budgets from the annual report.")
print(response)
```

**LlamaIndex-specific strengths:**
- Built-in support for multi-document agents (each document gets its own tool)
- Sub-question decomposition: automatically splits complex questions into sub-questions
- OpenAI function calling natively integrated

---

## Memory — Giving Agents Conversation History

Without memory, every query to the agent starts fresh. With memory, the agent remembers previous exchanges in the same session.

```python
from langchain.memory import ConversationBufferMemory
from langchain.agents import AgentExecutor

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,
    verbose=True
)

# Turn 1
agent_executor.invoke({"input": "What is the PhD fee?"})
# Answer: ₹45,000 per semester

# Turn 2 — agent remembers the previous exchange
agent_executor.invoke({"input": "And what about the Masters fee?"})
# Agent knows this is a follow-up about fees, not a fresh question
```

**Memory types:**

| Type | What it stores | Use case |
|------|---------------|----------|
| `ConversationBufferMemory` | Full conversation history | Short sessions |
| `ConversationSummaryMemory` | LLM-summarised history | Long sessions |
| `ConversationBufferWindowMemory` | Last N exchanges only | Memory-constrained |
| Vector memory | Semantic search over past exchanges | Very long sessions |

---

## Function Calling vs ReAct

Two ways to make LLMs call tools:

| Approach | How it works | Models that support it |
|----------|-------------|----------------------|
| **ReAct (text-based)** | LLM outputs `Action: tool_name` as text; parser extracts it | Any instruction-following LLM |
| **Function Calling** | LLM outputs structured JSON specifying tool name + parameters | GPT-4, Claude, Gemini, Llama-3 |

Function calling is more reliable — structured JSON is harder to misparse than free-form text.

```python
# Function calling with OpenAI
from openai import OpenAI

tools_spec = [{
    "type": "function",
    "function": {
        "name": "search_university_documents",
        "description": "Search university policy documents",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "The search query"}
            },
            "required": ["query"]
        }
    }
}]

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "What is the leave policy?"}],
    tools=tools_spec,
    tool_choice="auto"
)

if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    # Execute the tool, feed result back to model
```

---

## Choosing the Right Framework

```
How complex is your agent?
│
├── Single agent, simple Q&A with retrieval
│   └── LangChain AgentExecutor + ReAct  (simplest to build)
│
├── Document-heavy, multiple document collections
│   └── LlamaIndex ReActAgent  (best native document tool support)
│
├── Complex flow, conditional branching, human approval steps
│   └── LangGraph  (most control, best for production)
│
└── Just starting out / learning
    └── LangChain AgentExecutor  (most tutorials, easiest to debug)
```

## Related
[[agentic-rag]] · [[rag]] · [[production-rag-patterns]] · [[embeddings-and-vector-search]]
