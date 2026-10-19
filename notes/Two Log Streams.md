---
tags: [gtm-os, observability, data-model, decision]
status: growing
created: 2026-10-19
updated: 2026-10-19
aliases: [technical logs vs business events, event log]
---

# Two Log Streams

**Decision: split by audience.**

| | Technical logs | Business events |
| --- | --- | --- |
| Answers | Is the *system* healthy? | What happened to our *leads and accounts*? |
| Read by | Whoever maintains the system | The GTM team, managers, dashboards |
| Shape | Detailed, noisy, changes with the code | Simple, stable names ("lead enriched", "reply received") |
| Retention | Short (weeks) | Long-term |
| Access | Limited (may hold sensitive detail) | Broad |
| Lives in | The orchestrator's logs / a log tool | **One table in the core Supabase database**, next to people and accounts |

## Correction
Don't build manager-facing visuals on top of technical logs. They're noisy, unstable and can be sensitive. Manager visuals belong on business events, which are designed to be read, counted and charted.

## Clarifications
- The business event table is **not a separate database**. It's one table in the core Supabase database: the event log called "the backbone" in [[System Design Principles]].
- For business events, Supabase is both the **store** and the source for the **view** (the dashboard), which is why events must be clean and stable.
- The two streams share one thing: the [[Correlation ID]]. When a manager asks "why did this lead never get an email?", you jump from their business event straight to the technical trail.
- Both streams are written through the same [[Telemetry Module]].

> [!todo] Next
> Design the business event table: which events exist and what each one records. See [[Open Threads]].
