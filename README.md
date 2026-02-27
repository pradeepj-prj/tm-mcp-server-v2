# TM Skills MCP Server v2

An [MCP](https://modelcontextprotocol.io/) server that exposes the Talent Management Skills API — including attrition prediction — as tools, resources, and prompts for AI assistants (e.g. Joule agents on SAP BTP).

## How it works

```
AI Assistant (Joule / Claude / etc.)
        |
        |  MCP protocol (Streamable HTTP)
        v
+---------------------+
|  TM Skills MCP      |  <-- This project
|  Server v2          |
+--------+------------+
         |  HTTP + API key
         v
+---------------------+
|  TM Skills API      |  <-- Separate FastAPI service
|  (REST)             |
+--------+------------+
         |
         v
    PostgreSQL
```

The MCP server is a thin wrapper — it translates MCP tool calls into HTTP requests to the TM Skills API. Authentication, rate limiting, and input validation all happen at the API layer.

## Project structure

```
server.py              Main server: MCP tools, resources, prompts, UI view, audit REST
audit.py               SQLite-backed audit logger (lazy init, WAL mode)
config.py              Configuration via pydantic-settings (.env + env vars)
resources/
  tm_schema.sql        TM database schema (served as MCP resource)
  business_questions.md  Business questions catalog (served as MCP resource)
deploy.sh              Automated CF deployment with secret management
manifest.yml           CF app config (memory, health check, env vars)
Procfile               CF start command
requirements.txt       Production deps for CF Python buildpack
runtime.txt            Python version pin for CF
pyproject.toml         Project metadata and dependencies
.env.example           Environment variable template
```

## Setup

**Prerequisites:** Python 3.10+, access to the TM Skills API

```bash
pip install -e .
cp .env.example .env       # Set TM_API_BASE_URL and TM_API_KEY
python server.py           # Starts on http://localhost:8080
```

| URL | What |
|-----|------|
| `http://localhost:8080/` | UI view — lists all tools, resources, and prompts |
| `http://localhost:8080/mcp` | MCP endpoint (for AI clients) |
| `http://localhost:8080/audit/summary` | Audit stats (JSON) |

## Deployment (Cloud Foundry — SAP BTP)

### Using `deploy.sh`

```bash
./deploy.sh
```

The script:
1. Reads the API key from `../talent-management-app/.api-key`
2. Runs `cf push --no-start` (deploys without starting)
3. Injects the key via `cf set-env TM_API_KEY`
4. Starts the app

This keeps the secret out of `manifest.yml` and version control. If the key file doesn't exist, it prompts for manual input.

### Manual deploy

```bash
cf push --no-start
cf set-env tm-skills-mcp-v2 TM_API_KEY "your-api-key"
cf start tm-skills-mcp-v2
```

### Important notes

- **`manifest.yml` must NOT contain `TM_API_KEY`** — `cf push` re-applies manifest env vars on every deploy, which would overwrite the value set by `cf set-env`.
- **`$PORT` is set by CF** — the server reads it from the environment automatically.
- 256M memory is sufficient for the MCP SDK + httpx stack.
- The audit SQLite DB is ephemeral — it resets on each redeploy.

### Live URLs

| URL | What |
|-----|------|
| https://tm-skills-mcp-v2.cfapps.ap10.hana.ondemand.com/ | UI view |
| https://tm-skills-mcp-v2.cfapps.ap10.hana.ondemand.com/mcp | MCP endpoint |
| https://tm-skills-mcp-v2.cfapps.ap10.hana.ondemand.com/audit/summary | Audit stats |
| https://tm-skills-mcp-v2.cfapps.ap10.hana.ondemand.com/audit/recent | Recent tool calls |

**CF org/space:** `SEAIO_dial-3-0-zme762l7 / dev`

## MCP tools, resources, and prompts

The server exposes **21 tools** (18 TM API + 3 audit), **2 resources**, and **5 prompts**.

Visit the [UI view](https://tm-skills-mcp-v2.cfapps.ap10.hana.ondemand.com/) for a live listing with descriptions and parameters, or run `python server.py` locally and open `http://localhost:8080/`.

**Connecting an AI client:** Point any MCP client at the `/mcp` endpoint. In Joule Studio, add the URL as an MCP tool server.

## Configuration

All settings come from environment variables (or a `.env` file locally):

| Variable | Default | Description |
|----------|---------|-------------|
| `HOST` | `0.0.0.0` | Server bind address |
| `PORT` | `8080` | Server port (CF sets this automatically) |
| `TM_API_BASE_URL` | `http://localhost:8000` | TM Skills API URL |
| `TM_API_KEY` | *(empty)* | API key for the TM Skills API |
| `TM_API_TIMEOUT` | `30.0` | HTTP request timeout in seconds |
| `AUDIT_DB_PATH` | `audit.db` | SQLite audit database file path |
| `CORS_ORIGINS` | `http://localhost:5173,...` | Comma-separated CORS origins (for monitoring dashboard) |

## ID formats

These are validated by the API — useful for testing:

- **Employee IDs:** `EMP` + 6 digits (e.g. `EMP000001`)
- **Org IDs:** `ORG` + 1-4 digits + optional letter (e.g. `ORG030`, `ORG031B`)
- **Skill IDs:** numeric (e.g. `1`, `42`)
- **Categories:** `technical`, `functional`, `leadership`, `domain`, `tool`, `other`
