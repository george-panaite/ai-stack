# Local AI Stack

A fully self-hosted AI development environment with local models, cloud inference fallback, vector database for RAG, and relational storage. Everything runs in Docker containers, configured via a single `docker-compose.yml`.

## Architecture

| Service | Port | Function |
|---|---|---|
| Open WebUI | 3000 | Chat interface supporting multiple models |
| LiteLLM | 4000 | Proxy layer for cloud providers (Groq, OpenAI, Anthropic) |
| Ollama | 11434 | Local model server for open-source LLMs |
| Qdrant | 6333 | Vector database for RAG and semantic search |
| PostgreSQL | 5432 | Relational database for structured data |
| pgAdmin | 5050 | Web UI for PostgreSQL management |

The stack is model-agnostic — Open WebUI talks to Ollama for local models and to LiteLLM for cloud models, all through the same interface.

## Installation

### Requirements

- Docker Desktop or OrbStack
- A Groq API key (free at groq.com)

### Setup

```bash
git clone https://github.com/george-panaite/ai-stack.git
cd ai-stack
cp .env.example .env
```

Fill in `.env` with your own credentials:

```
GROQ_API_KEY=your-groq-key
POSTGRES_USER=your-user
POSTGRES_PASSWORD=strong-password
POSTGRES_DB=mydb
PGADMIN_EMAIL=your@email.com
PGADMIN_PASSWORD=strong-password
```

Start the stack:

```bash
docker compose up -d
```

## Initial Configuration

### Open WebUI

1. Open `http://localhost:3000`
2. Create an admin account (stays local)
3. Settings → Connections → add LiteLLM as an OpenAI-compatible connection:
   - URL: `http://litellm:4000`
   - API key: `sk-1234` (any value works)
   - Tag: `litellm`

### Pulling models

```bash
docker exec -it ollama ollama pull llama3.2
docker exec -it ollama ollama pull mistral
```

## Management

```bash
docker compose up -d           # start all services
docker compose down            # stop all services
docker compose ps              # status overview
docker compose logs -f <name>  # view logs
docker compose restart <name>  # restart one service
```

## Quick Links

- Open WebUI: http://localhost:3000
- Qdrant Dashboard: http://localhost:6333/dashboard
- pgAdmin: http://localhost:5050
- LiteLLM: http://localhost:4000

## Security

- `.env` is in `.gitignore` and never committed
- API keys and passwords stored in macOS Keychain
- LiteLLM reads the Groq key from environment, not from `config.yaml`

## Architecture Decisions

**Why Qdrant alongside PostgreSQL?** Vector search and relational storage are different workloads. Qdrant is optimized for scalable vector search with HNSW indexing, PostgreSQL for structured data. Separation gives clearer responsibilities and better scalability.

**Why LiteLLM as a proxy?** Model-agnostic architecture — Open WebUI doesn't need to know whether the model runs at Groq, OpenAI or Anthropic. Switching providers is a one-line change in `config.yaml`. Enables fallback strategies.

**Why Ollama as a container instead of the native app?** All services in the same Docker network, reachable via service name (`http://ollama:11434`). Single source of truth, no `host.docker.internal` workarounds.

## Roadmap

- [ ] Add Redis for caching
- [ ] Install pgvector extension in PostgreSQL
- [ ] Backup strategy for volumes
- [ ] Add Hermes Agent for autonomous workflows
