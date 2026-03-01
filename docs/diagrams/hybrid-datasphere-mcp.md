# Solution 6: Hybrid — Datasphere + MCP Server

> **Use SAP Datasphere for enterprise-grade data curation and governance, but expose the curated data through an MCP server for broad AI agent compatibility.** Best of both worlds: SAP governance + open AI access.

## Architecture

```mermaid
graph TB
    subgraph Sources["Data Sources"]
        SF["SuccessFactors"]
        ExtDB[("External DB<br/>(PostgreSQL)")]
        HRIS["Third-party HRIS"]
        CSV["Manual Uploads"]
    end

    subgraph BTP["SAP BTP"]
        subgraph DSP["SAP Datasphere"]
            Ingest["Data Integration<br/>(Replication Flows)"]
            Raw["Raw / Staging Layer"]
            Curate["Curation Layer<br/>- Skill normalization<br/>- Proficiency standardization<br/>- Evidence quality scoring<br/>- Attrition factor calculation"]
            BizModel["Business Model Layer<br/>(Dimensions, Facts,<br/>Analytic Models)"]
            ConsViews["Consumption Views<br/>(Exposed via SQL)"]
        end

        subgraph APILayer["API Layer"]
            CAPService["CAP Service or<br/>FastAPI Wrapper"]
            APIEndpoints["REST Endpoints<br/>(reads from Datasphere)"]
        end

        subgraph MCP["MCP Server<br/>(Python + FastMCP)"]
            MCPTools["21+ MCP Tools"]
            Prompts["Prompt Templates"]
            AuditLog["Audit Logger"]
        end

        subgraph JouleDirect["Joule Agent<br/>(Direct Datasphere Access)"]
            JouleGround["Grounded on<br/>Datasphere Views"]
        end
    end

    subgraph Clients["AI Clients"]
        Claude["Claude Code /<br/>Claude Desktop"]
        Joule["Joule Studio"]
        Custom["Custom MCP Agents"]
        JouleUsers["Joule Users<br/>(Direct)"]
    end

    SF -->|"OData"| Ingest
    ExtDB -->|"Open SQL"| Ingest
    HRIS -->|"API"| Ingest
    CSV -->|"Upload"| Ingest

    Ingest --> Raw --> Curate --> BizModel --> ConsViews

    ConsViews -->|"SQL Access"| CAPService
    CAPService --> APIEndpoints
    APIEndpoints --> MCPTools
    MCPTools --> AuditLog

    ConsViews -->|"Native"| JouleGround

    Claude -->|"MCP Protocol"| MCPTools
    Custom -->|"MCP Protocol"| MCPTools
    Joule -->|"MCP Protocol"| MCPTools
    JouleUsers --> JouleGround
```

## The Hybrid Advantage

```mermaid
graph LR
    subgraph "Data Governance (Datasphere)"
        Gov["Lineage ✓<br/>Cataloging ✓<br/>Quality ✓<br/>Security ✓<br/>Auditing ✓"]
    end

    subgraph "AI Flexibility (MCP)"
        Flex["Any MCP client ✓<br/>Claude ✓<br/>Joule ✓<br/>Custom agents ✓<br/>Open standard ✓"]
    end

    subgraph "Direct Access (Joule)"
        Direct["Joule grounding ✓<br/>No API needed ✓<br/>Native integration ✓"]
    end

    Gov --> Flex
    Gov --> Direct
```

## Datasphere Curation Pipeline

```mermaid
graph TB
    subgraph "Layer 1: Raw / Staging"
        RawEmp["raw.employee_import"]
        RawSkill["raw.skill_import"]
        RawEvidence["raw.evidence_import"]
    end

    subgraph "Layer 2: Curation"
        CurEmp["curated.employee<br/>- Standardized names<br/>- Validated org hierarchy"]
        CurSkill["curated.skill<br/>- Normalized taxonomy<br/>- Merged duplicates"]
        CurEvidence["curated.evidence<br/>- Quality scored<br/>- Freshness flagged"]
        CurAttrition["curated.attrition_risk<br/>- Factor computation<br/>- Probability scoring"]
    end

    subgraph "Layer 3: Business Model"
        DimEmp["Dimension: Employee"]
        DimSkill["Dimension: Skill"]
        DimOrg["Dimension: Org Unit"]
        FactProf["Fact: Proficiency"]
        FactEvidence["Fact: Evidence"]
        FactRisk["Fact: Attrition Risk"]
    end

    subgraph "Layer 4: Consumption"
        ViewSkillProfile["View: Employee Skill Profile"]
        ViewExperts["View: Skill Experts"]
        ViewAttrition["View: Attrition Dashboard"]
        ViewOrgSummary["View: Org Talent Summary"]
    end

    RawEmp --> CurEmp --> DimEmp
    RawSkill --> CurSkill --> DimSkill
    RawEvidence --> CurEvidence --> FactEvidence
    CurEmp --> CurAttrition --> FactRisk

    DimEmp --> ViewSkillProfile
    DimSkill --> ViewSkillProfile
    FactProf --> ViewSkillProfile

    DimSkill --> ViewExperts
    FactProf --> ViewExperts

    FactRisk --> ViewAttrition
    DimEmp --> ViewAttrition

    DimOrg --> ViewOrgSummary
    FactProf --> ViewOrgSummary
```

## How This Compares to Pure Approaches

| Aspect | Solution 1 (Current) | Solution 2 (DS + Joule) | Solution 6 (Hybrid) |
|--------|---------------------|------------------------|---------------------|
| Data governance | None | Full | Full |
| AI agent support | Any MCP | Joule only | Any MCP + Joule |
| Data curation | Custom code | Datasphere | Datasphere |
| Implementation effort | Low | Medium | Medium-High |
| Operational cost | Low | Medium | Medium-High |
| Vendor flexibility | Full | None (SAP only) | High |

## Migration from Current Architecture

```mermaid
graph TB
    subgraph "Phase 1: Add Datasphere"
        Step1["Replicate PostgreSQL data<br/>into Datasphere"]
        Step2["Build curation views<br/>in Datasphere"]
        Step3["Validate curated data<br/>matches current API output"]
    end

    subgraph "Phase 2: Redirect MCP Server"
        Step4["Build API layer reading<br/>from Datasphere views"]
        Step5["Point MCP server at<br/>new API layer"]
        Step6["Verify all 21 tools<br/>return identical results"]
    end

    subgraph "Phase 3: Extend"
        Step7["Add Joule direct access<br/>to Datasphere views"]
        Step8["Add new data sources<br/>via Datasphere integration"]
        Step9["Retire external PostgreSQL"]
    end

    Step1 --> Step2 --> Step3 --> Step4 --> Step5 --> Step6 --> Step7 --> Step8 --> Step9
```

## Pros

- **Enterprise governance + open AI access** — The key differentiator
- **Not locked to one AI vendor** — Claude, Joule, and custom agents all work
- **Datasphere curation** — Professional data modeling replaces custom Python scripts
- **Incremental migration** — Move from current architecture gradually
- **Dual consumption** — Joule gets direct Datasphere access; other agents get MCP
- **Future-proof** — As MCP ecosystem grows, your data is already accessible

## Cons

- **Most infrastructure** — Datasphere + API layer + MCP server
- **Higher cost** — Datasphere licensing + CF compute for API + MCP
- **Complexity** — Three layers to maintain (Datasphere, API, MCP)
- **Datasphere learning curve** — Still need Datasphere modeling expertise
- **Latency** — Additional hop: Datasphere → API → MCP (vs. direct DB access)

## When to Use This

- You need both enterprise governance AND multi-vendor AI access
- You want Joule for SAP users but also Claude/custom agents for developers
- Your data curation needs have outgrown custom code
- You plan to consolidate multiple HR data sources over time
- Incremental migration from the current architecture is preferred over a big-bang switch
