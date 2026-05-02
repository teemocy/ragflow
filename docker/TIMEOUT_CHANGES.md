# Model Call Timeout Changes

All LLM chat and embedding call timeouts have been increased for slow local models (e.g., LM-Studio on CPU).

## Timeout Values

| Category | Timeout | Reason |
|----------|---------|--------|
| LLM chat calls | **1440 min** (24h) | Local models may be extremely slow |
| Embedding calls | **30 min** | Embedding is faster but still needs headroom |
| Agent component exec | **1440 min** (24h) | Agent LLM calls inherit chat timeouts |

## Files Modified

### LLM Chat (1440 min)

| File | Change |
|------|--------|
| `rag/raptor.py` | `_chat()` and `summarize()`: 20min -> 1440min |
| `rag/graphrag/general/extractor.py` | entity/relation extraction: 20min -> 1440min |
| `rag/graphrag/general/community_reports_extractor.py` | community reports: 3min -> 1440min |
| `rag/graphrag/entity_resolution.py` | entity resolution: 5min -> 1440min |
| `rag/graphrag/general/index.py` | `resolve_entities()`, `extract_community()`: 30min -> 1440min |
| `rag/svr/task_executor.py` | `build_chunks()`: 80min -> 1440min |
| `rag/svr/task_executor.py` | `run_raptor_for_kb()`: 60min -> 1440min |
| `rag/svr/task_executor.py` | `do_handle_task()`: 180min -> 1440min |
| `rag/flow/base.py` | agent component base `_invoke()`: 10min -> 1440min |
| `agent/component/*.py` | all agent components: 10-20min -> 1440min |
| `api/utils/api_utils.py` | model strength check chat: 30s -> 30min |

### Embedding (30 min)

| File | Change |
|------|--------|
| `rag/raptor.py` | `_embedding_encode()`: 20s -> 30min |
| `rag/graphrag/utils.py` | `get_relation()`: 3s -> 30min |
| `rag/graphrag/utils.py` | `graph_node_to_chunk()` embedding: 3s -> 30min |
| `rag/graphrag/utils.py` | `graph_edge_to_chunk()` embedding: 3s -> 30min |
| `rag/graphrag/utils.py` | `set_graph()` doc store write: 3s -> 30min |
| `rag/graphrag/general/index.py` | `merge_subgraph()`: 3min -> 30min |
| `rag/svr/task_executor.py` | `batch_encode()` (x3): 60s -> 30min |
| `rag/flow/tokenizer/tokenizer.py` | `batch_encode()`: 60s -> 30min |
| `rag/llm/embedding_model.py` | multimodal embedding HTTP request: 60s -> 30min |
| `deepdoc/parser/figure_parser.py` | image-to-text model calls (x2): 30s -> 30min |
| `api/apps/llm_app.py` | LLM_TIMEOUT_SECONDS default: 10s -> 30min |
| `api/utils/api_utils.py` | model strength check embedding: 10s -> 30min |

### Unchanged (not model-related)

| File | Timeout | Purpose |
|------|---------|---------|
| `rag/svr/task_executor.py` `upload_to_minio()` | 60s | MinIO upload, not model |
| `agent/component/invoke.py` | 3s | Quick dispatch, not model |
| `agent/component/switch.py` | 3s | Conditional routing, not model |
| `agent/component/variable_aggregator.py` | 3s | Data aggregation, not model |
| Various Redis/DB locks | 10-60s | Infrastructure, not model |

## Docker Volume Mounts

`docker/docker-compose.yml` mounts the modified source files into the container so changes take effect without rebuilding the image:

```yaml
volumes:
  - ../rag/graphrag:/ragflow/rag/graphrag
  - ../rag/raptor.py:/ragflow/rag/raptor.py
  - ../rag/svr/task_executor.py:/ragflow/rag/svr/task_executor.py
  - ../rag/flow:/ragflow/rag/flow
  - ../rag/llm/embedding_model.py:/ragflow/rag/llm/embedding_model.py
  - ../agent/component:/ragflow/agent/component
  - ../deepdoc/parser/figure_parser.py:/ragflow/deepdoc/parser/figure_parser.py
  - ../api/apps/llm_app.py:/ragflow/api/apps/llm_app.py
  - ../api/utils/api_utils.py:/ragflow/api/utils/api_utils.py
```

## Deploy to New Machine

```bash
git clone git@github.com:teemocy/ragflow.git && cd ragflow/docker
# edit .env as needed
docker compose up -d
```

## Update Existing Deployment

```bash
cd ragflow && git pull origin main
cd docker && docker compose down && docker compose up -d
```
