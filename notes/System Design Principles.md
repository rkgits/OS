---
tags: [gtm-os, architecture, principles]
status: growing
created: 2026-10-18
updated: 2026-10-19
---

# System Design Principles

## Where we're starting from
The existing system grew by **organic growth** (architecture by accretion): small n8n automations (spam checks, adding sign-ups to Beehiiv) grew into lead signal processing and lead gen. Each became an independent **worker** with one job, and a dashboard was bolted on afterwards. Every piece made sense on its own day. What's missing is a shape that ties them together. The core flaw: **visibility came last**.

## Five deliberate decisions for the scaled version
1. **A shared data model**: every worker reads and writes the same core entities (person, account, event). No private formats per workflow. See [[Person vs Account]].
2. **An event log as the backbone**: every worker records what it did ("lead scored", "email flagged as spam"). The dashboard becomes a view over that log, not a separate project. *If you keep only one, keep this one.* See [[Two Log Streams]].
3. **Clear worker boundaries**: one input, one output, a defined **contract**. Workers trigger each other, but never reach into each other's data or logic. See [[Worker Coordination]].
4. **Orchestration separate from logic**: the engine schedules and retries; your rules live in code you own. See [[Engine vs Core]].
5. **Observability by design**: every run logs status, timing and failures, so fragility shows up before your manager notices. See [[Observability]].

Back to [[GTM OS - Map of Content]].
