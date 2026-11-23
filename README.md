# 🎓 Autonomous Research Agent Workshop

Build an autonomous **Research Agent** with **LangChain & LangGraph** (ReAct framework). The agent plans its research, picks the right tool, reads sources, and delivers a structured report with references.

## ✨ Features

- **ReAct agent** built with LangGraph's `create_react_agent`
- **Swappable LLM**: Groq (default), Google Gemini, or Hugging Face, changed with one variable
- **4 research tools**: smart web search, ArXiv, Wikipedia, and a web page reader
- **Improved search quality**: query expansion, multi-search, deduplication, and reranking
- **Robust tool handling**: tool errors (rate limits, timeouts) are sent back to the model so it can retry instead of crashing the run
- **Structured output**: Executive Summary, Key Findings, Supporting Evidence, and References

## 🧰 Tools

| Tool | Purpose |
|---|---|
| `web_search` | Recent news, benchmarks, and real-world implementations. Uses Tavily with query expansion (see below). |
| `arxiv_academic_search` | Scientific papers and preprints (AI, physics, math, CS). |
| `wikipedia_search` | Definitions, history, and background concepts. |
| `read_webpage` | Reads the full text of a single URL (via Tavily Extract) to verify details beyond search snippets. |

## 🔎 How Search Quality Is Improved

The plain Tavily call is replaced by a smart `web_search` tool:

1. **Query expansion**: the LLM rewrites the question into several variants (technical terms, recency, specific entities).
2. **Deeper search**: every variant runs with Tavily's `search_depth="advanced"`.
3. **Deduplication**: results are merged by URL, keeping the best relevance score per page.
4. **Rerank & filter**: results are sorted by score, weak matches are dropped, and a compact numbered list with URLs is returned.

The system prompt also teaches the agent to write specific keyword-rich queries, reformulate weak searches, cross-check important claims across sources, and read the most promising pages with `read_webpage`.

Tunable settings (in the notebook): `N_EXPANSIONS`, `MAX_RESULTS_OUT`, `MIN_SCORE`, `SNIPPET_CHARS`, `MAX_PAGE_CHARS`.

## 🔁 Choosing the LLM

Set `LLM_PROVIDER` in the "Alternative LLMs" cell:

| `LLM_PROVIDER` | Default model | Env variable | Get a key |
|---|---|---|---|
| `"groq"` | `openai/gpt-oss-120b` | `GROQ_API_KEY` | [console.groq.com](https://console.groq.com) |
| `"gemini"` | `gemini-3.6-flash` | `GOOGLE_API_KEY` | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) |
| `"huggingface"` | `meta-llama/Llama-3.3-70B-Instruct` | `HUGGINGFACEHUB_API_TOKEN` | [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) |

All three support tool calling, so nothing else changes. If a model is retired, update only its model-name variable (`MODEL_NAME`, `GEMINI_MODEL`, or `HF_MODEL`).

## 🚀 Getting Started

1. Open `research_agent_workshop_updated.ipynb` in Jupyter or Google Colab.
2. Run the install cells.
3. Set your API keys (see below).
4. Choose your LLM with `LLM_PROVIDER`.
5. Run the remaining cells in order, then try the demo queries or enter your own topic.

### 🔑 API Keys

You need a **Tavily** key (web search and page reader) plus the key for your chosen LLM provider.

```python
import os
os.environ["TAVILY_API_KEY"] = "your-tavily-key"
os.environ["GROQ_API_KEY"] = "your-groq-key"   # only if using Groq
```

> ⚠️ **Never commit real API keys.** Use environment variables, Colab Secrets, or `getpass`. If a key was ever shared or pushed, rotate it.

## 💡 Example Queries

- *What are recent advances in Test-Time Compute and Reasoning Models (e.g. DeepSeek-R1, OpenAI o-series)?*
- *Quantum Error Correction breakthroughs in 2025*

## 🛠️ Troubleshooting

- **Agent hits the step limit**: raise `MAX_STEPS` or ask a narrower question.
- **Rate-limit / token errors on free tiers**: lower `MAX_RESULTS_OUT` and `MAX_PAGE_CHARS`, or reduce `N_EXPANSIONS`.
- **Truncated report**: increase `max_tokens` (reasoning tokens count against it).
- **Model not found**: the model was likely retired. Check the provider's deprecation page and change the model-name variable.

## 📦 Main Dependencies

`langchain`, `langchain-groq`, `langchain-google-genai`, `langchain-huggingface`, `langchain-community`, `langchain-tavily`, `langgraph`, `wikipedia`, `arxiv<4`