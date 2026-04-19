# AGENTS.md

## Project Overview

Dify is an open-source platform for developing LLM applications with an intuitive interface combining agentic AI workflows, RAG pipelines, agent capabilities, and model management.

The codebase is split into:

- **Backend API** (`/api`): Python Flask application organized with Domain-Driven Design
- **Frontend Web** (`/web`): Next.js application using TypeScript and React
- **Docker deployment** (`/docker`): Containerized deployment configurations

## Backend Workflow

- Read `api/AGENTS.md` for details
- Run backend CLI commands through `uv run --project api <command>`.
- Integration tests are CI-only and are not expected to run in the local environment.

## Frontend Workflow

- Read `web/AGENTS.md` for details

## Testing & Quality Practices

- Follow TDD: red → green → refactor.
- Use `pytest` for backend tests with Arrange-Act-Assert structure.
- Enforce strong typing; avoid `Any` and prefer explicit type annotations.
- Write self-documenting code; only add comments that explain intent.

## Language Style

- **Python**: Keep type hints on functions and attributes, and implement relevant special methods (e.g., `__repr__`, `__str__`). Prefer `TypedDict` over `dict` or `Mapping` for type safety and better code documentation.
- **TypeScript**: Use the strict config, rely on ESLint (`pnpm lint:fix` preferred) plus `pnpm type-check`, and avoid `any` types.

## General Practices

- Prefer editing existing files; add new documentation only when requested.
- Inject dependencies through constructors and preserve clean architecture boundaries.
- Handle errors with domain-specific exceptions at the correct layer.

## Project Conventions

- Backend architecture adheres to DDD and Clean Architecture principles.
- Async work runs through Celery with Redis as the broker.
- Frontend user-facing strings must use `web/i18n/en-US/`; avoid hardcoded text.

## Heartbeats vers orchestrateur

Les heartbeats transitent via Redis (wa-redis container, port 6379 interne / 6380 hôte).

```bash
# Progression
docker exec wa-redis redis-cli xadd orchestrator:heartbeats '*' type PROGRESS session <NOM_SESSION> msg "description étape"
# Blocage
docker exec wa-redis redis-cli xadd orchestrator:heartbeats '*' type BLOCKED session <NOM_SESSION> msg "description problème"
# Fin de tâche
docker exec wa-redis redis-cli xadd orchestrator:heartbeats '*' type DONE session <NOM_SESSION> msg "résumé tâche"
```

Remplacer `<NOM_SESSION>` par le nom de la session courante (ex: `binance-trading-codex`).

**Vérification** : après envoi, le message doit apparaître dans le stream :
```bash
docker exec wa-redis redis-cli xrevrange orchestrator:heartbeats + - COUNT 1
```

**Escalade question** :
```bash
docker exec wa-redis redis-cli xadd orchestrator:heartbeats '*' type QUESTION session <NOM_SESSION> msg "question courte"
```
