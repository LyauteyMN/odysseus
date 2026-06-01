# Running Odysseus Locally on Windows

This guide documents the local Windows path that was validated in this repository.

## What this project actually needs

- Backend: FastAPI app served by `uvicorn app:app`
- Frontend: static files under `static/` (no frontend build step required for the core UI)
- Bootstrap: `setup.py` creates `data/`, `logs/`, the SQLite database, `.env`, and the first admin user
- Optional services:
  - ChromaDB for vector memory
  - SearXNG for self-hosted web search
  - Node.js / `npx` for the optional Browser MCP integration

## Prerequisites

- Windows PowerShell
- Python 3.11+
- Internet access for the first dependency install

Recommended:

- Use `python -m pip` instead of plain `pip` on Windows. It avoids broken launcher/path issues.

## One-time setup

From the repository root:

```powershell
python -m venv venv
.\venv\Scripts\python.exe -m pip install -r requirements.txt
.\venv\Scripts\python.exe setup.py
```

What `setup.py` does:

- creates `data/` and `logs/`
- creates `.env` from `.env.example` if needed
- initializes `data/app.db`
- creates the first admin user in `data/auth.json`

If you want to choose the first admin password yourself, set `ODYSSEUS_ADMIN_PASSWORD` in `.env` before running `setup.py` for the first time.

## Start the app

For a local-only development run:

```powershell
.\venv\Scripts\python.exe -m uvicorn app:app --host 127.0.0.1 --port 7000
```

Then open:

```text
http://127.0.0.1:7000
```

## Quick verification

Health check:

```powershell
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:7000/api/health | Select-Object -ExpandProperty Content
```

Root page check:

```powershell
(Invoke-WebRequest -UseBasicParsing http://127.0.0.1:7000/).StatusCode
```

Expected results:

- `/api/health` returns JSON with `"status":"healthy"`
- `/` returns HTTP `200`

## First login

On the first successful `setup.py` run, the script prints a temporary admin password.

- Username default: `admin`
- Password source: printed by `setup.py`

If you lose that password before logging in:

1. set `ODYSSEUS_ADMIN_PASSWORD` in `.env`
2. delete `data/auth.json`
3. run `.\venv\Scripts\python.exe setup.py` again

## What works without extra services

The core application boots and serves the UI with only Python installed.

You can:

- open the UI
- log in
- use the local database and core app settings
- configure model providers later from inside the app

## What is degraded until you add optional services

### ChromaDB not running

If no ChromaDB server is available, the app still boots, but vector-backed memory features degrade gracefully.

Typical log message:

```text
MemoryVectorStore DEGRADED: ChromaDB vector memory unavailable
```

This is not fatal for local startup.

### No LLM endpoint configured yet

The app still boots, but chat/research features need a configured provider in Settings.

### No SearXNG instance

Core UI still loads, but self-hosted web search and some research flows will not be fully available until you point the app at a working search provider.

Practical Windows fallback:

- set the search provider to `duckduckgo` instead of `searxng`
- this avoids needing Docker or WSL just to get web search working

### Browser MCP on Windows

The Browser MCP integration is optional. Core startup does not depend on it, but browser-automation features may require extra Playwright/browser setup on the machine.

## Windows-specific notes

- The first startup may download the local FastEmbed model into `data/fastembed_cache/`.
- On Windows without Developer Mode, Hugging Face cache symlink warnings are expected and non-fatal.
- `python-magic` is optional here; the app falls back to basic file detection if it is missing.
- PTY-style shell streaming is not available on Windows; normal shell execution still works.

## Optional local services on Windows

### Local Ollama backend

If you want a local chat backend on Windows, Ollama works with Odysseus.

Verified local binary path:

```text
%LOCALAPPDATA%\Programs\Ollama\ollama.exe
```

Example commands:

```powershell
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" --version
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" pull qwen2.5:1.5b
```

Then add the endpoint in Odysseus as:

```text
http://127.0.0.1:11434/v1
```

Verified lightweight model:

- `qwen2.5:1.5b`

### Local ChromaDB backend

If you want vector memory without Docker, you can run ChromaDB from the project venv:

```powershell
.\venv\Scripts\python.exe -m pip install chromadb
.\venv\Scripts\chroma.exe run --host 127.0.0.1 --port 8100 --path .\data\chroma
```

After starting ChromaDB, restart Odysseus so startup can connect to it.

## Stop the app

In the terminal running Uvicorn, press `Ctrl+C`.

## Summary

Validated local Windows path:

```powershell
python -m venv venv
.\venv\Scripts\python.exe -m pip install -r requirements.txt
.\venv\Scripts\python.exe setup.py
.\venv\Scripts\python.exe -m uvicorn app:app --host 127.0.0.1 --port 7000
```

This is the shortest path to a working local Odysseus instance on Windows without Docker.
