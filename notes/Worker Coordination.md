---
tags: [gtm-os, architecture, agents]
status: growing
created: 2026-10-18
updated: 2026-10-18
aliases: [orchestration vs choreography]
---

# Worker Coordination

## Correction
"Clear boundaries" does **not** mean workers can't trigger each other. They should. It means they cooperate through a defined **contract** (inputs and outputs) and never reach into another worker's data or logic.

## Two ways to cooperate
1. **Orchestration**: a conductor workflow calls workers in order. Easy to follow and debug.
2. **Choreography**: each worker emits an event ("I finished"), and interested workers react. No conductor; more flexible, harder to trace.

## AI agents
AI agents can decide what to do next, and the rule applies *more* strongly to them. Agents calling each other freely, with no contracts, means one bad output cascades through the chain and you can't tell which agent caused it. That's the fragility you're escaping.

**Scaled answer:** AI handles the **judgment inside** each worker; the **wiring between** workers stays explicit; every handoff is logged on the event log ([[System Design Principles]]).
