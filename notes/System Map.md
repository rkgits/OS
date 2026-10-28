---
tags: [gtm-os, architecture, moc]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [six layers, the big picture, where to start]
---

# System Map

## Why this note exists
The pain point, in Reza's words: building GTM systems by **vibe coding** without a software-engineering background, and wanting enough **architecture knowledge to make the right decisions**, so the system can scale and take new plugins, features and add-ons later.

> [!warning] Correction
> The "next steps" offered earlier (pull sync, CI/CD, event catalog, identity, dashboard…) were all relevant, but **a list of topics isn't a map.** Without the map you can't tell where a topic sits or what to do first.

## The six layers (bottom up)
| # | Layer | What lives here | Notes |
| --- | --- | --- | --- |
| 1 | **Connectors** | Pulling from and receiving data from every tool | [[Ingestion Models]], [[Webhook Ingress]], [[Pull-Based Sync]], [[Source Routing]] |
| 2 | **Core data** | The central store, the shared model of people and accounts, the event log | [[Core Data Model]], [[Choosing the Store]] |
| 3 | **Workers** | Jobs and AI agents that enrich, score and decide | [[Worker Coordination]], [[Form Intake Pipeline]] |
| 4 | **Actions** | Sending emails, updating the CRM, triggering campaigns | *(not covered yet)* |
| 5 | **Interface** | The dashboard and controls | *(not covered yet)* |
| 6 | **Running it** | Observability, security, deploys, cost | [[Observability]], [[Engineering Basics]] |

**Extensibility is not a seventh layer.** Plugins and add-ons come from **clean contracts between the layers**: a new tool is a new connector mapping into the same core model, and a new feature is a new worker reading the same events ([[Engine vs Core]], [[Worker Coordination]]).

## Where to start: layer 2
**Decision: start with the core data model.** Everything else plugs into it, and it's the **hardest thing to change later**. Get it right and every new tool simply maps into it; get it wrong and every new tool needs its own special case.

Back to [[GTM OS - Map of Content]].
