# Solution 3: HANA Cloud + CAP + MCP

> **Build the talent management service using SAP Cloud Application Programming (CAP) model on HANA Cloud, then wrap it with an MCP server for AI agent access.** Combines SAP's enterprise database with the flexibility of MCP.

## Architecture

```mermaid
graph TB
    subgraph Clients["AI Clients"]
        Joule["Joule Studio"]
        Claude["Claude Code"]
        Custom["Custom MCP Client"]
    end

    subgraph BTP["SAP BTP"]
        subgraph MCPLayer["MCP Server<br/>(Python + FastMCP)"]
            MCPTools["MCP Tools<br/>(wraps OData/REST)"]
            AuditLog["Audit Logger"]
        end

        subgraph CAPApp["CAP Application<br/>(Node.js or Java)"]
            CDS["CDS Models<br/>(Entity Definitions)"]
            Service["CDS Services<br/>(OData V4 / REST)"]
            Logic["Custom Handlers<br/>(Business Logic)"]
            Auth["CAP Auth<br/>(XSUAA / JWT)"]
        end

        subgraph HANA["SAP HANA Cloud"]
            Tables["CDS-Generated Tables<br/>(employee, skill, evidence,<br/>attrition)"]
            CalcViews["Calculation Views<br/>(Aggregations, Hierarchies)"]
            Prediction["PAL / APL<br/>(Predictive Analysis Library)"]
        end

        XSUAA["XSUAA<br/>(Auth Service)"]
    end

    Clients -->|"MCP (Streamable HTTP)"| MCPTools
    MCPTools -->|"OData V4 / REST"| Service
    Service --> Auth
    Auth -->|"JWT validation"| XSUAA
    Service --> Logic
    Logic --> CDS
    CDS --> Tables
    CDS --> CalcViews
    CalcViews --> Prediction
```

## CDS Model Example

```mermaid
graph LR
    subgraph "CDS Domain Model"
        Employee["entity Employee {<br/>  key ID: String(9);<br/>  name: String;<br/>  org: Association to OrgUnit;<br/>  skills: Composition of many<br/>    EmployeeSkill;<br/>}"]

        Skill["entity Skill {<br/>  key ID: Integer;<br/>  name: String;<br/>  category: SkillCategory;<br/>}"]

        EmpSkill["entity EmployeeSkill {<br/>  key employee: Association to Employee;<br/>  key skill: Association to Skill;<br/>  proficiency: Integer;<br/>  confidence: Integer;<br/>  evidence: Composition of many<br/>    SkillEvidence;<br/>}"]

        AttrRisk["entity AttritionRisk {<br/>  key employee: Association to Employee;<br/>  probability: Decimal;<br/>  riskLevel: RiskLevel;<br/>  factors: Composition of many<br/>    RiskFactor;<br/>}"]

        Employee --> EmpSkill
        Skill --> EmpSkill
        Employee --> AttrRisk
    end
```

## Key Differences from Current Architecture

| Aspect | Current (Solution 1) | HANA + CAP + MCP |
|--------|---------------------|------------------|
| **Database** | External PostgreSQL | HANA Cloud (SAP-managed) |
| **API framework** | FastAPI (custom Python) | CAP (CDS-defined services) |
| **Data model** | SQL DDL scripts | CDS model → auto-generated DDL |
| **OData support** | None (pure REST) | Native OData V4 |
| **Authentication** | API key header | XSUAA + JWT tokens |
| **Attrition ML** | External API logic | HANA PAL/APL (in-database ML) |
| **MCP layer** | Same (FastMCP wrapper) | Same (FastMCP wrapper) |

## Attrition Prediction with HANA PAL

HANA Cloud's Predictive Analysis Library (PAL) can run attrition models **inside the database**, eliminating the need for external ML services:

```mermaid
graph LR
    subgraph HANA["HANA Cloud"]
        TrainData["Training Data<br/>(historical attrition)"]
        PAL["PAL: Random Decision Trees<br/>or Gradient Boosting"]
        Model["Trained Model<br/>(stored in HANA)"]
        Predict["PAL: PREDICT<br/>(real-time scoring)"]
        Results["Attrition Predictions<br/>(probability + factors)"]

        TrainData --> PAL --> Model
        Model --> Predict --> Results
    end

    CAPService["CAP Service<br/>(CDS View on Results)"] --> Results
```

## Pros

- **SAP-managed database** — HANA Cloud handles backups, scaling, HA
- **CDS modeling** — Declarative data model, auto-generated APIs, type safety
- **In-database ML** — HANA PAL for attrition prediction without external services
- **Enterprise auth** — XSUAA integration, role-based access, JWT tokens
- **MCP compatibility** — Still supports any MCP client (not locked to Joule)
- **OData V4** — Standard protocol, usable by SAP Fiori, Analytics Cloud, etc.
- **Migration path** — Can evolve toward Datasphere views later

## Cons

- **HANA Cloud cost** — Significant licensing (minimum ~$500/month)
- **CAP learning curve** — CDS syntax, service definitions, custom handlers
- **Two runtimes** — CAP app (Node.js/Java) + MCP server (Python)
- **More infrastructure** — HANA Cloud instance, XSUAA service, CAP deployment
- **Heavier deployment** — MTA (Multi-Target Application) builds
- **Over-engineered for small datasets** — HANA is designed for large-scale analytics

## When to Use This

- Your organization is committed to SAP BTP and HANA Cloud
- You need in-database ML for attrition prediction (HANA PAL)
- Enterprise-grade authentication (XSUAA) is required
- You want CDS-modeled data that can also serve Fiori apps and SAC dashboards
- You need MCP flexibility (not just Joule) but also SAP service integration
