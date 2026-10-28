---
tags: [gtm-os, data-model, concept, decision]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [event log vs state table, event sourcing, projection, reconciliation]
---

# Events vs State

## The difference
- An **event** is a **fact about something that happened**, and it **never changes**. *"At 3pm, deal 42 moved from Proposal to Negotiation."*
- A **state** is the **current picture**. *"Deal 42 is in Negotiation."*
- **The state is just the result of all the events so far.**

So a stage change is an event, and **you don't need a separate stage-history table: the event log *is* the history** ([[Two Log Streams|business event log]]).

## The practical pattern
Asking "which deals are in Negotiation right now?" by replaying every event is slow and clumsy. So:
- When something changes, **write the event *and* update the current-state table, together in one transaction.**
- **Events are the truth for history; the state table is a fast shortcut for the present.**

Taken all the way, where everything is rebuilt from events, this is **event sourcing**. It's powerful, but **overkill here**. **Decision:** an event log plus state tables.

## When they disagree
The **event log wins**. It's the **source of truth**; the state table is a **derived view** (a **projection**).

> [!warning] Corrections
> 1. **Wording:** "dig into the logs" should be "check the **business events**". Logs are the technical debugging stream. Keep the words apart or the design blurs in your head.
> 2. **Don't write a new event to fix the table.** Nothing happened in the business, so a new event would be **fake history**. Recompute the state from the events instead. That's **reconciliation** (rebuilding the projection).

**The only time you add an event to fix something is when an event itself was wrong.** Then you never edit it; you add a **correcting event**, like an accountant's **reversing entry**.

Related: [[Temporal Modelling]], [[Core Data Model]], [[System Design Principles]].
