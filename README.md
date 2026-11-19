# AI Learning Service

`ai-learning-service` is a FastAPI-based backend service powering an AI learning and community platform[cite: 1, 3]. It combines local/remote LLM inference (**Qwen3 4B** via **Ollama**), document extraction (**Docling**), semantic retrieval (**Qdrant**), and strict schema validation (**Pydantic**) to deliver fast, grounded learning workflows without over-engineering.

Not every feature uses full RAG: semantic retrieval is reserved for features that genuinely need grounded lookup or similarity search (such as **AI Tutor** and **Duplicate Questions**), while structured features operate directly on selected lecture content, user performance data, or classification rules.

---

## Tech Stack

* **Language:** Python
* **API Framework:** FastAPI
* **Model Runtime:** Ollama
* **LLM:** Qwen3 4B (quantized)
* **Document Extraction:** Docling
* **Embeddings:** Multilingual embeddings (`multilingual-e5-small` / Qwen embeddings)
* **Vector Database:** Qdrant
* **Application Database:** PostgreSQL (owned primarily by the main backend/service layer, passing structured inputs as needed)[cite: 3]
* **Validation & Schemas:** Pydantic[cite: 3]
* **Background Jobs:** FastAPI background tasks initially; Redis + RQ when needed[cite: 3]
* **Agent Orchestration:** Direct workflow modules initially; LangGraph later if needed

---

## Supported Features

1. **Lecture Summary:** `Extract -> Summarize -> Validate`
2. **Mind Map:** `Extract -> Analyze -> Build Graph`
3. **AI Tutor:** `Retrieve -> Generate -> Cite`
4. **Quiz Generator:** `Retrieve/Select Content -> Generate Questions`
5. **Flashcards:** `Extract Concepts -> Generate Cards`
6. **Performance Analysis:** `Analyze Quiz Results -> Identify Weak Topics`
7. **Personalized Plan:** `Analyze Progress -> Generate Plan`
8. **Community Moderation:** `Classify -> Flag -> Human Review`
9. **Duplicate Questions:** `Embed -> Similarity Search`[cite: 1, 2]

---
## Local Setup & Running the Project

Follow these steps to set up and run `ai-learning-service` locally on your machine:

### Prerequisites
* **Python 3.10+** installed
* **Ollama** installed and running locally ([ollama.com](https://ollama.com))

### Step 1: Clone & Create a Virtual Environment
```bash
git clone <your-repo-url>
cd ai-learning-service
python -m venv .venv
```
Activate the virtual environment:
* **Windows (PowerShell):**
  ```powershell
  .venv\Scripts\Activate.ps1
  ```
* **Linux / macOS:**
  ```bash
  source .venv/bin/activate
  ```

### Step 2: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 3: Pull Required Ollama Models
Download the chat model and the embedding model before starting the API:
```bash
ollama pull qwen3:4b
ollama pull qwen3-embedding:0.6b
```

### Step 4: Configure Environment Variables
Copy `.env.example` to `.env` and adjust values if needed:
* **Windows (PowerShell):**
  ```powershell
  Copy-Item .env.example .env
  ```
* **Linux / macOS:**
  ```bash
  cp .env.example .env
  ```

### Step 5: Run the FastAPI Server
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```
Once running, open the interactive API docs at `http://localhost:8000/docs`.

---


## Repository Structure

```text
ai-learning-service/
├── app/
│   ├── main.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── documents.py
│   │   ├── tutor.py
│   │   ├── summaries.py
│   │   ├── mindmaps.py
│   │   ├── quizzes.py
│   │   ├── flashcards.py
│   │   ├── analytics.py
│   │   └── community.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── model_manager.py
│   │   ├── prompt_manager.py
│   │   └── exceptions.py
│   ├── document_processing/
│   │   ├── __init__.py
│   │   ├── parser.py
│   │   ├── chunker.py
│   │   └── metadata.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── llm.py
│   │   └── embeddings.py
│   ├── rag/
│   │   ├── __init__.py
│   │   ├── vector_store.py
│   │   ├── retriever.py
│   │   └── context_builder.py
│   ├── workflows/
│   │   ├── __init__.py
│   │   ├── ingestion.py
│   │   ├── tutor.py
│   │   ├── summary.py
│   │   ├── mindmap.py
│   │   ├── quiz.py
│   │   ├── flashcards.py
│   │   ├── performance.py
│   │   ├── learning_plan.py
│   │   ├── moderation.py
│   │   └── duplicate_questions.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── common.py
│   │   ├── document.py
│   │   ├── tutor.py
│   │   ├── summary.py
│   │   ├── mindmap.py
│   │   ├── quiz.py
│   │   ├── flashcards.py
│   │   ├── analytics.py
│   │   └── community.py
│   └── evaluation/
│       ├── datasets/
│       ├── metrics.py
│       └── benchmarks.py
├── tests/
│   ├── test_rag.py
│   ├── test_workflows.py
│   └── test_api.py
├── .env.example
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── README.md
```

### Root & Testing Files
* `requirements.txt` — Python package dependencies for the API, model runtime client, document parser, vector store, and validation.
* `.env.example` — Template of required environment variables (Ollama URL, model names, Qdrant path/URL, chunking/retrieval settings).
* `Dockerfile` — Container definition for building and deploying `ai-learning-service`.
* `docker-compose.yml` — Multi-service local/production setup orchestrating `ai-learning-service`, Ollama, and Qdrant (as needed).
* `README.md` — Project overview, architecture guide, feature mapping, and repository reference.
* `tests/` — Automated unit and integration tests (`test_rag.py`, `test_workflows.py`, `test_api.py`).

### Entrypoint (`app/main.py`)
* `app/main.py` — Application entrypoint. Initializes the FastAPI instance, manages startup/shutdown lifecycles (models, vector store connections), registers API routers, and attaches exception handlers.

### `app/api/` — HTTP Layer
Thin route handlers responsible for receiving HTTP requests, validating payloads via `app/schemas/`, invoking `app/workflows/`, and returning responses (including streaming responses where applicable).

* `app/api/__init__.py` — Package initializer for API routers.
* `app/api/documents.py` — Endpoints for uploading, listing/inspecting, and deleting study materials.
* `app/api/tutor.py` — Conversational endpoint for the **AI Tutor** feature, supporting grounded Q&A and streaming output.
* `app/api/summaries.py` — Endpoint for generating **Lecture Summary** outputs.
* `app/api/mindmaps.py` — Endpoint for generating **Mind Map** graph structures.
* `app/api/quizzes.py` — Endpoints for **Quiz Generator** (and quiz evaluation/grading workflows if exposed here).
* `app/api/flashcards.py` — Endpoint for generating **Flashcards** from study material.
* `app/api/analytics.py` — Endpoints for **Performance Analysis** and **Personalized Plan** generation.
* `app/api/community.py` — Endpoints for **Community Moderation** and **Duplicate Questions** detection.

### `app/core/` — Core Configuration & Shared Services
Cross-cutting infrastructure used across the application.

* `app/core/__init__.py` — Package initializer for core utilities.
* `app/core/config.py` — Centralized configuration loaded from environment variables using Pydantic `BaseSettings`.
* `app/core/model_manager.py` — Manages initialization, caching, and access to LLM/embedding clients and shared infrastructure instances.
* `app/core/prompt_manager.py` — Stores and formats system/task prompts, grounding/anti-hallucination instructions, and output formatting rules.
* `app/core/exceptions.py` — Custom domain exceptions (e.g., parsing errors, empty retrieval, model errors) mapped to consistent HTTP responses.

### `app/document_processing/` — Document Ingestion & Preparation
Transforms raw uploaded files into clean, structured, metadata-enriched chunks ready for storage or direct feature generation.

* `app/document_processing/__init__.py` — Package initializer for document processing modules.
* `app/document_processing/parser.py` — Extracts text, sections, and tables from documents using **Docling**.
* `app/document_processing/chunker.py` — Splits parsed content into manageable chunks with controlled size and overlap.
* `app/document_processing/metadata.py` — Builds and normalizes metadata (document ID, course/lecture identifiers, page/section info, hashes) for filtering and citations.

### `app/models/` — Model Client Wrappers
Isolates direct communication with AI models so switching models or endpoints does not break workflows.

* `app/models/__init__.py` — Package initializer for model wrappers.
* `app/models/llm.py` — Wrapper around **Qwen3 4B** via **Ollama**, supporting standard generation, streaming, thinking mode control, and structured JSON output.
* `app/models/embeddings.py` — Wrapper for generating text embeddings used in semantic search and duplicate question matching.

### `app/rag/` — Retrieval & Context Assembly
Handles vector persistence, semantic search, and prompt context construction.

* `app/rag/__init__.py` — Package initializer for RAG components.
* `app/rag/vector_store.py` — **Qdrant** client wrapper managing collections (e.g., course materials and community questions), upserts, deletes, and similarity queries.
* `app/rag/retriever.py` — Queries `vector_store.py` using embedded queries and metadata filters to fetch relevant chunks or similar questions.
* `app/rag/context_builder.py` — Formats retrieved chunks with source references/citations and enforces context budget limits before passing prompts to the LLM.

### `app/workflows/` — Feature & Pipeline Orchestration
Contains the business logic for each capability, keeping `app/api/` thin and reusable across HTTP routes, background tasks, tests, and evaluation scripts.

* `app/workflows/__init__.py` — Package initializer for workflows.
* `app/workflows/ingestion.py` — End-to-end document pipeline: `Parse -> Chunk -> Embed -> Store in Qdrant`.
* `app/workflows/tutor.py` — **AI Tutor** workflow: `Retrieve -> Generate -> Cite`.
* `app/workflows/summary.py` — **Lecture Summary** workflow: `Extract -> Summarize -> Validate`.
* `app/workflows/mindmap.py` — **Mind Map** workflow: `Extract -> Analyze -> Build Graph`.
* `app/workflows/quiz.py` — **Quiz Generator** workflow: `Retrieve/Select Content -> Generate Questions`.
* `app/workflows/flashcards.py` — **Flashcards** workflow: `Extract Concepts -> Generate Cards`.
* `app/workflows/performance.py` — **Performance Analysis** workflow: `Analyze Quiz Results -> Identify Weak Topics`.
* `app/workflows/learning_plan.py` — **Personalized Plan** workflow: `Analyze Progress -> Generate Plan`.
* `app/workflows/moderation.py` — **Community Moderation** workflow: `Classify -> Flag -> Human Review`.
* `app/workflows/duplicate_questions.py` — **Duplicate Questions** workflow: `Embed -> Similarity Search`.

### `app/schemas/` — Pydantic Data Contracts
Defines strict request, response, and structured LLM output schemas to keep API contracts and JSON payloads predictable.

* `app/schemas/__init__.py` — Package initializer for schemas.
* `app/schemas/common.py` — Shared response wrappers, status models, and source citation schemas.
* `app/schemas/document.py` — Request/response models for document upload, processing status, and deletion.
* `app/schemas/tutor.py` — Request/response/message models for AI Tutor conversations.
* `app/schemas/summary.py` — Structured schema for lecture summaries.
* `app/schemas/mindmap.py` — Structured schema for mind map nodes and edges/relationships.
* `app/schemas/quiz.py` — Structured schema for generated quiz questions, choices, answers, and explanations.
* `app/schemas/flashcards.py` — Structured schema for study flashcards.
* `app/schemas/analytics.py` — Schemas for performance analysis reports and personalized study plans.
* `app/schemas/community.py` — Schemas for content moderation verdicts and similar/duplicate questions.

### `app/evaluation/` — Quality & Performance Benchmarking
Tools and datasets to test accuracy, grounding, JSON reliability, and latency before shipping changes.

* `app/evaluation/datasets/` — Test datasets (sample lecture Q&A pairs, edge cases, out-of-scope questions, community samples).
* `app/evaluation/metrics.py` — Evaluation metrics (retrieval relevance, grounding/faithfulness, schema validity, classification/similarity checks).
* `app/evaluation/benchmarks.py` — Scripts to run batch evaluations and measure latency/throughput across workflows.

---

## Data Flow Conventions

* **Grounded Q&A (`tutor.py`):** User query + optional filters -> `retriever.py` -> `context_builder.py` -> `prompt_manager.py` -> `llm.py` (`Qwen3 4B`) -> streamed/structured answer with citations.
* **Content-Driven Generation (`summary.py`, `mindmap.py`, `flashcards.py`, `quiz.py`):** Target lecture/topic content (selected directly or retrieved) -> `prompt_manager.py` -> `llm.py` with Pydantic schema (`app/schemas/`) -> validated JSON output.
* **Duplicate Question Detection (`duplicate_questions.py`):** New question -> `embeddings.py` -> similarity search in `Qdrant` -> return top matches above threshold.
* **Performance, Planning & Moderation (`performance.py`, `learning_plan.py`, `moderation.py`):** Structured metrics/progress or post/comment text -> `prompt_manager.py` -> `llm.py` with Pydantic schema -> validated result.