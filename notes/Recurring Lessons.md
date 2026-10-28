---
tags: [gtm-os, learning, summary]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [learning summary, brief, what I've learned]
---

# Recurring Lessons

**The meta-lesson: separate things that look like one thing.** Nearly every correction so far has been a version of it.

| Looks like one thing | Is actually two | Note |
| --- | --- | --- |
| How data arrives / how it's used | OLTP-shaped writes vs OLAP-shaped questions | [[OLTP vs OLAP]] |
| The dashboard | System of insight vs system of action | [[Ingestion Models]] |
| n8n flows | Extraction vs enrichment orchestration | [[Ingestion Layer Options]] |
| The tool | Engine vs core | [[Engine vs Core]] |
| The webhook | Receiver vs processor | [[Webhook Ingress]] |
| Logging | Technical logs vs business events | [[Two Log Streams]] |
| Run ID | Engine's run ID vs your correlation ID | [[Correlation ID]] |
| "Prospect … closed won" | Lifecycle stage vs deal stage | [[Temporal Modelling]] |
| The deal's stage | Event (history) vs state (the present) | [[Events vs State]] |
| A contact from HubSpot | Raw record vs your master Person | [[Raw to Core Layers]] |
| An identifier | Primary key vs external IDs | [[Core Data Model]] |
| "The source" | What the payload claims vs how it authenticated | [[Source Routing]] |
| Authenticating | The sending *site* vs the *person* who typed | [[Source Routing]] |
| Postponing | Deferring vs skipping | [[Engineering Basics]] |

## Habits
- **Don't overwrite; record the change** ([[Temporal Modelling]], [[Events vs State]]).
- **Store raw first, process later** ([[Webhook Ingress]], [[Raw to Core Layers]]).
- **Idempotency:** the same input twice has the effect of once. **Side effects** get a check before acting.
- **Fail loudly.** **Defence in depth** ([[Bot Protection]]).
- **Conflicts are resolved by designed rules, not opinions** ([[Survivorship]]).
- **Limit work in progress** ([[Engineering Basics]]).

Start from the map: [[System Map]].
