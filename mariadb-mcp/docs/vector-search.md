## Vector & Semantic Search (Optional)

When an embedding provider is configured, additional tools are available for building RAG pipelines directly through the MCP interface:

| Tool | What it does |
|---|---|
| `create_vector_store` | Creates a vector store table |
| `insert_docs_vector_store` | Embeds and stores documents |
| `search_vector_store` | Semantic similarity search |
| `list_vector_stores` | Lists available vector stores |
| `delete_vector_store` | Removes a vector store |

**Configure an embedding provider in `.env`:**
```
EMBEDDING_PROVIDER=openai   # or gemini or huggingface
OPENAI_API_KEY=sk-...
```

For building pure vector search applications, using MariaDB's native `VECTOR` type and `VEC_DISTANCE_*` functions directly is more efficient. The MCP vector tools are useful for quick prototyping and when AI agents need to manage the vector store themselves. See the `mariadb-vector` skill for native vector SQL.
