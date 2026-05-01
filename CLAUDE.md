# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RAGFlow is an open-source RAG (Retrieval-Augmented Generation) engine based on deep document understanding. Full-stack application with Python backend (Quart/async Flask), React/TypeScript frontend (Vite), and Docker-based microservice deployment. Data stores: MySQL, Elasticsearch/Infinity/OpenSearch, Redis, MinIO.

## Common Commands

### Backend

```bash
# Setup
uv sync --python 3.12 --all-extras
uv run python3 download_deps.py
pre-commit install

# Start infrastructure services
docker compose -f docker/docker-compose-base.yml up -d

# Run backend (requires services running, loads docker/.env automatically)
export PYTHONPATH=$(pwd)
bash docker/launch_backend_service.sh

# Run all backend tests
uv run pytest

# Run specific test file
uv run pytest test/testcases/test_http_api/test_kb_app/test_kb_create.py

# Run tests by priority level (p1=smoke, p2=core, p3=full; default p2)
pytest -s --tb=short --level=p1 test/testcases/test_http_api/

# Lint
ruff check          # check
ruff format         # format
```

### Frontend

```bash
cd web
npm install
npm run dev          # Dev server with HMR
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Jest with coverage
npm run type-check   # TypeScript validation (tsc --noEmit)
```

### Docker

```bash
cd docker
docker compose -f docker-compose.yml up -d          # Full stack
docker compose -f docker-compose-base.yml up -d      # Infrastructure only
docker build --platform linux/amd64 -f Dockerfile -t infiniflow/ragflow:nightly .
```

## Architecture

### Backend (`api/`)

- **Framework**: Quart (async Flask), not standard Flask. All route handlers are `async def`.
- **Entry point**: `api/ragflow_server.py` — initializes DB, loads plugins, starts background threads
- **Blueprint auto-discovery** (`api/apps/__init__.py`): Files matching `*_app.py` are auto-loaded as Quart blueprints. URL routing:
  - `api/apps/*_app.py` → `/<API_VERSION>/<page_name>` (e.g. `/v1/dialog`)
  - `api/apps/sdk/*.py` and `api/apps/restful_apis/*.py` → `/api/<API_VERSION>/...` (e.g. `/api/v1/datasets`)
  - API_VERSION is `"v1"` defined in `api/constants.py`
- **Auth**: JWT via `Authorization: bearer <token>` header, or API token. `@login_required` decorator in `api/apps/__init__.py`. User loaded into `g.user` via `current_user` LocalProxy.
- **ORM**: Peewee (`api/db/db_models.py` for models, `api/db/services/` for business logic)
- **Background processing**: `rag/svr/task_executor.py` — launched as separate processes by `docker/launch_backend_service.sh`. Controlled by `WS` env var (worker count, default 1).

### Core RAG Pipeline

Data flow: **Upload → Parse → Chunk → Embed → Store → Retrieve → Generate**

- **Document parsing** (`deepdoc/`): PDF, DOCX, PPTX with OCR (PaddleOCR), layout analysis, table/figure extraction
- **Chunking & tokenization** (`rag/flow/`): Multiple parsing strategies, configurable chunk sizes
- **Embedding & vector store** (`rag/utils/`): Backends for Elasticsearch, Infinity, OpenSearch with connection pooling
- **LLM integration** (`rag/llm/`): Abstractions for chat, embedding, reranking. Supports 20+ providers via litellm and direct SDKs
- **Graph RAG** (`rag/graphrag/`): Knowledge graph construction and querying

### Agent System (`agent/`)

Component-based workflow system using `ComponentBase` with a canvas execution model. Components: LLM, retrieval, categorize, tool calling. External tools in `agent/tools/` (Tavily, Wikipedia, SQL, etc.). Sandboxed execution in `agent/sandbox/`.

### Frontend (`web/`)

See `web/CLAUDE.md` for detailed frontend conventions. Key points: React 18 + Vite, Zustand for state, TanStack Query for data fetching, shadcn/ui components, Tailwind CSS, react-i18next for i18n.

## Testing

### Structure

- `test/testcases/test_http_api/` — HTTP integration tests (require running server)
- `test/testcases/test_sdk_api/` — Python SDK integration tests (require running server)
- `test/testcases/test_web_api/` — Web API unit tests (mocked, can run standalone)
- `test/testcases/test_common_data_source/` — Data source unit tests
- `test/playwright/` — E2E tests
- `sdk/python/test/` — SDK unit tests

### Test Markers and Levels

Markers: `p0` (critical), `p1` (high), `p2` (medium), `p3` (low). The `--level` flag in `conftest.py` expands to marker expressions:
- `p1` → only p1 tests
- `p2` (default) → p1 + p2
- `p3` → p1 + p2 + p3

Run with: `pytest -s --tb=short --level=p2 test/testcases/`

Integration tests require Docker services and set `DOC_ENGINE` env var (elasticsearch or infinity).

## Key Configuration

- `docker/.env` — All environment variables (ports, engine selection, credentials)
- `docker/service_conf.yaml.template` — Backend service config template
- `pyproject.toml` — Python deps (requires `>=3.12,<3.15`), ruff config (line-length 200, ignore E402), pytest config
- `web/package.json` — Frontend deps and scripts

### DOC_ENGINE

Switch between Elasticsearch (default) and Infinity: set `DOC_ENGINE=infinity` in `docker/.env`, then restart: `docker compose down -v && docker compose up -d`.

## Development Conventions

1. Think before acting. Read existing files before writing code.
2. Be concise in output but thorough in reasoning.
3. Prefer editing over rewriting whole files.
4. Do not re-read files you have already read.
5. Test your code before declaring done.
6. No sycophantic openers or closing fluff.
7. Keep solutions simple and direct.
8. User instructions always override this file.
