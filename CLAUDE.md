# CLAUDE.md — Air India RAG Chatbot

This file describes the codebase for AI assistants. Follow these conventions when reading, modifying, or extending this project.

---

## Project Overview

A Retrieval-Augmented Generation (RAG) chatbot that answers questions about Air India using a vector database of PDF documents and AWS Bedrock LLMs. Users interact via a Streamlit web UI or by running the CLI directly.

---

## Repository Layout

```
Air-India-RAG-Chatbot/
├── main.py                  # Core RAG logic: embeddings, vector store, LLM inference
├── app.py                   # Streamlit web UI (imports get_response from main.py)
├── test.py                  # Standalone AWS Bedrock streaming demo (not a test suite)
├── requirements.txt         # Pip-installable dependencies
├── pyproject.toml           # Project metadata; canonical dependency list
├── uv.lock                  # Locked dependency versions (managed by uv)
├── .python-version          # Pins Python 3.13
├── AirIndia/                # Source PDF knowledge base (do not delete or rename)
│   ├── Aiesl Employees service regulation.pdf
│   ├── Air India Fact Sheet.pdf
│   ├── Domestic Routes Feb 2025.pdf
│   ├── International Routes Feb 2025.pdf
│   └── List of Major Air India Disasters....pdf
└── chroma_vectorestore/     # Persisted Chroma vector DB (generated; do not commit changes)
    ├── chroma.sqlite3
    └── 5b2c7491-*/          # HNSW index files
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| Language | Python 3.13 |
| Web UI | Streamlit >= 1.55 |
| LLM inference | AWS Bedrock — Amazon Nova Pro (`us.amazon.nova-pro-v1:0`) |
| Embeddings | AWS Bedrock — Amazon Titan Embed Text v2 (`amazon.titan-embed-text-v2:0`) |
| Vector store | Chroma (via `langchain-chroma`) persisted to `./chroma_vectorestore` |
| LLM framework | LangChain + LangChain Community |
| PDF loading | `langchain_community.document_loaders.PyPDFDirectoryLoader` |
| Tokenization | `tiktoken` (cl100k_base encoding) |
| AWS SDK | `boto3` / `botocore` |
| Package manager | `uv` (preferred) or `pip` |

---

## Running the Project

### Prerequisites

- Python 3.13 (see `.python-version`)
- AWS credentials configured with Bedrock access in `us-east-1`
  - Set via `~/.aws/credentials`, environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`), or IAM role
  - Required Bedrock models: `amazon.titan-embed-text-v2:0` and `us.amazon.nova-pro-v1:0`

### Install dependencies

```bash
# Preferred — uses uv with locked versions
uv sync

# Alternative
pip install -r requirements.txt
```

### Run the Streamlit web UI

```bash
streamlit run app.py
```

Opens at `http://localhost:8501`. The UI calls `get_response()` from `main.py`.

### Run the CLI / RAG pipeline directly

```bash
python main.py
```

Executes a hardcoded sample query (`"What is the current status of Air India?"`) and prints the answer. This is useful for quick smoke-testing.

### Run the Bedrock streaming demo

```bash
python test.py
```

Demonstrates `invoke_model_with_response_stream` against Amazon Nova Pro. Independent of the RAG pipeline — useful for verifying AWS credentials and model access.

---

## Key Files In Detail

### `main.py`

Contains all RAG logic. Runs end-to-end on import (instantiates AWS clients, loads vector store, and executes a sample query at module level).

**Important classes and functions:**

```python
class AmazonTitanEmbedding(Embeddings):
    # Wraps AWS Bedrock Titan Embed Text v2:0
    # Truncates input to 8000 tokens (cl100k_base) before embedding
    def embed_query(text: str) -> list[float]
    def embed_documents(texts: list[str]) -> list[list[float]]
    def _safe_truncate(text: str) -> str  # internal token truncation

def get_response(question: str) -> dict:
    # 1. Similarity search: retrieves k=3 most relevant Chroma documents
    # 2. Builds a prompt with the retrieved context + user question
    # 3. Calls Amazon Nova Pro via boto3 invoke_model()
    # Returns the raw Bedrock response dict:
    # result['output']['message']['content'][0]['text']  <- the answer string
```

**Module-level side effects (runs on import):**

- Instantiates `AmazonTitanEmbedding` (creates a `bedrock-runtime` boto3 client)
- Loads the Chroma vector store from `./chroma_vectorestore`
- Creates a second boto3 client for text generation
- Calls `get_response("What is the current status of Air India?")` and `print()`s the result

> **When importing `main` in `app.py` or tests, all of the above executes immediately.** Avoid adding further top-level side effects.

**Commented-out initialisation block (lines 11–17, 61–62):**

These lines load PDFs, split them, embed them, and add them to Chroma. They only need to run **once** to populate the vector store. The existing `chroma_vectorestore/` already contains the embeddings. Do not uncomment unless you are rebuilding the vector store from scratch.

### `app.py`

Thin Streamlit wrapper around `get_response`.

**Known bug — line 9:**
```python
st# Input field   ← broken comment; causes NameError at runtime
```
This should be:
```python
# Input field
```
Fix this before running the Streamlit app.

**Response parsing:**
```python
result['output']['message']['content'][0]['text']
```
This mirrors the Bedrock Nova Pro response schema.

### `test.py`

Standalone AWS Bedrock demo adapted from Amazon's official examples. Uses `invoke_model_with_response_stream` (streaming), whereas `main.py` uses `invoke_model` (non-streaming). Not connected to the RAG pipeline; safe to run independently.

---

## AWS Bedrock Configuration

| Setting | Value |
|---|---|
| Region | `us-east-1` (hardcoded in both `main.py:22` and `main.py:66`) |
| Embedding model | `amazon.titan-embed-text-v2:0` |
| Text generation model | `us.amazon.nova-pro-v1:0` |
| Max embedding tokens | 8000 (truncated via tiktoken cl100k_base) |
| Max generation tokens | 300 (`maxTokens` in `inferenceConfig`) |
| Inference params | `topP=0.1, topK=20, temperature=0` (deterministic) |

To change the region or model IDs, update the hardcoded values in `main.py:21`, `main.py:66-67`. There is no config file abstraction yet.

---

## Vector Store

- **Type**: Chroma with SQLite persistence
- **Location**: `./chroma_vectorestore/`
- **Collection name**: `example_collection`
- **Chunking**: `chunk_size=1000`, `chunk_overlap=200` (RecursiveCharacterTextSplitter)
- **Retrieval**: `similarity_search(question, k=3)` — top 3 chunks returned

**Rebuilding the vector store** (only needed when PDF sources change):
1. Uncomment lines 11–17 and 61–62 in `main.py`
2. Run `python main.py` once
3. Re-comment those lines
4. The `chroma_vectorestore/` directory now contains fresh embeddings

---

## Code Conventions

- **Classes**: PascalCase (`AmazonTitanEmbedding`)
- **Functions / variables**: snake_case (`get_response`, `embed_query`)
- **Constants**: UPPER_SNAKE_CASE (`MODEL_ID`, `max_tokens`)
- **Private helpers**: leading underscore (`_safe_truncate`)
- **Import order**: standard library → third-party → local (not enforced by tooling currently)

---

## Known Issues

1. **`app.py` line 9 syntax error** — `st# Input field` should be `# Input field`. The app crashes on launch until this is fixed.
2. **Module-level query execution in `main.py`** — importing `main` triggers a live AWS Bedrock call. Wrap the bottom of `main.py` in `if __name__ == "__main__":` to isolate CLI behaviour.
3. **Hardcoded AWS region and model IDs** — no environment variable or config file support. Changing the region requires editing source.
4. **Magic numbers** — `k=3`, `maxTokens=300`, `chunk_size=1000`, `chunk_overlap=200` are inline with no named constants or config.
5. **No formal test suite** — `test.py` is a Bedrock demo, not a pytest suite. `pytest tests/` (as documented in README) will fail because `tests/` does not exist.
6. **README inaccuracies** — README describes a `src/` directory layout, `config.py`, and `OPENAI_API_KEY` that do not exist. The actual stack uses AWS Bedrock, not OpenAI.

---

## Development Workflow

### Adding or updating source PDFs

1. Place new PDFs in `AirIndia/`
2. Rebuild the vector store (see above)
3. Commit both the PDFs and the regenerated `chroma_vectorestore/`

### Changing the LLM or embedding model

1. Update `model_id` in `AmazonTitanEmbedding.__init__` (embedding) or `MODEL_ID` (generation) in `main.py`
2. Verify the model is available in your Bedrock region
3. If the embedding dimension changes, delete `chroma_vectorestore/` and rebuild

### Adding a new question / endpoint

Extend `get_response()` in `main.py`. The Streamlit UI will pick it up automatically since `app.py` imports and calls it directly.

### Dependency management

Prefer `uv` for reproducible installs:
```bash
uv add <package>      # adds to pyproject.toml and uv.lock
uv sync               # installs from lock file
```

Do not edit `requirements.txt` by hand if using `uv`; keep it in sync with `pyproject.toml`.

---

## Git Branches

| Branch | Purpose |
|---|---|
| `main` | Stable / production |
| `claude/add-claude-documentation-4gaLU` | Current development branch |

Default development branch for AI-assisted work: `claude/add-claude-documentation-4gaLU`.
