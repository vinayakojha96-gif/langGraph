# LangGraph Learning Project

A set of Jupyter notebooks exploring [LangGraph](https://github.com/langchain-ai/langgraph) and [LangChain](https://github.com/langchain-ai/langchain), built up from a plain state graph to a tool-calling agent with memory, tracing, and human-in-the-loop approval — using Google Gemini as the LLM and live stock prices (via `yfinance`) as the running example.

## Setup

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Copy `.env.example` to `.env` and fill in your own keys:

```bash
cp .env.example .env
```

| Variable | Purpose |
|---|---|
| `GOOGLE_API_KEY` | Gemini API key (from [Google AI Studio](https://ai.google.dev/)), used by `init_chat_model(..., model_provider="google_genai")` |
| `LANGSMITH_API_KEY` | API key for [LangSmith](https://smith.langchain.com/) tracing |
| `LANGSMITH_TRACING` | Set to `true` to enable tracing |
| `LANGSMITH_ENDPOINT` | LangSmith API endpoint |
| `LANGSMITH_PROJECT` | Project name traces are grouped under in LangSmith |

## Notebooks

| # | Notebook | What it covers |
|---|---|---|
| 1 | [`1_simple_graph.ipynb`](1_simple_graph.ipynb) | The basics of LangGraph: a `TypedDict` state, plain node functions, and a linear `StateGraph` (`START → calc_total → convert_to_inr → END`) that converts a USD amount to INR. |
| 2 | [`2_grpah.ipynb`](2_grpah.ipynb) | Introduces **conditional edges** — `add_conditional_edges` branches to either an INR or EUR conversion node based on a `target_currency` field in the state, then both branches join back into `END`. |
| 3 | [`3_chatbot.ipynb`](3_chatbot.ipynb) | A minimal LLM chatbot graph: one `chatbot` node wraps a Gemini chat model, state accumulates conversation history via `add_messages`, and the notebook ends with an interactive terminal chat loop. |
| 4 | [`4_toolCall.ipynb`](4_toolCall.ipynb) | **Tool calling**: defines a `get_stock_price` tool (backed by `yfinance` for real, live prices) and binds it to the LLM with `bind_tools`. Uses `ToolNode` / `tools_condition` to let the graph decide when to invoke the tool. |
| 5 | [`5_toolCall_agent.ipynb`](5_toolCall_agent.ipynb) | Extends notebook 4 into a proper **agent loop**: adds an edge from `tools` back to `chatbot`, so the model can call the tool, see the result, and keep reasoning (e.g. pricing out a multi-stock purchase) instead of stopping after one tool call. |
| 6 | [`6_memory_toolCall_agent.ipynb`](6_memory_toolCall_agent.ipynb) | Adds **persistent memory** to the agent using `MemorySaver` as a checkpointer, keyed by `thread_id`. Lets a follow-up question ("add it to the previous total") reference earlier turns in the same conversation thread. |
| 7 | [`7_langsmith.ipynb`](7_langsmith.ipynb) | Same tool-calling agent, wrapped with LangSmith's `@traceable` decorator so every graph/tool/LLM call is traced and viewable in the LangSmith dashboard for debugging and observability. |
| 8 | [`8_human_in_the_loop.ipynb`](8_human_in_the_loop.ipynb) | Adds a **human-in-the-loop approval step**: a `buy_stocks` tool calls `interrupt(...)` to pause the graph and ask for explicit approval before "buying" stock, then resumes execution with `Command(resume=decision)` once the user responds. |

## Tech stack

- **LangGraph** — graph orchestration / state machine for the agent
- **LangChain** (`langchain`, `langchain-core`, `langchain-google-genai`) — LLM abstraction and tool-calling interface
- **Google Gemini** — the underlying LLM (via `init_chat_model`)
- **yfinance** — live stock price data used as an example tool
- **LangSmith** — tracing/observability for graph runs
