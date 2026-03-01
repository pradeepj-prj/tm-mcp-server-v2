# Architecture — TM Skills MCP Server v2

## Overview

The TM Skills MCP Server v2 is a **protocol bridge** that translates AI agent requests (MCP protocol) into HTTP API calls against a Talent Management Skills API. It adds attrition prediction capabilities, audit logging, and a web UI on top of the original v1 server.

The server does **not** access the database directly — all data flows through the REST API, preserving the API's authentication, rate limiting, and input validation.

## System Architecture

```mermaid
graph TB
    subgraph Clients["AI Clients"]
        Joule["Joule Studio<br/>(SAP BTP)"]
        Claude["Claude Code /<br/>Claude Desktop"]
        Custom["Custom MCP<br/>Client"]
    end

    subgraph BTP["SAP BTP — Cloud Foundry"]
        subgraph MCPServer["TM Skills MCP Server v2"]
            FastMCP["FastMCP Framework"]
            Tools["21 MCP Tools"]
            Resources["2 MCP Resources"]
            Prompts["5 MCP Prompts"]
            Audit["Audit Logger<br/>(SQLite)"]
            UI["Web UI<br/>(HTML Dashboard)"]
            AuditREST["Audit REST<br/>Endpoints"]
        end

        subgraph API["TM Skills API (FastAPI)"]
            Auth["API Key Auth"]
            RateLimit["Rate Limiter<br/>(60/min)"]
            Validation["Input Validation"]
            Endpoints["18 REST Endpoints"]
        end
    end

    DB[("PostgreSQL<br/>hr_data / tm schema")]

    Joule -->|"MCP Protocol<br/>(Streamable HTTP)"| FastMCP
    Claude -->|"MCP Protocol<br/>(Streamable HTTP)"| FastMCP
    Custom -->|"MCP Protocol<br/>(Streamable HTTP)"| FastMCP

    FastMCP --> Tools
    FastMCP --> Resources
    FastMCP --> Prompts
    Tools -->|"@audited decorator"| Audit
    Tools -->|"_api_get()"| Auth
    UI -->|"GET /"| FastMCP
    AuditREST -->|"GET /audit/*"| Audit

    Auth --> RateLimit --> Validation --> Endpoints
    Endpoints -->|"asyncpg (TLS)"| DB
```

## Data Flow

```mermaid
sequenceDiagram
    participant User as HR Manager
    participant AI as AI Agent (Joule/Claude)
    participant MCP as MCP Server v2
    participant AuditDB as SQLite (Audit)
    participant API as TM Skills API
    participant DB as PostgreSQL

    User->>AI: "Who are the high-risk employees in ORG030?"

    Note over AI: Reads tool descriptions,<br/>selects appropriate tools

    AI->>MCP: tools/call: get_org_attrition_summary(org_unit_id="ORG030")
    MCP->>MCP: @audited: start timer
    MCP->>API: GET /tm/attrition/orgs/ORG030/summary
    API->>API: Auth ✓ → Rate limit ✓ → Validate ✓
    API->>DB: SQL query (org hierarchy + attrition data)
    DB-->>API: Result rows
    API-->>MCP: JSON response
    MCP->>AuditDB: INSERT tool_call record
    MCP-->>AI: MCP result (JSON text)

    Note over AI: Identifies top-risk employees,<br/>decides to get details

    AI->>MCP: tools/call: get_employee_attrition_risk(employee_id="EMP000042")
    MCP->>API: GET /tm/attrition/employees/EMP000042
    API->>DB: SQL query
    DB-->>API: Result
    API-->>MCP: JSON response
    MCP->>AuditDB: INSERT tool_call record
    MCP-->>AI: MCP result

    AI->>MCP: tools/call: get_employee_skills(employee_id="EMP000042")
    MCP->>API: GET /tm/employees/EMP000042/skills
    API-->>MCP: JSON response
    MCP-->>AI: MCP result

    Note over AI: Composes final answer<br/>with risk factors + skill context

    AI-->>User: "ORG030 has 3 critical-risk employees..."
```

## Component Details

### MCP Server (`server.py`)

The core of the project. Responsibilities:
- **21 MCP Tools**: Each tool wraps a single API endpoint. The AI agent reads tool descriptions to decide which to call.
- **2 Resources**: Static context (database schema, business questions) loaded into the AI's context window.
- **5 Prompts**: Reusable multi-step workflows that guide the AI through complex analyses.
- **Audit REST endpoints**: Plain HTTP endpoints (`/audit/recent`, `/audit/query`, `/audit/summary`) for querying audit data without an MCP client.
- **UI view**: HTML dashboard at `/` listing all tools, resources, and prompts.

### Audit Logger (`audit.py`)

SQLite-backed audit trail for compliance and debugging:
- Records every tool invocation (tool name, parameters, duration, success/failure)
- Captures MCP session metadata (session ID, client name/version)
- WAL mode for concurrent read/write performance
- Lazy initialization — works even if no MCP client has connected yet

### Configuration (`config.py`)

Environment-aware settings via `pydantic-settings`:
- Reads from `.env` locally, from CF environment variables in production
- No code changes between local dev and production deployment

## Deployment

```mermaid
graph LR
    subgraph Dev["Local Development"]
        DevServer["python server.py<br/>localhost:8080"]
        DevAPI["TM Skills API<br/>localhost:8000"]
        DevDB[("PostgreSQL<br/>localhost:5432")]
        DevServer -->|HTTP| DevAPI -->|SQL| DevDB
    end

    subgraph Prod["SAP BTP — Cloud Foundry (ap10)"]
        ProdServer["tm-skills-mcp-v2<br/>256M memory"]
        ProdAPI["tm-skills-api"]
        ProdDB[("PostgreSQL<br/>AWS ap-southeast-1")]
        ProdServer -->|"HTTPS + API Key"| ProdAPI -->|"asyncpg (TLS)"| ProdDB
    end

    Deploy["./deploy.sh"] -->|"cf push + cf set-env"| ProdServer
```

## Security Model

| Layer | Mechanism |
|-------|-----------|
| AI Client → MCP Server | MCP protocol over HTTPS (TLS in production) |
| MCP Server → TM API | API key in `X-API-Key` header |
| TM API → PostgreSQL | Connection credentials (TLS) |
| Secret management | API key injected via `cf set-env`, never in manifests or VCS |
| Rate limiting | 60 requests/min enforced at API layer |
| Input validation | Pydantic models at API layer (ID format, value ranges) |
| CORS | Restricted origins for monitoring dashboard |
