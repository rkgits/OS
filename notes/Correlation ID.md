---
tags: [gtm-os, observability, concept, decision]
status: growing
created: 2026-10-19
updated: 2026-10-19
aliases: [trace ID, context propagation]
---

# Correlation ID

One ID given to something when it **first enters the system** (a webhook arrives, a lead is created), then passed through every step and worker. Every log line and business event carries it. Searching that one ID shows the whole journey in order: **tracing without a tracing tool**.

Passing it from step to step is called **context propagation**.

## Who mints it? Your core, not the engine
**Decision: your own code mints the correlation ID.**

- Trigger.dev (or any orchestrator) gives each *run* its own **run ID**. That's useful, but one lead's journey spans many runs, sometimes days apart (sign-up today, reply next week). A run ID can't follow the lead.
- So the core mints the correlation ID at the entry point, stores it with the lead, and passes it into every task. The engine's run ID is recorded as an *extra field* next to it.
- If the IDs came from the engine, switching engines would break the trail. That's the [[Engine vs Core]] rule again.

## Who produces technical logs?
The orchestrator does, because it's what calls HeyReach, Smart Lead and the rest. Vendors won't hand over their internal logs; you only log **what you sent them and what came back**.

Implemented by the [[Telemetry Module]]. Links the [[Two Log Streams]].
