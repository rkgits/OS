---
tags: [gtm-os, ingestion, framework]
status: evergreen
created: 2026-10-15
updated: 2026-10-18
---

# Per-Source Decision Framework

Assess every source on two sides; the [[Ingestion Models|ingestion model]] then picks itself.

## Demand side (what you need)
- **Freshness requirement**: how stale can it be before a decision suffers?
- **Criticality**: how much damage if it's wrong or missing?

These are two different axes. A hot reply is high-freshness; quarterly campaign cost is low-freshness but high-criticality. Don't collapse them into "importance".

## Supply side (the vendor's integration surface)
- **Webhooks**: do they exist, and for which events?
- **Incremental sync**: can you pull "changed since timestamp"? (The richer form is **change data capture**.) If not, you're doing full re-pulls.
- **Rate limits and pagination**: the unglamorous killer, usually discovered in production.

> [!todo] Next step
> Build this as a per-source inventory table of all 20–40 systems, including whether Airbyte already covers each one. See [[Open Threads]].
