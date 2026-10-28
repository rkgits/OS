---
tags: [gtm-os, data-model, identity, concept]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [golden record, conflict resolution, source of truth per field]
---

# Survivorship

**Question:** Clay and HubSpot give different job titles for the same person. Which wins?

> [!warning] Correction
> "It's subjective; I'd trust Clay because it's connected to the internet and HubSpot goes stale." **It's not subjective. It's a rule you design on purpose**, called **survivorship**. And "Clay always wins" is fine for one case but wrong as a blanket rule: if a sales rep spoke to the person yesterday and typed the new title into HubSpot by hand, that beats Clay's scraped guess.

## Rules are set per field, from a few signals
- **Source priority:** e.g. for job titles, enrichment beats the CRM.
- **Recency:** newer beats older.
- **Manual override:** a human edit beats any automated source.

**Keep every source's value with its timestamp, never just the winner.** The winning value goes on the **golden record**. Change the rules later and you just **recompute**.

## Who knows what
- **Enrichment tools know the outside world**: title, company size, LinkedIn.
- **Your CRM knows your relationship with them.** Fields where HubSpot should always win: **deal stage**, **account owner**, **lifecycle stage**, and above all **opt-out / do-not-contact**.

## When nobody should win
- **IDs:** HubSpot ID and Clay ID don't compete; both are kept in the crosswalk ([[Raw to Core Layers]]).
- **Multi-valued attributes:** a person can have a work and a personal email, or several phones. Keep a **list**, each value with its source, rather than picking one.

Related: [[Core Data Model]], [[Identity Resolution]].
