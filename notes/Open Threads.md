---
tags: [gtm-os, todo]
status: growing
created: 2026-10-15
updated: 2026-11-26
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
- [ ] **First exam round** on the data model ([[Self-Exam]]).
- [ ] Layer 1 of the [[System Map]]: connectors.
- [ ] Pick one of the [[Data Model Management]] practices to go deeper on.
- [ ] Layers 4 and 5 of the [[System Map]]: actions, and the interface.
- [ ] **Mandatory two-step verification for the dashboard?** It would need an extra check inside the view and locks a person out until they enrol; the access tests would need to simulate a second factor ([[Dashboard Architecture#Choices]], [GTM-Engine #12](https://github.com/galliot-us/GTM-Engine/issues/12)).
- [ ] **How long do we keep personal data, field by field,** and how does erasure reach the copies (table, emails, backups)? ([[Personal Data and Retention]], [GTM-Engine #40](https://github.com/galliot-us/GTM-Engine/issues/40))
- [x] **Is Haiku good enough, and where should the promotion confidence floor sit?** → Haiku passed 27 of 28 and found no real inquiry hidden among 39 real messages; the floor is 0.85 ([[Message Triage and Sender Research#What real mail taught us]]).
- [ ] **Should the email wait for research** (one email, but a failure mode), or go at once with research shown later? ([[Message Triage and Sender Research#Open]])
- [ ] Moving from visualization to **action** from the dashboard, without welding the insight and action planes together.

- [ ] **Design system: components next.** Confirm the build path (CSS variables in the existing stylesheet, not a Tailwind and shadcn/ui migration) and approve the expandable row ([[Dashboard Design System#First component: the expandable row]]).

## Resolved
- [x] Where does the design system live? → a folder in the dashboard code ([[Dashboard Design System#Where it lives]]).
- [x] Side panels? → No: details expand in the row, or open on their own page ([[Dashboard Design System#Where details open]]).
- [x] Chart colours and motion → Tableau 10 reordered (red and green last); three speeds, three curves; tooltip, dropdown and save-then-toast approved ([[Dashboard Design System#Chart colours]]).
- [x] Density, corners, shadows, theme → Compact · Medium · Subtle, light only ([[Dashboard Design System#Density, corners and shadows]]).
- [x] Which icon set? → **Lucide**, the shadcn/ui default ([[Dashboard Design System#Icons]]).
- [x] Which colours? → logo blue `#179CFF` as the accent, with darker steps for text and buttons; green and yellow chosen for readability ([[Dashboard Design System#Colour]]).
- [x] Which font for the dashboard? → **Inter**, after a side-by-side test with Geist and IBM Plex Sans ([[Dashboard Design System#Typography]]).
- [x] One task or several for a multi-step worker? → split only where a step differs in kind (waits long, spends money, depends on an outside system); guard the rest ([[Message Triage and Sender Research#One task or several? A rule of thumb]]).
- [x] What are our working rules, in one place? → [[How We Work]].
- [x] Which login system for the dashboard, and why not Cognito? → Supabase Auth, because Postgres can check the login directly ([[Dashboard Architecture#Choices]]).
- [x] How can a browser read some data without being able to read the table? → one definer view with an allowlist guard, a security barrier and tight grants ([[Dashboard Architecture#Pattern: a definer view plus an allowlist]]).
- [x] Is logging in enough to see the data? → No: authentication and authorisation are separate locks ([[Dashboard Architecture#Correction: logging in is not permission to read]]).
- [x] The cursor exercise → missed, because the record is stamped before the cursor; fixes: cursor from the data, lookback window, upserts, advance only after saving ([[Pull-Based Sync#Rules for a safe cursor]]).
- [x] What is idempotency, and are dashboard actions naturally idempotent? → No, it's designed in ([[Idempotency]]).
- [x] How accounts roll up → [[Account Hierarchy]].
- [x] Where does a plugin store its own data? → an extension table ([[Core Plus Extensions]]).
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
