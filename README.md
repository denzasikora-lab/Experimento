# Experimento — AI-Driven Formulation & R&D Co-Pilot

Intelligent platform for chemists, biologists, and pharmaceutical scientists: design formulations, predict synthesis success, toxicity and stability, run stress-test simulations, get traceable rationale, and keep an immutable audit trail.

## Stack

- **Backend**: .NET 10 / C#, Clean Architecture (Domain / Application / Infrastructure / Ai / WebApi)
- **Queues**: MassTransit + RabbitMQ (async heavy computations)
- **Database**: PostgreSQL 17 + pgvector (relational data + vector search)
- **AI**: Microsoft Semantic Kernel, provider-agnostic (OpenAI / Azure OpenAI / Ollama / none)
- **Frontend**: Next.js (App Router) + TypeScript strict + Tailwind + shadcn/ui + Recharts
- **Auth**: JWT access token + refresh token in httpOnly cookie, BCrypt passwords

## Prerequisites

- Docker (PostgreSQL + RabbitMQ run via docker-compose)
- .NET 10 SDK
- Node.js 20+

## Quick start

```bash
# 1. Infrastructure
docker compose up -d

# 2. Backend
cd backend
dotnet run --project src/Experimento.WebApi

# 3. Frontend
cd frontend
npm install
npm run dev
```

- API: http://localhost:5080 (Swagger UI included)
- Frontend: http://localhost:3000

## Configuration

All secrets and connection settings come from environment variables (see `backend/src/Experimento.WebApi/appsettings.json` for keys). Never commit secrets.

Key settings:

| Setting | Description |
|---|---|
| `ConnectionStrings__Default` | PostgreSQL connection string |
| `RabbitMq__Host` / `RabbitMq__Username` / `RabbitMq__Password` | RabbitMQ connection |
| `Jwt__Key` | JWT signing key (min 32 chars) |
| `Ai__Provider` | `openai` \| `azureopenai` \| `ollama` \| `none` |
| `Ai__OpenAi__ApiKey` | OpenAI API key (only when provider=openai) |

With `Ai__Provider=none` the system runs fully offline: heuristic predictions work, rationale is marked "LLM not configured".

## Проверки

```bash
(cd backend && dotnet test)
(cd frontend && npm run lint && npx tsc --noEmit)
```

Для браузерных проверок задайте `JWT_KEY` в `.env`, затем из корня репозитория
запустите тестовый стек и заполните локальный каталог веществ:

```bash
docker compose -f docker-compose.yml -f .github/compose.ci.yml up -d --build
docker compose -f docker-compose.yml -f .github/compose.ci.yml exec -T postgres psql -v ON_ERROR_STOP=1 -U experimento -d experimento < .github/ci/seed-catalog.sql
cd frontend
npm ci
npx playwright install chromium
npm run test:e2e:ci
```

После проверки остановите стек из корня репозитория:

```bash
docker compose -f docker-compose.yml -f .github/compose.ci.yml down
```

`npm run test:e2e` запускает весь набор браузерных сценариев, включая проверки,
которым нужны внешние сервисы. В CI выполняется набор `@ci` с локальным каталогом.
