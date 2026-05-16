# Local AI Stack

A self-hosted AI development environment with local models, cloud inference fallback, vector database for RAG, and relational storage. Docker Compose orchestrates the services that benefit from containerization, while Ollama runs natively on macOS to leverage Metal GPU acceleration.

## Architecture

| Service | Runs in | Port | Function |
|---|---|---|---|
| **Ollama** | Native (macOS) | 11434 | Local model server with Metal GPU acceleration |
| Open WebUI | Docker | 3000 | Chat interface supporting multiple models |
| LiteLLM | Docker | 4000 | Proxy layer for cloud providers with admin UI |
| Qdrant | Docker | 6333 | Vector database for RAG and semantic search |
| PostgreSQL | Docker | 5432 | Relational database, also used by LiteLLM for stats |
| pgAdmin | Docker | 5050 | Web UI for PostgreSQL management |

## Why Ollama Runs Natively

Docker containers on macOS cannot access the Metal GPU. Running Ollama in a container forces it to use CPU-only inference, which is 10-30x slower than native execution. Native Ollama uses Metal acceleration on Apple Silicon, while Open WebUI in Docker reaches it via `host.docker.internal:11434`.

This separation reflects a broader principle: containerize what benefits from isolation and orchestration, run natively what needs hardware access.

## Installation

### Requirements

- Docker Desktop or OrbStack
- Native Ollama installed from [ollama.com/download](https://ollama.com/download)
- A Groq API key (free at [groq.com](https://groq.com))
- Optional: Google AI Studio API key for Gemini access

### Setup

```bash
git clone https://github.com/george-panaite/ai-stack.git
cd ai-stack
cp .env.example .env
```

Fill in `.env` with your own credentials:

```
GROQ_API_KEY=your-groq-key
GEMINI_API_KEY_1=your-gemini-key
GEMINI_API_KEY_2=your-second-gemini-key
POSTGRES_USER=your-user
POSTGRES_PASSWORD=strong-password
POSTGRES_DB=mydb
PGADMIN_EMAIL=your@email.com
PGADMIN_PASSWORD=strong-password
LITELLM_MASTER_KEY=sk-your-strong-random-string
UI_USERNAME=admin
UI_PASSWORD=strong-password
```

Start the Docker services:

```bash
docker compose up -d
```

Pull models for native Ollama:

```bash
ollama pull qwen3:4b
ollama pull nomic-embed-text
```

## Initial Configuration

### Open WebUI

1. Open `http://localhost:3000`
2. Create an admin account (stays local)
3. Settings → Connections → add LiteLLM as an OpenAI-compatible connection:
   - URL: `http://litellm:4000`
   - API key: your `LITELLM_MASTER_KEY` value
   - Tag: `litellm`

### LiteLLM Admin UI

Open `http://localhost:4000/ui` and login with:
- Username: `admin`
- Password: your `UI_PASSWORD` value

Here you can view request logs, usage statistics, configure virtual keys with budgets and rate limits, and manage which models each key can access.

## Management

```bash
docker compose up -d           # start all Docker services
docker compose down            # stop all Docker services
docker compose ps              # status overview
docker compose logs -f <name>  # view logs
docker compose restart <name>  # restart one service
```

Native Ollama runs in the background after installation. Use the menubar app to manage it, or `ollama list` / `ollama pull <model>` from the terminal.

## Quick Links

- Open WebUI: http://localhost:3000
- LiteLLM Admin: http://localhost:4000/ui
- Qdrant Dashboard: http://localhost:6333/dashboard
- pgAdmin: http://localhost:5050

## Security

- `.env` is in `.gitignore` and never committed
- API keys and passwords stored in macOS Keychain
- LiteLLM reads provider keys from environment variables, not from `config.yaml`
- Virtual keys can be issued with per-app restrictions and budgets

## Architecture Decisions

**Why Ollama native, the rest in Docker?** Metal GPU access is the deciding factor. Containerized Ollama on macOS forces CPU-only inference, which is unacceptably slow. Open WebUI and other services don't need GPU access and benefit from container isolation and orchestration.

**Why Qdrant alongside PostgreSQL?** Vector search and relational storage are different workloads. Qdrant is optimized for scalable vector search with HNSW indexing, PostgreSQL for structured data and LiteLLM's tracking tables. Separation gives clearer responsibilities and better scalability.

**Why LiteLLM as a proxy?** Model-agnostic architecture — Open WebUI doesn't need to know whether the model runs at Groq, OpenAI or Google. Switching providers is a one-line change in `config.yaml`. Enables fallback strategies and centralized usage tracking. Virtual keys add governance: per-app budgets, model restrictions, and rate limits without exposing provider credentials.

## Roadmap

- [ ] Add Redis for caching
- [ ] Install pgvector extension in PostgreSQL
- [ ] Backup strategy for volumes
- [ ] First RAG experiment with Qdrant
- [ ] Add Hermes Agent for autonomous workflows on VPS

## License

MIT