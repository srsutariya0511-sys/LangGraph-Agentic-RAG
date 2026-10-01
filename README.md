# 🤖 Agentic RAG System with LangGraph

A **production-ready Retrieval-Augmented Generation (RAG) system** that intelligently corrects retrieval errors using LangGraph for orchestration and FAISS for vector storage. This system demonstrates advanced multi-step reasoning, automatic error recovery, and document grading with structured outputs.

## ✨ Key Features

- **🔄 Self-Correcting Retrieval** — Automatically detects and fixes poor retrieval quality using LLM-based document grading
- **🔍 Document Relevance Scoring** — Strict LLM-based evaluation before generation to ensure high-quality answers
- **🔀 Query Rewriting** — Transforms unclear queries into search-optimized formats for better results
- **📊 Transparent State Machine** — Clear routing decisions and explicit control flow for easy debugging
- **🔌 Modular Architecture** — Swap components easily (FAISS → Pinecone, OpenAI → Anthropic, etc.)
- **🌐 Web-Based Document Ingestion** — Automatically scrapes and indexes web pages into the vector store
- **⚙️ Structured Output Validation** — Uses Pydantic models for reliable decision-making

---

## 🏗️ Architecture

The system uses LangGraph to orchestrate a sophisticated multi-node workflow with intelligent routing:

```
┌─────────────────────────────────────────────────────┐
│ Agent Node (with Tools)                             │
│ → Decides whether to retrieve or answer directly    │
└────────────────┬────────────────────────────────────┘
                 │
        ┌────────▼─────────┐
        │ Has tool calls?  │
        └────────┬────┬────┘
                 │    │
              YES│    │NO (END)
                 │    │
         ┌───────▼─┐  │
         │ Retrieve│  │
         └────┬────┘  │
              │       │
         ┌────▼──────────────┐
         │ Grade Documents   │
         │ (Pydantic Output) │
         └────┬──────┬───────┘
              │      │
           YES│      │NO
              │      │
        ┌─────▼──┐  ┌──────────┐
        │Generate│  │Rewrite   │
        └─────┬──┘  └────┬─────┘
              │          │
             END         │
                    (Loop to Agent)
```

### Core Components

| Component | Purpose | Key Files |
|-----------|---------|-----------|
| **Configuration Layer** | Manages environment variables and LLM clients | `config/settings.py`, `config/openai.py` |
| **Retrieval Module** | Web scraping, document chunking, FAISS vectorization | `retriever.py` |
| **Agent Nodes** | Decision logic for routing (agent, rewrite, generate) | `agents/nodes.py` |
| **Routing Edges** | Conditional logic with document grading | `agents/edges.py` |
| **Graph Orchestration** | LangGraph state machine assembly | `agents/graph.py` |
| **Entry Point** | Sample execution with hardcoded questions | `main.py` |

---

## 📁 Project Structure

```
LangGraph-Agentic-RAG/
├── src/
│   ├── config/
│   │   ├── __init__.py            # Exports LLM and embeddings factory functions
│   │   ├── settings.py            # Load .env and validate OPENAI_API_KEY
│   │   └── openai.py              # ChatOpenAI and embedding clients
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── nodes.py               # Agent, rewrite, generate node functions
│   │   ├── edges.py               # Document grading and routing logic
│   │   └── graph.py               # LangGraph StateGraph assembly
│   ├── __init__.py
│   ├── main.py                    # Entry point with sample questions
│   └── retriever.py               # Document loading, chunking, vectorization
├── pyproject.toml                 # Python package config (uv compatible)
├── run.sh                         # Shell wrapper for execution
├── .env                           # (Not committed) OpenAI API key
├── README.md                      # This file
├── architecture.png               # Detailed flow diagram
├── architecture_simple.png        # Simplified state machine diagram
└── .venv/                         # (Optional) Virtual environment

```

---

## 🚀 Quick Start

### Prerequisites

- **Python 3.10+**
- **OpenAI API Key** (with access to gpt-4o-mini or similar)

### 1. Clone & Setup Virtual Environment

```bash
git clone https://github.com/Panth19/LangGraph-Agentic-RAG.git
cd LangGraph-Agentic-RAG

# Install uv (fast Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create and activate virtual environment
uv venv
source .venv/bin/activate  # macOS/Linux
# or
.venv\Scripts\activate     # Windows
```

### 2. Install Dependencies

```bash
uv pip install -e .
```

### 3. Configure Environment

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=sk-your-api-key-here
```

### 4. Run the System

```bash
python -m src.main
```

Or use the shell wrapper:

```bash
bash run.sh
```

---

## 📖 How It Works

### Workflow Execution

When you run the system with a question like *"What are the key components of an AI agent?"*:

1. **Agent Node** receives the question and decides whether to retrieve documents
2. **Retrieve** node executes the `retrieve_documents` tool to fetch relevant documents from FAISS
3. **Grade Documents** node uses structured LLM output to evaluate relevance:
   - ✅ **Relevant** → Proceed to Generate
   - ❌ **Not Relevant** → Rewrite the query and loop back to Agent
4. **Generate** node produces a final answer based on graded documents
5. **Output** streams through the event loop for real-time visibility

### Sample Execution Log

```
============================================================
❓ Question: What are the key components of an AI agent?
============================================================

🔄 Processing...

📍 Node: agent
🔧 Tool Call: retrieve_documents

📍 Node: retrieve
💬 Output: Retrieved 4 documents about AI agents...

📍 Node: grade_documents (internal)
🔍 Grading: yes
💭 Reasoning: Documents contain specific information about agent components

📍 Node: generate
💬 Output: Based on the documents...

============================================================
✨ FINAL ANSWER:
============================================================
The key components of an AI agent include:
1. Planning and reasoning capabilities
2. Memory systems for context retention
3. Tool use and execution capabilities
4. Reflection and self-improvement mechanisms
============================================================
```

---

## ⚙️ Configuration & Customization

### Change the LLM Provider

Edit `src/config/openai.py`:

```python
# Switch to Anthropic
from langchain_anthropic import ChatAnthropic

def get_llm(model_name: str = "claude-3-sonnet-20240229", temperature: float = 0):
    return ChatAnthropic(model=model_name, temperature=temperature)
```

### Replace FAISS with Another Vector Store

Edit `src/retriever.py`:

```python
# Example: Use Pinecone instead
from langchain_pinecone import PineconeVectorStore

def ingest_documents(urls: list[str]):
    # ... document loading and splitting ...
    
    vectorstore = PineconeVectorStore.from_documents(
        documents=splits,
        embedding=get_embeddings(),
        index_name="agentic-rag"
    )
    return vectorstore
```

### Adjust Chunk Size & Overlap

Edit `src/retriever.py`:

```python
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,    # Smaller chunks for more specific retrieval
    chunk_overlap=100  # Overlap prevents cutting mid-sentence
)
```

### Modify Model Temperature

Edit `src/config/openai.py`:

```python
def get_llm(model_name: str = "gpt-4o-mini", temperature: float = 0.3):
    # Higher temperature (0.7-1.0) = more creative, less consistent
    # Lower temperature (0.0-0.3) = more deterministic, focused
    ...
```

### Custom Document Sources

Edit `src/main.py`:

```python
# Add different URLs
urls = [
    "https://your-documentation.com/page1",
    "https://your-knowledge-base.com/page2"
]

# Or load from local files instead
from langchain_community.document_loaders import TextLoader
loader = TextLoader("documents/knowledge.txt")
docs = loader.load()
```

---

## 🧪 Testing & Validation

### Verify API Key

```bash
echo $OPENAI_API_KEY
```

### Reinstall Dependencies (if needed)

```bash
uv pip install -e . --force-reinstall
```

### Check Import Paths

```bash
python -c "from src.config import get_llm, get_embeddings; print('✅ Imports OK')"
```

### Debug LangGraph Execution

Add verbose logging to `main.py`:

```python
for output in app.stream(inputs):
    print(f"DEBUG: {output}")  # See raw node outputs
```

---

## 📚 Dependencies

| Package | Purpose |
|---------|---------|
| `langchain` | LLM framework & abstractions |
| `langchain-openai` | OpenAI integration (ChatOpenAI, Embeddings) |
| `langchain-community` | Community loaders & vectorstores (WebBaseLoader, FAISS) |
| `langgraph` | State machine orchestration & graph compilation |
| `faiss-cpu` | Local vector store for similarity search |
| `beautifulsoup4` | HTML parsing for web document loading |
| `python-dotenv` | Environment variable management |
| `tiktoken` | Token counting for OpenAI models |

---

## 🎯 Key Advantages Over Simple RAG

| Aspect | Simple RAG | This System |
|--------|-----------|------------|
| **Error Detection** | Retrieves once, hopes for best ❌ | Grades documents, retries if needed ✅ |
| **Query Refinement** | Static queries | Rewrites unclear queries dynamically ✅ |
| **Transparency** | Black box | Full state machine visibility ✅ |
| **Flexibility** | Tightly coupled | Swappable components ✅ |
| **Production-Ready** | Edge cases fail silently | Handles failures gracefully ✅ |

---

## 🔗 References & Resources

- **LangGraph Documentation**: https://python.langchain.com/docs/langgraph/
- **LangChain Documentation**: https://python.langchain.com/
- **Inspiration**: Based on the article on building agentic RAG systems with self-correcting retrieval
- **Vector Stores**: FAISS, Pinecone, Weaviate, Milvus, Qdrant

---

## 🤝 Contributing

Contributions are welcome! Feel free to:

- Submit issues for bugs or feature requests
- Open pull requests with improvements
- Add support for new vector stores or LLM providers
- Improve documentation and examples

---

## 📄 License

This project is open source. Feel free to use and modify as needed.

---

## 🆘 Troubleshooting

### Error: `OPENAI_API_KEY not found`

**Solution**: Ensure `.env` file exists in project root with valid key:
```bash
echo "OPENAI_API_KEY=sk-..." > .env
```

### Error: `ModuleNotFoundError`

**Solution**: Reinstall in development mode:
```bash
uv pip install -e . --force-reinstall
```

### Error: `WebBaseLoader fails`

**Solution**: Check internet connection and URL accessibility:
```python
from langchain_community.document_loaders import WebBaseLoader
loader = WebBaseLoader(["https://example.com"])
docs = loader.load()  # Test loading
```

### LLM Response is truncated

**Solution**: Adjust chunk size or use a longer context window model in `config/openai.py`

---


