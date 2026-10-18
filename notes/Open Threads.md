---
tags: [gtm-os, todo]
status: growing
created: 2026-10-15
updated: 2026-10-18
---

# Open Threads

- [ ] Design the pilot: one **Smart Lead connector** on [[Trigger.dev]], following [[Engine vs Core]].
- [ ] A shared connector pattern so custom connectors don't become twenty one-offs (common interface: auth, incremental cursor, pagination, raw landing table).
- [ ] Per-source inventory of all 20–40 systems ([[Per-Source Decision Framework]]), including Airbyte coverage.
- [ ] Design the **event log**: schema, event names, how the dashboard reads it ([[System Design Principles]]).
- [ ] How the match score is built ([[Identity Resolution]]).
- [ ] Real-time vs good-enough tradeoffs, source by source.
- [ ] Moving from visualization to **action** from the dashboard, without welding the insight and action planes together.
- [ ] Optional: compare Trigger.dev with Inngest.

## Resolved
- [x] Which hurts more, n8n cost or fragility? → Both, so move to a code-first orchestrator ([[Ingestion Layer Options]]).
- [x] Choose a tool vs build around it → [[Engine vs Core]].
