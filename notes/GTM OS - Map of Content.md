---
tags: [moc, gtm-os]
status: growing
created: 2026-10-15
updated: 2026-10-28
aliases: [GTM OS MOC, Home]
---

# GTM OS — Map of Content

Research notes for designing a company "OS": a scaled system of workers that ingest, enrich and act on go-to-market data, with a unified dashboard on top. Each note holds one concept; follow the links.

**Start here:** [[System Map]] (the six layers and where each note sits) and [[Recurring Lessons]] (the one-page brief).

## Foundations — the data layer
- [[Data Integration]]: naming the core problem; ETL vs ELT
- [[Integration Approaches]]: query live vs CRM hub vs central warehouse
- [[Choosing the Store]]: Airtable vs Supabase vs Snowflake
- [[OLTP vs OLAP]]: row vs column storage, and how to pick an engine

## The core data model (layer 2)
- [[Core Data Model]]: the core nouns; your model is the master; IDs and the crosswalk
- [[Temporal Modelling]]: relationships with start and end dates; lifecycle vs deal stages
- [[Events vs State]]: the event log is the truth, state tables are shortcuts; reconciliation
- [[Raw to Core Layers]]: raw → cleaned → core (medallion); keep the original
- [[Survivorship]]: designed rules for conflicting sources; the golden record
- [[Account Hierarchy]]: parents and subsidiaries; rollups; dated parent links
- [[Core Plus Extensions]]: plugins get their own tables, never columns on the core
- [[Data Model Management]]: eight practices for keeping the model healthy

## Getting data in
- [[Ingestion Models]]: push vs pull, webhooks, polling, hybrid
- [[Per-Source Decision Framework]]: freshness, criticality, the vendor's integration surface
- [[Ingestion Layer Options]]: managed connectors, self-hosted n8n, code-first orchestrators, fully custom
- [[Trigger.dev]]: the recommended orchestrator, in depth
- [[Webhook Ingress]]: receiver vs processor; the inbox pattern, store-then-process
- [[Source Routing]]: many sources, one front door; per-source config; server vs public sources
- [[Pull-Based Sync]]: cursors, pagination, rate limits, invisible deletes
- [[Bot Protection]]: honeypots and defence in depth
- [[Idempotency]]: twice equals once; upserts, idempotency keys, check-before-act

## Making sense of it
- [[Identity Resolution]]: one canonical ID per human; matching, thresholds, review queues
- [[Person vs Account]]: the B2B hierarchy and slowly changing dimensions

## System design
- [[System Design Principles]]: from organic growth to five deliberate decisions
- [[Worker Coordination]]: contracts, orchestration vs choreography, AI agents
- [[Engine vs Core]]: build *on* a tool, never *around* it

## Observability
- [[Observability]]: logs, metrics, traces, what good logging looks like, tiers of tooling
- [[Two Log Streams]]: technical logs vs business events, split by audience; the four v0 events
- [[Correlation ID]]: tracing without a tracing tool; the core mints it, not the engine
- [[Telemetry Module]]: the small shared logging component every worker calls; what v0 covers and where it stops
- [[OpenTelemetry]]: the vendor-neutral standard that keeps backends swappable

## Builds
- [[Form Intake Pipeline]]: first real build, **live (v0)**: website form → receiver → spam decision → email + business events; lessons from the first real submission
- [[Form Submission Contract]]: the **verified** webhook payload, the plugin's own signing and retries, PHP quirks, plus the CloudEvents and Standard Webhooks standards

## Running it (layer 6)
- [[Engineering Basics]]: finish what you shipped, CI/CD, environments, secrets, limiting work in progress

## Open
- [[Open Threads]]
- [[Self-Exam]]: how understanding gets tested, in your own words

## Current position (Oct 28, 2026)
- Central store: **Supabase (Postgres)**.
- Architecture decided: shared data model, event log, explicit worker contracts, logic in owned code ([[Engine vs Core]]).
- Ingestion engine: **Trigger.dev runs the first production worker**, kept replaceable by [[Engine vs Core]]. The fair comparison with Inngest, Temporal, self-hosted n8n and fully custom is still open.
- The dashboard reads only from Supabase, never from vendor APIs.
- **Learning focus:** this thread is for architecture; build tickets are a side job. Working through the [[System Map]]: layer 2 ([[Core Data Model]]) is **done**, including [[Account Hierarchy]] and [[Core Plus Extensions]]; **next is layer 1, connectors**.
- Observability: two streams (technical logs, business events), one core-minted correlation ID, a shared telemetry module, and a Supabase event table with four stable v0 events ([[Two Log Streams#The four v0 events]]). Failures email an alert. Gaps: the receiver's logging, and an alert channel independent of the mail path.
- First build: [[Form Intake Pipeline]], **built (v0)** in `galliot-us/GTM-Engine` ([issue #1](https://github.com/galliot-us/GTM-Engine/issues/1)). The plugin **already signs** requests with its own HMAC scheme; the receiver doesn't verify it yet, and which scheme to standardise on is open ([[Form Submission Contract]]).
- Next: a second source, Life Science Tech Collective, through the same front door; v1 scope is **store + notify** only ([[Source Routing]], [GTM-Engine #3](https://github.com/galliot-us/GTM-Engine/issues/3)).
