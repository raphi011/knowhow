# memcp

MCP server that gives AI agents persistent memory using SurrealDB as a knowledge graph backend.

## Architecture

```mermaid
graph LR
    Agent[AI Agent] <-->|MCP Protocol| Server[memcp]
    Server <--> DB[(SurrealDB)]
    Server <--> Embed[Embedding Model]
    Server <--> NLI[NLI Model]

    subgraph Web Dashboard
        API[GraphQL API] --> Server
        UI[Next.js Frontend] --> API
    end
```

## How It Works

```mermaid
graph TB
    subgraph Storage
        E1[Entity] -->|relation| E2[Entity]
        E2 -->|relation| E3[Entity]
        E1 -.->|embedding| V1[384-dim vector]
        E2 -.->|embedding| V2[384-dim vector]
    end

    subgraph Search
        Q[Query] --> Hybrid{Hybrid Search}
        Hybrid --> BM25[BM25 Keyword]
        Hybrid --> Vec[Vector Similarity]
        BM25 --> Results
        Vec --> Results
    end
```

Entities store content with vector embeddings for semantic search. Relations create a traversable graph. Search combines BM25 keyword matching with cosine similarity for hybrid retrieval.

## Features

- **Semantic Memory**: Store entities with vector embeddings for semantic search
- **Episodic Memory**: Complete conversation sessions with timestamps
- **Procedural Memory**: Step-by-step workflows and processes
- **Knowledge Graph**: Typed relations between entities with traversal and path finding
- **Hybrid Search**: Combined BM25 keyword + vector similarity search
- **Contradiction Detection**: NLI-based detection of conflicting information
- **Context Isolation**: Project-level namespacing (auto-detected from git remote)
- **Memory Decay**: Temporal weighting for relevance scoring
- **Web Dashboard**: Next.js frontend with GraphQL API for visual exploration

## Tools

### Search & Retrieval
| Tool | Description |
|------|-------------|
| `search` | Hybrid semantic + keyword search with context filtering |
| `get_entity` | Retrieve entity by ID |
| `list_labels` | List all categories/tags in memory |
| `list_contexts` | List all project namespaces |
| `get_context_stats` | Get entity/episode counts for a context |
| `list_entity_types` | List all available entity types with descriptions |
| `search_by_type` | Search entities by type (e.g., "preference", "decision") |

### Episodic Memory
| Tool | Description |
|------|-------------|
| `add_episode` | Store a complete conversation session |
| `search_episodes` | Search episodes by content and time range |
| `get_episode` | Retrieve episode with linked entities |
| `delete_episode` | Delete an episode |

### Graph Traversal
| Tool | Description |
|------|-------------|
| `traverse` | Explore graph connections from a starting entity |
| `find_path` | Find shortest path between two entities |

### Persistence
| Tool | Description |
|------|-------------|
| `remember` | Store entities and relations with context/importance |
| `forget` | Delete entity and its relations |

### Procedural Memory
| Tool | Description |
|------|-------------|
| `add_procedure` | Store a step-by-step workflow or process |
| `get_procedure` | Retrieve a procedure by ID |
| `search_procedures` | Search procedures by name, description, or steps |
| `list_procedures` | List all stored procedures |
| `delete_procedure` | Delete a procedure |

### Maintenance
| Tool | Description |
|------|-------------|
| `reflect` | Decay old memories, find duplicates, recalculate importance |
| `check_contradictions` | Detect conflicting information using NLI |

## Entity Types

memcp supports typed entities for better organization:

| Type | Description |
|------|-------------|
| `concept` | General knowledge or idea |
| `fact` | Verified piece of information |
| `preference` | User preference or choice |
| `requirement` | Requirement or constraint |
| `decision` | Decision that was made |
| `person` | A person |
| `organization` | Company, team, or group |
| `location` | Physical or virtual location |
| `event` | Something that happened at a specific time |
| `tool` | Software tool or technology |
| `project` | A project or initiative |
| `code` | Code snippet or technical implementation |
| `procedure` | Step-by-step workflow |
| `step` | Single step within a procedure |

## Installation

### With pip/uv

```bash
# Install with uv
uv pip install memcp

# Or with pip
pip install memcp
```

### Run directly with uvx

```bash
uvx memcp
```

### From source

```bash
git clone https://github.com/raphaelgruber/memcp
cd memcp
uv pip install -e .
```

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SURREALDB_URL` | `ws://localhost:8000/rpc` | SurrealDB connection URL |
| `SURREALDB_NAMESPACE` | `knowledge` | Database namespace |
| `SURREALDB_DATABASE` | `graph` | Database name |
| `SURREALDB_USER` | `root` | Username |
| `SURREALDB_PASS` | `root` | Password |
| `SURREALDB_AUTH_LEVEL` | `root` | Auth scope (root or database) |
| `MEMCP_DEFAULT_CONTEXT` | - | Default project context for all operations |
| `MEMCP_CONTEXT_FROM_CWD` | `false` | Auto-detect context from working directory |
| `MEMCP_ALLOW_CUSTOM_TYPES` | `true` | Allow entity types beyond predefined ones |
| `MEMCP_QUERY_TIMEOUT` | `30` | Query timeout in seconds |
| `MEMCP_LOG_FILE` | `/tmp/memcp.log` | Log file path |
| `MEMCP_LOG_LEVEL` | `INFO` | Logging level |

## Claude Desktop Config

```json
{
  "mcpServers": {
    "memory": {
      "command": "uvx",
      "args": ["memcp"],
      "env": {
        "SURREALDB_URL": "ws://localhost:8000/rpc"
      }
    }
  }
}
```

## Docker

### Docker Compose (Recommended)

The easiest way to run memcp with SurrealDB:

```bash
docker compose up -d
```

This starts:
- **SurrealDB** on port 8000
- **memcp Dashboard** on port 3000 (Next.js frontend + GraphQL API)

### Dockerfile

Build and run the container manually:

```bash
# Build
docker build -t memcp .

# Run
docker run -p 3000:3000 -p 8080:8080 \
  -e SURREALDB_URL=ws://host.docker.internal:8000/rpc \
  memcp
```

The container runs:
- **Port 3000**: Next.js web dashboard
- **Port 8080**: FastAPI GraphQL API

## Kubernetes

Deploy with Helm:

```bash
# Add values as needed
helm install memcp ./helm/memcp \
  --set surrealdb.url=ws://surrealdb:8000/rpc \
  --set surrealdb.user=root \
  --set surrealdb.pass=your-password
```

See `helm/memcp/values.yaml` for all configuration options.

## Web Dashboard

The web dashboard provides a visual interface for exploring and managing your knowledge graph.

### Features
- Entity search and browsing
- Graph visualization
- Episode and procedure management
- Maintenance tools (reflect, contradiction detection)
- Real-time statistics

### Running the Dashboard

```bash
# Install webui dependencies
uv pip install -e ".[webui]"

# Start the GraphQL API
uvicorn memcp.api.main:app --port 8080

# In another terminal, start the Next.js frontend
cd dashboard
npm install
npm run dev
```

The dashboard will be available at http://localhost:3000

### GraphQL API

The GraphQL API is available at `http://localhost:8080/graphql` with a built-in GraphiQL explorer.

## Models Used

- **Embeddings**: `all-MiniLM-L6-v2` - 384-dim vectors for semantic similarity
- **NLI**: `cross-encoder/nli-deberta-v3-base` - contradiction detection between statements

## Example Prompts

### Storing Knowledge
```
"Remember that I prefer TypeScript over JavaScript for new projects"
→ remember(entities=[{id: "pref-typescript", content: "User prefers TypeScript over JavaScript", labels: ["preference"]}])

"Store this conversation for later reference"
→ add_episode(content: "...", context: "project-x")
```

### Searching Memory
```
"What do you know about my coding preferences?"
→ search(query: "coding preferences", labels: ["preference"])

"What did we discuss last week about the API design?"
→ search_episodes(query: "API design", time_start: "2024-01-01")
```

### Project Namespacing
```
"Remember this for the memcp project"
→ remember(entities=[...], context: "memcp")

"What do I know about project X?"
→ search(query: "*", context: "project-x")
→ get_context_stats(context: "project-x")
```

### Knowledge Graph
```
"How are authentication and user-service connected?"
→ find_path(from_id: "authentication", to_id: "user-service")

"What's related to the payment system?"
→ traverse(start: "payment-system", depth: 2)
```

### Maintenance
```
"Clean up my memory and find duplicates"
→ reflect(find_similar: true, apply_decay: true)

"Check for contradictions in my preferences"
→ check_contradictions(labels: ["preference"])
```

### Importance Scoring
```
"This is very important to remember"
→ remember(entities=[{id: "...", content: "...", importance: 0.9}])

"Recalculate importance scores for all memories"
→ reflect(recalculate_importance: true)
```

### Entity Types
```
"What types of knowledge do you store?"
→ list_entity_types()

"Show all my preferences"
→ search_by_type(entity_type: "preference")

"List all decisions I've made for this project"
→ search_by_type(entity_type: "decision", context: "myproject")

"Remember this as a requirement"
→ remember(entities=[{id: "req-auth", type: "requirement", content: "API must use OAuth2"}])
```

### Procedural Memory
```
"How do I deploy the app?"
→ search_procedures(query: "deploy")

"Remember these deployment steps"
→ add_procedure(
    name: "Deploy to production",
    description: "Steps to deploy the application",
    steps: [
      {content: "Run tests with pytest"},
      {content: "Build Docker image"},
      {content: "Push to registry"},
      {content: "Update Kubernetes deployment"}
    ],
    labels: ["deployment", "devops"]
  )

"What procedures do we have for this project?"
→ list_procedures(context: "myproject")

"Show me the testing workflow"
→ get_procedure(procedure_id: "testing-workflow")
```

## Development

### Running Tests

```bash
# Install dev dependencies
uv pip install -e ".[dev]"

# Run tests (requires SurrealDB running)
pytest

# Run only unit tests (no SurrealDB required)
pytest -m "not integration"
```

### Project Structure

```
memcp/
├── memcp/                 # Core Python package
│   ├── server.py          # Main MCP server
│   ├── db.py              # Database layer & queries
│   ├── models.py          # Pydantic models
│   ├── servers/           # Modular tool servers
│   │   ├── search.py
│   │   ├── persist.py
│   │   ├── graph.py
│   │   ├── episode.py
│   │   ├── procedure.py
│   │   └── maintenance.py
│   ├── api/               # GraphQL API
│   │   ├── main.py
│   │   └── schema.py
│   └── utils/
│       ├── embedding.py   # ML models
│       └── logging.py
├── dashboard/             # Next.js frontend
├── helm/                  # Kubernetes Helm chart
├── Dockerfile
├── docker-compose.yml
└── pyproject.toml
```

## License

MIT
