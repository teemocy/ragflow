# RAGFlow API Reference - Read-Only Retrieval Endpoints

Base URL: `http://localhost:9380/api/v1`
Auth: `Authorization: Bearer <api_key>`

## Datasets

### GET /datasets
List all accessible datasets.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| id | string | - | Filter by dataset ID |
| name | string | - | Filter by dataset name |
| page | int | 1 | Page number |
| page_size | int | 30 | Items per page |
| orderby | string | create_time | Sort field |
| desc | bool | true | Sort descending |

Response: `{"code": 0, "data": [{"id", "name", "description", "document_count", "chunk_count", ...}]}`

### GET /datasets/{dataset_id}
Get a single dataset's full details.

Response: `{"code": 0, "data": {"id", "name", "description", "chunk_method", "embedding_model", "parser_config", ...}}`

## Documents

### GET /datasets/{dataset_id}/documents
List documents in a dataset.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | int | 1 | Page number |
| page_size | int | 30 | Items per page |
| orderby | string | create_time | Sort field |
| desc | bool | true | Sort descending |
| name | string | - | Filter by document name |
| id | string | - | Filter by document ID |
| run | string | - | Filter by processing status |

Response: `{"code": 0, "data": {"docs": [...], "total": int}}`

### GET /datasets/{dataset_id}/documents/{document_id}
Get a single document's details.

Response: `{"code": 0, "data": {"id", "name", "location", "size", "chunk_count", "token_count", "run", ...}}`

## Chunks

### GET /datasets/{dataset_id}/documents/{document_id}/chunks
List chunks of a document.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | int | 1 | Page number |
| page_size | int | 30 | Items per page |
| keywords | string | - | Filter by keywords |
| available | bool | - | Filter by availability |
| id | string | - | Filter by chunk ID |

Response: `{"code": 0, "data": {"chunks": [...], "total": int}}`

### GET /datasets/{dataset_id}/documents/{document_id}/chunks/{chunk_id}
Get a single chunk's content.

Response: `{"code": 0, "data": {"id", "content", "document_id", "dataset_id", ...}}`

## Knowledge Graph

### GET /datasets/{dataset_id}/graph
Get the knowledge graph of a dataset.

Response: `{"code": 0, "data": {"graph": {...}, "mind_map": {...}}}`

### GET /datasets/{dataset_id}/graph/search
Search the knowledge graph.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| query | string | - | Search query (passed as query param) |

Response: `{"code": 0, "data": {"graph": {...}, "mind_map": {...}}}`

## Search & Retrieval

### POST /retrieval
Retrieve chunks across datasets based on a query.

Body: `{"question": str, "dataset_ids": list[str], "document_ids": list[str], "page": int, "page_size": int, "similarity_threshold": float, "vector_similarity_weight": float, "top_k": int, "rerank_id": str, "keyword": bool}`

Response: `{"code": 0, "data": {"chunks": [...], "total": int}}`

### POST /datasets/{dataset_id}/search
Search within a specific dataset.

Body: `{"question": str (required), "doc_ids": list[str], "top_k": int, "page": int, "size": int, "similarity_threshold": float, "vector_similarity_weight": float, "use_kg": bool, "cross_languages": list[str], "keyword": bool, "meta_data_filter": dict}`

Response: `{"code": 0, "data": {"chunks": [...], "total": int, "labels": [...]}}`
