---
tags: [moc, gtm-os]
status: growing
created: 2026-10-15
updated: 2026-10-19
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

## Observability
- [[Observability]]: logs, metrics, traces, what good logging looks like, tiers of tooling
- [[Two Log Streams]]: technical logs vs business events, split by audience
- [[Correlation ID]]: tracing without a tracing tool; the core mints it, not the engine
- [[Telemetry Module]]: the small shared logging component every worker calls
- [[OpenTelemetry]]: the vendor-neutral standard that keeps backends swappable

## Builds
- [[Form Intake Pipeline]]: first real build, website form → receiver → spam decision → business events
- [[Form Submission Contract]]: the webhook payload, plus the CloudEvents and Standard Webhooks standards

## Open
- [[Open Threads]]

## Current position (Oct 19, 2026)
- Central store: **Supabase (Postgres)**.
- Architecture decided: shared data model, event log, explicit worker contracts, logic in owned code ([[Engine vs Core]]).
- Ingestion engine **not decided**: a code-first orchestrator is the default; candidates are Trigger.dev, Inngest, Temporal, self-hosted n8n, fully custom.
- The dashboard reads only from Supabase, never from vendor APIs.
- Observability: two streams (technical logs, business events), one core-minted correlation ID, a shared telemetry module; start with the orchestrator's built-in logs + Slack alerts + a Supabase event table.
- First build: [[Form Intake Pipeline]] for galliot.us forms. A free Trigger.dev account exists; the webhook isn't configured yet. Contract standards: CloudEvents + Standard Webhooks.
