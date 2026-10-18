---
tags: [gtm-os, foundations, decision]
status: evergreen
created: 2026-10-15
updated: 2026-10-18
---

# Integration Approaches

**Decision: Approach C, a dedicated central store.**

## A — Query the sources live (federated / query virtualization)
Each system stays the source of truth; you query it on demand.
- **Pros:** no duplication, zero staleness, no storage or governance burden; cheapest proof of concept.
- **Cons:** **tight coupling** to every vendor's API, rate limits, schema changes and uptime. **No historical state / no point-in-time analysis**, which disqualifies it for a management dashboard built on trends.

## B — CRM as hub (system of record)
Everything flows into HubSpot or Salesforce.
- **Pros:** one authoritative **system of record** the team already trusts; the viewer can also act there.
- **Cons:** **impedance mismatch** (CRMs model contacts and deals, flattening campaign data); not an analytics engine (slow aggregations, API read quotas); vendor lock-in.

## C — Dedicated central database / warehouse ✅
Land raw data in one store, transform with SQL.
- **Pros:** full history; dashboard **decoupled** from vendors; heavy analytics without touching sources; a **single source of truth for analytics** with **staging** (raw) and **transformed** layers.
- **Cons:** you own the pipeline. See [[Ingestion Layer Options]].

> [!warning] Recurring trap
> "Let the dashboard call the vendor APIs" is Approach A sneaking back in. See [[Ingestion Layer Options#Two corrections]].

Related: [[Choosing the Store]], [[Data Integration]].
