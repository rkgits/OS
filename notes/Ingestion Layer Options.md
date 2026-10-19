---
tags: [gtm-os, ingestion, decision]
status: growing
created: 2026-10-18
updated: 2026-10-19
---

# Ingestion Layer Options

## Two corrections
1. **The dashboard must not call vendor APIs.** That is Approach A again ([[Integration Approaches]]). The principle is **separation of concerns**: the dashboard reads only from Supabase, and a separate **ingestion layer** fills Supabase.
2. **The old n8n flows mix two different jobs:**
   - **Extraction**: copying vendor records into the database. Dumb, repetitive, high-volume.
   - **Enrichment orchestration**: multi-step reasoning about one record (sign-up → spam check → domain research → LinkedIn lookup → write). Low volume, many steps.
   Paying per execution for high-volume extraction is part of why the n8n bill hurts.

## The four options
1. **Managed ELT connectors** (Airbyte, Fivetran): ready-made pipes per vendor, no code; they handle pagination, incremental sync and retries. *Weakness:* niche GTM tools (HeyReach) are often missing.
2. **Self-hosted n8n**: same tool, community edition on your own server, flat cost. *Trade-off:* you run the server; still a poor fit for bulk extraction.
3. **Code-first orchestrator** ([[Trigger.dev]], Inngest, Prefect, Dagster): connectors as small functions; the platform schedules, retries and logs.
4. **Fully custom** (Supabase Edge Functions + pg_cron): cheapest; every failure is yours to catch.

## Current position (research, not a commitment)
**The architecture is decided; the engine is not.** Both pains are real (cost *and* fragility), so a **code-first orchestrator** is a reasonable default, likely alongside managed connectors (Airbyte) where they exist. [[Trigger.dev]] is one candidate, not the choice.

> [!warning] Correction (Oct 18)
> Earlier versions of these notes marked Trigger.dev as "the decision" and planned a pilot on it. That overstated things: it was used as the worked example, without a fair comparison. Inngest, Temporal, self-hosted n8n and fully custom (Supabase + cron) are all still live options.

Per [[Engine vs Core]], the tool choice matters *less* than the data model, the event log and the worker contracts, because the engine is a swappable part.
