# Local AI Stack

Een volledig self-hosted AI development environment met lokale modellen, cloud inference fallback, vector database voor RAG, en relationele opslag. Alles draait in Docker containers, geconfigureerd via één `docker-compose.yml`.

## Architectuur

| Service | Poort | Functie |
|---|---|---|
| Open WebUI | 3000 | Chat interface, ondersteunt meerdere modellen tegelijk |
| LiteLLM | 4000 | Proxy laag voor cloud providers (Groq, OpenAI, Anthropic) |
| Ollama | 11434 | Lokale modelserver voor open-source LLMs |
| Qdrant | 6333 | Vector database voor RAG en semantisch zoeken |
| PostgreSQL | 5432 | Relationele database voor gestructureerde data |
| pgAdmin | 5050 | Web UI voor PostgreSQL beheer |

De stack is modelagnostisch — Open WebUI praat met Ollama voor lokale modellen en met LiteLLM voor cloud-modellen, allemaal via dezelfde interface.

## Installatie

### Vereisten

- Docker Desktop of OrbStack
- Een Groq API key (gratis op groq.com)

### Setup

```bash
git clone <repo-url>
cd ai-stack
cp .env.example .env
```

Vul `.env` in met je eigen credentials:

```
GROQ_API_KEY=jouw-groq-key
POSTGRES_USER=jouw-user
POSTGRES_PASSWORD=sterk-wachtwoord
POSTGRES_DB=mydb
PGADMIN_EMAIL=jouw@email.com
PGADMIN_PASSWORD=sterk-wachtwoord
```

Start de stack:

```bash
docker compose up -d
```

## Eerste configuratie

### Open WebUI

1. Ga naar `http://localhost:3000`
2. Maak een admin account aan (blijft lokaal)
3. Instellingen → Verbindingen → voeg LiteLLM toe als OpenAI-compatibele connectie:
   - URL: `http://litellm:4000`
   - API key: `sk-1234` (willekeurig)
   - Tag: `litellm`

### Modellen pullen

```bash
docker exec -it ollama ollama pull llama3.2
docker exec -it ollama ollama pull mistral
```

## Beheer

```bash
docker compose up -d           # start alle services
docker compose down            # stop alle services
docker compose ps              # status overzicht
docker compose logs -f <name>  # logs bekijken
docker compose restart <name>  # één service herstarten
```

## Quick links

- Open WebUI: http://localhost:3000
- Qdrant Dashboard: http://localhost:6333/dashboard
- pgAdmin: http://localhost:5050
- LiteLLM: http://localhost:4000

## Beveiliging

- `.env` staat in `.gitignore` en wordt nooit gecommit
- API keys en wachtwoorden worden in macOS Keychain bewaard
- LiteLLM leest de Groq key uit de environment, niet uit `config.yaml`

## Architectuurkeuzes

**Waarom Qdrant naast PostgreSQL?** Vector search en relationele opslag zijn verschillende workloads. Qdrant is geoptimaliseerd voor schaalbare vector search met HNSW indexing, PostgreSQL voor gestructureerde data. Scheiden geeft duidelijkere verantwoordelijkheden en betere schaalbaarheid.

**Waarom LiteLLM als proxy?** Modelagnostische architectuur — Open WebUI hoeft niet te weten of het model bij Groq, OpenAI of Anthropic draait. Switchen van provider gaat met één regel in `config.yaml`. Maakt fallback strategieën mogelijk.

**Waarom Ollama als container in plaats van native app?** Alle services in hetzelfde Docker netwerk, bereikbaar via service-naam (`http://ollama:11434`). Eén bron van waarheid, geen `host.docker.internal` workarounds.

## Volgende stappen

- [ ] Redis toevoegen voor caching
- [ ] pgvector extensie installeren in PostgreSQL
- [ ] Backup strategie voor volumes
- [ ] Hermes Agent toevoegen voor autonome workflows
