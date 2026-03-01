# TM Skills MCP Server v2 — Documentation

This directory contains all project documentation, architecture diagrams, and reference materials.

## File Index

| File | Purpose |
|------|---------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | Current system architecture with Mermaid diagrams — how the MCP server, API, and database fit together |
| [MCP_GUIDE.md](MCP_GUIDE.md) | Comprehensive introduction to the Model Context Protocol (MCP) for newcomers — concepts, communication lifecycle, transports, and how this server implements them |
| [SOLUTION_ALTERNATIVES.md](SOLUTION_ALTERNATIVES.md) | Overview of 6 different approaches to solving the talent intelligence problem within the SAP BTP ecosystem, with trade-off analysis |
| [dependency-graph.md](dependency-graph.md) | Function-level dependency graph showing how every function in every source file connects to others |
| [business-questions.md](business-questions.md) | Catalog of 17 business questions the system can answer, mapped to API endpoints and MCP tools |

## Diagrams

Each file in `diagrams/` details one solution architecture with a Mermaid diagram, component descriptions, pros/cons, and guidance on when to use it.

| File | Solution Approach |
|------|-------------------|
| [diagrams/current-architecture.md](diagrams/current-architecture.md) | **Current** — MCP Server + REST API + PostgreSQL (what we have today) |
| [diagrams/datasphere-joule.md](diagrams/datasphere-joule.md) | **Datasphere + Joule** — Ingest into SAP Datasphere, curate data, connect a Joule agent |
| [diagrams/hana-cap-mcp.md](diagrams/hana-cap-mcp.md) | **HANA Cloud + CAP + MCP** — Build on CAP framework with HANA Cloud, expose via MCP |
| [diagrams/integration-suite-aicore.md](diagrams/integration-suite-aicore.md) | **Integration Suite + AI Core** — Orchestrate data with CPI, run ML models on AI Core |
| [diagrams/successfactors-extension.md](diagrams/successfactors-extension.md) | **SuccessFactors Extension** — Build directly on SAP SuccessFactors with BTP side-by-side extension |
| [diagrams/hybrid-datasphere-mcp.md](diagrams/hybrid-datasphere-mcp.md) | **Hybrid: Datasphere + MCP** — Enterprise data curation in Datasphere with MCP for broad AI agent access |

## How to read these docs

- **New to MCP?** Start with [MCP_GUIDE.md](MCP_GUIDE.md)
- **Understanding this project?** Read [ARCHITECTURE.md](ARCHITECTURE.md) then [dependency-graph.md](dependency-graph.md)
- **Evaluating approaches?** Start with [SOLUTION_ALTERNATIVES.md](SOLUTION_ALTERNATIVES.md), then dive into the specific diagram files
- **Building on the API?** Check [business-questions.md](business-questions.md) for what queries are supported

## Rendering Mermaid diagrams

All architecture diagrams use [Mermaid](https://mermaid.js.org/) syntax. They render automatically on:
- **GitHub** — native Mermaid support in markdown
- **VS Code** — install the "Markdown Preview Mermaid Support" extension
- **Online** — paste into [mermaid.live](https://mermaid.live/)
