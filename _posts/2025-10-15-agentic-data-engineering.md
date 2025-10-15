---
layout: default
title: "Agentic Data Engineering"
description: "How we implemented an AI coding agent to accelerate data pipeline delivery"
date: 2025-10-15
categories: data-engineering agentic ai
---

*Disclaimer: The views expressed here are my own and do not necessarily reflect those of Microsoft.*

*Project team: [Alex Peles](https://www.linkedin.com/in/alex-peles-2237313/), [Ariel Haim](https://www.linkedin.com/in/ariel-haim/), [Avihu Elbaz](https://www.linkedin.com/in/avihuelbaz/).*

## Implementation Context (Why It Matters)
We are a data engineering team inside one of Microsoft’s internal R&D organizations. We build and evolve an internal analytical Data Warehouse serving engineering, product, and operational stakeholders. Our stack: Azure Data Factory for orchestration, Microsoft Fabric for unified lakehouse + warehouse + semantic + BI, dimensional (star) modeling layered on a Medallion-style refinement path, and full source control + CI/CD in Azure DevOps.  
Why mention this?  
- We operate in an enterprise environment with compliance, reliability, and velocity pressure.  
- Our workflow resembles that of many modern data platform teams—whether your storage/compute layer is Snowflake, BigQuery, or Databricks, and whether you orchestrate and transform with Airflow and dbt (or similar tools).  
- Showing this context up front helps you map our automation pattern to your ecosystem.  
This article is not about “Microsoft-only magic”—it is about a replicable agentic automation blueprint.  The pattern is not Cline-specific; any agent environment exposing auditable, step‑gated execution (e.g., Cursor or Roo) can map similar conventions.  

A single focused internal Microsoft Global Hackathon week gave us the protected window to pause routine delivery and prototype agentic automation safely. We treated that week as an accelerated discovery + architecture sprint—versioning prompts, codifying conventions, and building MCP capability surfaces—not a throwaway experiment. What you read below is the hardened evolution of that prototype. We deliberately constrained scope to patterns any convention‑mature data platform team can replicate with analogous tooling.

## TL;DR
In one week (Hackathon) we wired an AI coding agent (Cline) plus custom MCP servers to automate the repetitive middle of our data product development lifecycle: source data profiling, dimensional model proposal (fact/dim draft from requirements + observed source data), automated delivery branch creation in all component repos, table DDL creation, ingestion pipeline authoring + execution + validation, and internal wiki documentation. Result: materially faster iteration, more human focus on semantic modeling & analytics quality, and a pattern any modern data engineering team can adapt—regardless of whether you run Microsoft Fabric + ADF or another stack.

**Executive Snapshot**
- Scope: Automated the convention-rich middle (profiling → model draft → branching → DDL → pipelines → validation → docs)
- Stack: Azure Data Factory, Microsoft Fabric (Lakehouse/Warehouse), Git (Azure DevOps), Cline (agent), custom MCP servers (Kusto / ADF Debug / Fabric)
- Productivity: ~40–60% reduction in the convention-rich middle (profiling → model draft setup → branching → DDL → pipelines → docs; Steps 3–7 & 10 below) after initial ramp
- Governance: Human gates retained at requirements, model approval, and BI/semantic validation
- Safety: Conventions + constrained MCP capability surfaces (no broad shell / unrestricted queries)
- Theme: Agentic automation amplifies data engineering productivity by codifying standards, not bypassing them

<div class="exec-flow">
  <div class="exec-flow-text">
    <p><strong>Automated Middle (Steps 3–7 & 10):</strong> The agent accelerates profiling, model draft proposal, multi‑repo branching, DDL/object generation, pipeline authoring & validation, and documentation while preserving human semantic gates at requirements, model approval, and BI validation.</p>
    <ul>
      <li><strong>Human Gates:</strong> Requirements, Model Approval, BI/Semantic Validation</li>
      <li><strong>Agent Cycle:</strong> Profiling → Draft Model → Branching → DDL/Pipelines → Validation (retry loop) → Docs</li>
      <li><strong>Safety:</strong> Convention-enforced prompts + constrained MCP capability surfaces</li>
    </ul>
  </div>
  <figure class="exec-flow-figure">
    <img src="{{ "/assets/images/agentic/diagram1.png" | relative_url }}" alt="Automated middle bracketed by human gates: profiling, model draft, branching, DDL/pipelines, validation, docs" />
    <figcaption>Figure 1. Automated middle bracketed by human gates.</figcaption>
  </figure>
</div>

---

## Problem: Friction in Incremental Data Warehouse Delivery
A typical data product increment (new dataset or enhancement) required a chain of labor‑intensive steps:

1. Clarify business requirements & expected outcomes  
2. Inventory / access data sources  
3. Profile each source & land raw (bronze) data  
4. Design dimensional (star schema) model (facts & dimensions)  
5. Create delivery branches across multiple repos (ADF, Fabric, Wiki, etc.)  
6. Create physical objects (tables, SQL procedures, notebooks)  
7. Build & test ingestion / transformation pipelines  
8. Build & test semantic model + dashboards  
9. Open & approve PRs (CI/CD handles deploy)  
10. Document datasets internally  

Steps 3–7 & 10 are high-frequency, pattern-rich, and rule-constrained—strong candidates for agentic automation while keeping humans in the loop for semantic intent (1, 4, 8).

---

## Guiding Principles
- Automate the “boring middle,” not judgment-heavy design thinking.  
- Encode conventions (naming, branching, foldering, assertions) so the agent acts safely.  
- Modular workflows: run full orchestration or targeted sub-flows.  
- Favor reversible implementation choices; optimize after proving value.  
- Mandatory human approval at semantic model checkpoints.  

---

## End‑to‑End Data + Agentic Stack  
These are the platform components and integration surfaces the agent uses to automate deterministic development steps.
- Orchestration: Azure Data Factory (ADF)  
- Unified Analytics & BI: Microsoft Fabric (Lakehouse, Warehouse, Notebooks, Direct Lake semantic layer, Power BI)  
- Modeling Strategy: Medallion (Bronze → Silver → Gold) + Dimensional (Star Schema)  
- Repos & CI/CD: Azure DevOps Git + Pipelines  
- AI Runtime: Cline (workflow + rules engine)  
- Custom MCP Servers:  
  - Kusto MCP (executes agent‑generated KQL queries for schema introspection, sampling, and profiling metrics)  
  - ADF Debug MCP (launch & monitor pipeline debug runs)  
  - Fabric MCP (DDL creation, metadata ops, and query execution—agent validates loaded data via direct Warehouse/Lakehouse queries)  
- Documentation: Markdown dataset pages committed to a dedicated Wiki Git repository (no MCP; agent writes/updates files and commits them like code)  

Fabric note: Microsoft Fabric unifies ingestion, storage, processing, notebooks, semantic modeling, and reporting in one managed platform—eliminating much of the custom scripting otherwise needed between discrete services and making agent orchestration simpler.  
Official Fabric MCP: reference implementation you can use instead of building a custom Fabric MCP (https://blog.fabric.microsoft.com/en-US/blog/introducing-fabric-mcp-public-preview/)

---

## What We Automated (First Iteration)

| Step | Status | Mechanism |
|------|--------|-----------|
| 3 Source data profiling & bronze landing | Automated | Kusto MCP (executes agent‑generated KQL) + generated ADF pipeline |
| 4 Dimensional model draft | Semi‑automated (human gate) | Model workflow proposes → human edits |
| 5 Multi‑repo branch creation | Automated | Structured folder layout + Git CLI |
| 6 Table / object creation | Automated | Fabric MCP generates DDL / notebooks |
| 7 Pipeline build, execution, validation | Automated w/ retry loop | ADF JSON authoring + debug MCP + assertions |
| 10 Documentation | Automated | Markdown page committed to Wiki repo (agent‑generated) |

Deferred for next iteration: broader multi-source profiling (beyond initial Kusto focus—relational, API, file-based sources), deeper source access automation (#2 expansion), semantic model & BI deployment (#8), PR orchestration (#9).

---

## Architecture Overview
The whole process is wrapped by a master Cline workflow object, providing structure, state passing, deterministic ordering, and safe execution boundaries.

- Master Workflow: Orchestrates end‑to‑end data product development; passes structured metadata (profiling outputs, model spec, branch names).  
- Sub-Workflows: Source Profiling, Model Drafting, Object Provisioning, Pipeline Authoring & Validation, Documentation Publishing.  
- MCP Servers: Typed capability surfaces bridging the agent and external systems (cluster, pipelines, warehouse, wiki).  

![Architecture topology]({{ "/assets/images/agentic/diagram2.png" | relative_url }})

Figure 2. Master workflow orchestrates specialized sub‑workflows; MCP servers expose constrained capability surfaces (Kusto queries, ADF debug runs, Fabric DDL/query) against external systems, with Git providing the artifact backbone.

Why Cline Workflows? Determinism, replayability, auditability, modular iteration. (Similar structured, step‑gated agent flows presumably exist in other environments like Cursor or Roo; the pattern—not the specific runtime—is the replicable element.)

---

## Deep Dive by Step

### Step 3: Source Data Profiling & Bronze Landing
Kusto MCP: executes agent‑generated KQL queries (table scans plus targeted aggregations) to return raw schema metadata, null ratios, distinct counts, distribution samples, and anomaly indicators. The workflow composes and interprets these results; the MCP only runs queries. Initial profiling scope targets Kusto sources; subsequent iterations extend the same agent pattern to additional internal relational, API, and file-based sources.
- Input: source table name or KQL query.  
- Output: schema, inferred types, null ratios, cardinalities, categorical distributions, potential key candidates, anomaly markers.  
Bronze landing:
- Agent synthesizes ADF pipeline JSON (pattern-driven).  
- Creates & pushes delivery branch (push required so ADF debug run references the branch’s JSON artifacts)  
- Debug run via ADF MCP (not a full publish).  
- Validates raw landing (row arrival, schema conformity).  

### Step 4: Dimensional Model Draft
Inputs: profiling metadata + business context (prompted if absent).  
Agent proposes: fact tables (business grain, surrogate vs natural keys), dimension tables (attributes, SCD hints), key strategy.  
- Looks up existing conformed dimensions and surrogate keys to recommend foreign key columns and avoid duplicating shared entities.  
Human adjusts / approves.  
Artifacts: Generated DDL (fact & dimension CREATE / incremental merge statements) committed directly to the repo; model rationale embedded as inline comments.

Before drafting new facts and dimensions, the workflow queries existing warehouse metadata (conformed dimension inventory, surrogate key columns, established grains, prior fact → dimension relationships). This context enables reuse of shared dimensions, proper foreign key naming, and grain alignment while preventing redundant dimensional structures.

### Step 5: Branch Creation (Multi-Repo)
Folder strategy (Appendix) enables relative Git commands to create synchronized branches in ADF, Fabric, Wiki repos; consistent naming fosters traceability across artifacts.

### Step 6: Physical Object Creation
Fabric MCP executes CREATE TABLE statements for facts & dimensions; conditionally generates SQL stored procedures (incremental merge patterns) and Spark notebooks (complex transforms) with standardized naming, logging, parameterization. Incremental merge templates enforce idempotent loads (surrogate key handling, late-arriving dimension rows, update detection) so repeated agent executions are safe.

Example fact table DDL ( abridged ):
```sql
CREATE TABLE dw.fact_security_event (
    event_sk            BIGINT                    NOT NULL,        -- Surrogate key
    event_id_nk         VARCHAR(64)               NOT NULL,        -- Natural/business key
    event_ts_utc        DATETIME2(6)              NOT NULL,        -- Grain timestamp
    device_sk           BIGINT                    NOT NULL,        -- FK to dim_device
    user_sk             BIGINT                    NULL,            -- FK to dim_user (nullable if anonymous)
    severity_code       INT                       NOT NULL,        -- Categorical severity
    created_utc         DATETIME2(6),
    updated_utc         DATETIME2(6)
);
-- Grain: one row per (event_id_nk)
-- Foreign keys resolved post-load via conformed dimensions
```

### Step 7: Pipeline Authoring, Execution & Validation
1. Pattern selection (standard parameterized pipeline blueprint enforcing structural conventions).  
2. JSON Generation (activities, datasets, linked services).  
3. Commit & push under feature branch.  
4. Debug run via ADF MCP; collect activity metrics + errors.  
5. Validation Assertions (row thresholds, not-null keys, basic referential sanity) using Fabric MCP query execution to fetch live row counts and sample records.  
6. Failure Path: targeted fix + bounded exponential retry.  
7. Success: validation summary persisted (metrics + assertions).  

Validation assertions structure (illustrative):
```json
{
  "table": "dw.fact_security_event",
  "checks": [
    { "type": "row_count_min", "threshold": 1000, "observed": 1324 },
    { "type": "not_null", "column": "event_ts_utc", "violations": 0 },
    { "type": "not_null", "column": "event_id_nk", "violations": 0 },
    { "type": "fk_reference_sample", "column": "device_sk", "sample_invalid": 0 }
  ],
  "status": "pass",
  "execution_ts_utc": "2025-10-13T16:05:00Z"
}
```

### Step 10: Documentation Generation
Documentation step: agent generates a Markdown page (purpose, lineage summary, table inventory (facts/dims), column catalog (type, nullability, key role)) and commits it into the Wiki Git repository (same branching model). 

---

## Appendix: Multi-Repo Layout for Agent Automation
This layout provides deterministic relative paths for the agent while isolating histories per platform component.
Goal: Enable the agent to perform cross-repo branching and commits via stable relative paths without nesting Git histories.

```
root-cline-repo/
  workflows/
  mcp-servers/
  engineering-standards/
  (ignored) adf-repo/
  (ignored) fabric-repo/
  (ignored) wiki-repo/
```

Mechanics:
- Setup script clones subordinate repos into deterministic subfolders.  
- Root `.gitignore` excludes component repos (avoids nested history noise).  
- Agent runs relative commands (`cd adf-repo && git checkout -b feature/...`).  

Benefits: deterministic paths, rapid onboarding, ownership separation, uniform traceability.

## What Worked
- Deterministic acceleration of repetitive middle steps  
- Reduced cognitive switching (branching / scaffolding delegated)  
- Higher modeling quality focus (grain correctness, conformed dimensions)  
- Modular adoption (re-run specific sub-workflows as needed)  

---

## Impact (Early Signals)
<div class="table-scroll">
<table>
  <thead>
    <tr>
      <th>Phase (Steps)</th>
      <th>Before (hrs)</th>
      <th>After (hrs)</th>
      <th>Delta %</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Source Profiling & Bronze (3)</td><td>2.0</td><td>0.9</td><td>-55%</td><td>Automated profiling queries + auto-generated pipeline</td></tr>
    <tr><td>Model Draft Prep (4 pre‑gate)</td><td>1.5</td><td>0.8</td><td>-47%</td><td>Agent proposal + human edits</td></tr>
    <tr><td>Branching / Object Scaffolding (5–6)</td><td>1.2</td><td>0.4</td><td>-67%</td><td>Multi‑repo creation + DDL generation</td></tr>
    <tr><td>Pipeline Build + Validation (7)</td><td>3.0</td><td>1.5</td><td>-50%</td><td>Parameterized pipeline pattern + automated validation assertions</td></tr>
    <tr><td>Documentation (10)</td><td>1.0</td><td>0.3</td><td>-70%</td><td>Generated Markdown</td></tr>
    <tr><td><strong>Total Targeted Slice</strong></td><td><strong>8.7</strong></td><td><strong>3.9</strong></td><td><strong>-55%</strong></td><td>Variation by complexity</td></tr>
  </tbody>
</table>
</div>
<p><em>Qualitative:</em> Higher semantic focus, reduced context switching, more consistent structural patterns.</p>

## Risks & Mitigations
<div class="table-scroll">
<table>
  <thead>
    <tr><th>Risk</th><th>Mitigation</th></tr>
  </thead>
  <tbody>
    <tr><td>Over-automation of semantics</td><td>Mandatory model approval gate</td></tr>
    <tr><td>Credential sprawl</td><td>Central secrets + audited MCP boundaries</td></tr>
    <tr><td>Debug opacity</td><td>Structured run logs provided by Cline</td></tr>
    <tr><td>Prompt regression</td><td>Versioned workflow definitions</td></tr>
  </tbody>
</table>
</div>

---

## Roadmap
- Semantic model + Power BI artifact automation  
- Data dictionary & glossary synthesis  
- Policy-driven assertion generation  

## Closing Thought
Agentic acceleration in data engineering is won by turning dimensional modeling, ingestion patterns, and validation rules into enforceable conventions—once codified, an AI workflow can safely own profiling, scaffolding, and structural QA while humans stay focused on business grain and conformed dimensions.
