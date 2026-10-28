---
tags: [gtm-os, todo]
status: growing
created: 2026-10-15
updated: 2026-10-28
---

# Open Threads

Learning and design questions only. Build work lives as issues in [GTM-Engine](https://github.com/galliot-us/GTM-Engine/issues).

- [ ] Fair comparison note: Trigger.dev vs Inngest vs Temporal vs self-hosted n8n vs fully custom.
- [ ] Design one example connector (e.g. Smart Lead) engine-agnostically, following [[Engine vs Core]].
- [ ] A shared connector pattern so custom connectors don't become twenty one-offs (common interface: auth, incremental cursor, pagination, raw landing table).
- [ ] Per-source inventory of all 20–40 systems ([[Per-Source Decision Framework]]), including Airbyte coverage.
- [ ] **Which signing scheme to standardise on** across sources: verify the plugin's own `X-Galliot-Signature` HMAC (no plugin change), or move every source to Standard Webhooks (shared convention, library support, plugin change) ([[Form Submission Contract#Delivery: Standard Webhooks vs the plugin's own scheme]]).
- [ ] **Telemetry in two runtimes:** one shared module that runs in both Node and Deno, or two thin implementations of one field contract ([[Telemetry Module#Where v0 stops]]).
- [ ] **Who owns recovery?** When the receiver can't hand off to the engine, or a run fails for good, the submission sits in a table. Is reprocessing a scheduled sweeper, a manual replay, or part of the engine's retry? And which alert channel is independent of the mail path it watches ([[Form Intake Pipeline#Lessons from the first real submission]])?
- [ ] Tuning the spam rules: how to use human overrides (`reviewed` events) to adjust weights and thresholds, and when the grey zone justifies an LLM judge ([[Form Intake Pipeline#The spam decision]]).
- [ ] Verify how Trigger.dev uses [[OpenTelemetry]] in its docs.
- [ ] How the match score is built ([[Identity Resolution]]).
- [ ] Real-time vs good-enough tradeoffs, source by source.
- [ ] Newsletter platform: stay on Substack (manual/CSV) or move to one with an API? ([[Source Routing#Substack]])
- [ ] **Answer the cursor exercise** in [[Pull-Based Sync#Exercise (unanswered)]] before opening the answer.
- [ ] How accounts roll up: parent companies and subsidiaries ([[Core Data Model]]).
- [ ] Pick one of the [[Data Model Management]] practices to go deeper on.
- [ ] Layers 4 and 5 of the [[System Map]]: actions, and the interface.
- [ ] Moving from visualization to **action** from the dashboard, without welding the insight and action planes together.

## Resolved
- [x] Where do I start, and what's the big picture? → [[System Map]]; start at layer 2, the [[Core Data Model]].
- [x] A person changes company: update or keep history? → keep it; employment records with dates ([[Temporal Modelling]]).
- [x] Is a stage change an event, or a separate history table? → an event; plus a state table as a shortcut ([[Events vs State]]).
- [x] Store a tool's record as is, or translate it? → both: raw first, then translate ([[Raw to Core Layers]]).
- [x] Two sources disagree: which wins? → designed per-field rules ([[Survivorship]]).
- [x] Does the plugin sign requests, and does it retry? → Yes to both: its own HMAC scheme (optional secret) and a retry queue at 5 min, 15 min, 1 h, 3 h, 6 h, with possible duplicate deliveries ([[Form Submission Contract#How the plugin delivers]]).
- [x] What does the real payload look like? → Captured and verified, including PHP's empty-array quirk ([[Form Submission Contract#Quirks the real traffic exposed]]).
- [x] Design the business event table → four stable v0 events, outcome in the data ([[Two Log Streams#The four v0 events]]).
- [x] Why does the webhook land on Supabase, not Trigger.dev? → [[Webhook Ingress]] (receiver vs processor).
- [x] How to add a second website without building twice? → [[Source Routing]].
- [x] First build → [[Form Intake Pipeline]] v0 built ([GTM-Engine #1](https://github.com/galliot-us/GTM-Engine/issues/1)).
- [x] Which hurts more, n8n cost or fragility? → Both, so move to a code-first orchestrator ([[Ingestion Layer Options]]).
- [x] Choose a tool vs build around it → [[Engine vs Core]].
- [x] What's a good logging standard? → [[Observability]], [[Two Log Streams]], [[Telemetry Module]].
- [x] Who creates the job / event ID? → the core mints a [[Correlation ID]]; the engine's run ID is an extra field.
