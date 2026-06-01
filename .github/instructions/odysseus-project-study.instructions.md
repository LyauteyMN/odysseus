---
description: "Use when studying or explaining how the Odysseus project works, doing repo onboarding, tracing feature flow, debugging cross-layer behavior, or entendendo o funcionamento do projeto, arquitetura, rotas, servicos, frontend, dados e integracoes."
name: "Odysseus Project Study Guide"
---
# Odysseus Project Study Guide

- Use `README.md` as the canonical product overview for features, deployment modes, bundled services, security assumptions, and the top-level architecture sketch.
- Use `LICENSE` and `ACKNOWLEDGMENTS.md` only for legal, attribution, and reuse questions. They do not define runtime behavior.

## Mental Model

- Treat `app.py` as the orchestration entrypoint. Start there to understand middleware, auth, timeout exemptions, static mounting, and route registration.
- For any feature, trace this path before making claims: `routes/` endpoint -> `src/` handler or processor -> `services/` or `core/` domain logic -> storage in `data/`, SQLite, Chroma, or external integrations.
- Keep layer boundaries explicit:
  - `core/`: auth, database, middleware, constants, shared models, and exceptions.
  - `routes/`: FastAPI HTTP surface per feature.
  - `src/`: request orchestration, LLM and agent loop runtime, handlers, background jobs, and shared runtime helpers.
  - `services/`: domain implementations and adapters.
  - `mcp_servers/`: MCP tool servers exposed to the agent stack.
  - `static/`: production frontend assets. Baseline local startup does not require an npm build step.
  - `tests/`: pytest coverage for regressions and intended flows.

## How To Study A Feature

- Start from the user-visible feature name in `README.md` or the relevant route file in `routes/`.
- Read the endpoint, then jump to the first function or class that computes behavior instead of staying in wrappers or registration code.
- Inspect the nearest test in `tests/` before broad searching. Use tests to confirm intended behavior, regressions, and edge cases.
- For UI-driven behavior, trace `static/index.html`, `static/app.js`, or files in `static/js/` to the API endpoints they call.
- For persistence, verify whether the feature uses JSON files under `data/`, SQLAlchemy sessions and tables, Chroma or vector storage, or uploaded/generated files.
- For external dependencies, separate required systems from optional integrations. ChromaDB, search providers, email, and model servers may degrade gracefully and should not be assumed mandatory.

## Concrete Flow Anchors

- Chat and agent flow usually starts in `routes/chat_routes.py`, then moves into `src/chat_handler.py`, `src/chat_processor.py`, `src/agent_loop.py`, and `src/llm_core.py`.
- Memory flow usually starts in `routes/memory_routes.py`, then moves into `src/memory.py`, `src/memory_vector.py`, and Chroma-backed storage under `data/chroma/` or the configured Chroma service.
- Research flow usually starts in `routes/research_routes.py`, then moves into `src/research_handler.py`, `src/deep_research.py`, and related search or synthesis helpers.

## Important Project Realities

- Odysseus is a local-first FastAPI app with a modular vanilla-JS frontend, agent tooling, memory and RAG, research, email and calendar features, and model-serving integrations.
- Security is part of functionality. When analyzing a flow, account for admin gates, API tokens, localhost bypass, internal-tool loopback headers, and per-user privileges.
- Deployment context matters. Use `.env.example`, `docker-compose.yml`, `Dockerfile`, `pyproject.toml`, and `setup.py` when answering setup or runtime questions.
- User and runtime state live under `data/` and are intentionally gitignored.

## Baseline Reading Order

- `README.md`
- `app.py`
- `core/constants.py`
- `core/database.py`
- the relevant file in `routes/`
- the matching orchestrator in `src/`
- the supporting implementation in `services/` or `core/`
- the nearest regression test in `tests/`

## Response Expectations

- Ground explanations in actual files and concrete flows, not generic FastAPI assumptions.
- Distinguish product overview from verified implementation details.
- When uncertain, say which file or abstraction boundary still needs verification instead of guessing.
- Prefer concise architecture maps and always point to the concrete files that control the behavior.