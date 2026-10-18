---
tags: [gtm-os, architecture, principles]
status: growing
created: 2026-10-18
updated: 2026-10-18
aliases: [build on not around, ports and adapters, hexagonal architecture]
---

# Engine vs Core

## Correction
You *do* choose one engine (n8n, [[Trigger.dev]], ...). The error is **building around** it instead of **building on** it.

- **Building around it**: the tool becomes your architecture. Business logic lives inside its nodes, formats and way of thinking. That's the n8n situation today: the spam rules, scoring and enrichment exist only as n8n workflows, so leaving means rebuilding everything. This is **vendor lock-in**, and it's why the pricing traps you.
- **Building on it**: the engine is **infrastructure**, a replaceable part. Your architecture is what you own: the data model, the event log, business rules as plain functions, and the contracts between workers. The engine only schedules, triggers, retries, queues and logs.

## The test
> If I swapped Trigger.dev for Inngest next year, what would I rewrite?

- "A thin layer of task wrappers" → sound architecture.
- "Everything" → the tool has become your architecture.

The formal names are **ports and adapters** (also called **hexagonal architecture**) and **dependency inversion**: core logic never depends on the tool; the tool depends on the core.

## Concretely
The Smart Lead extraction is a plain function in your codebase, `syncSmartLeadCampaigns()`. The Trigger.dev task is about five lines that call it on a schedule.

Related: [[System Design Principles]] (decision 4), [[Ingestion Layer Options]].
