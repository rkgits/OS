---
tags: [moc, gtm-os]
status: growing
created: 2026-10-15
updated: 2026-10-18
aliases: [GTM OS MOC, Home]
---

# GTM OS — Map of Content

Research notes for designing a company "OS": a scaled system of workers that ingest, enrich and act on go-to-market data, with a unified dashboard on top. Each note holds one concept; follow the links.

## Foundations — the data layer
- [[Data Integration]]: naming the core problem; ETL vs ELT
- [[Integration Approaches]]: query live vs CRM hub vs central warehouse
- [[Choosing the Store]]: Airtable vs Supabase vs Snowflake
- [[OLTP vs OLAP]]: row vs column storage, and how to pick an engine

## Getting data in
- [[Ingestion Models]]: push vs pull, webhooks, polling, hybrid
- [[Per-Source Decision Framework]]: freshness, criticality, the vendor's integration surface
- [[Ingestion Layer Options]]: managed connectors, self-hosted n8n, code-first orchestrators, fully custom
- [[Trigger.dev]]: the recommended orchestrator, in depth

## Making sense of it
- [[Identity Resolution]]: one canonical ID per human; matching, thresholds, review queues
- [[Person vs Account]]: the B2B hierarchy and slowly changing dimensions

## System design
- [[System Design Principles]]: from organic growth to five deliberate decisions
- [[Worker Coordination]]: contracts, orchestration vs choreography, AI agents
- [[Engine vs Core]]: build *on* a tool, never *around* it

## Open
- [[Open Threads]]

## Current position (Oct 18, 2026)
- Central store: **Supabase (Postgres)**.
- Ingestion: hybrid — Airbyte where a connector exists, **Trigger.dev** for the rest and for enrichment. Pilot: one Smart Lead connector.
- The dashboard reads only from Supabase, never from vendor APIs.
