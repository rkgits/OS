---
tags: [gtm-os, ingestion, decision]
status: growing
created: 2026-10-18
updated: 2026-10-18
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

## Decision ✅
Both pains are real (cost *and* fragility), which points past self-hosted n8n to a **hybrid**: Airbyte wherever a connector exists, Trigger.dev for everything else and for enrichment. Pilot with **one Smart Lead connector**.

The tool is still a replaceable part: [[Engine vs Core]].
