---
tags: [gtm-os, build, ingestion]
status: seed
created: 2026-10-19
updated: 2026-10-19
aliases: [website form webhook, first build]
---

# Form Intake Pipeline

**The first real build:** website form submissions (Galliot Forms, a custom WordPress plugin on galliot.us) → spam decision → business events. Everything else gets built around the patterns proven here.

## Status
**v0 is built** in `galliot-us/GTM-Engine`: Supabase Edge Function receiver → Trigger.dev task → rule-based spam decision → result email → business events. The ticket is [issue #1](https://github.com/galliot-us/GTM-Engine/issues/1); remaining gaps are tracked in that repo, not here.

> [!warning] Correction
> The assumption "the plugin doesn't sign requests" was wrong. It can already sign (`X-Galliot-Signature`); the receiver just doesn't verify it yet, so a URL token is the only check.

## Correction
"Build on top of Trigger.dev". The phrase should be **build *on* it, not around it** ([[Engine vs Core]]).

## Design
Trigger.dev doesn't (as far as we know) expose a public URL that WordPress can post to; tasks are started from your own code. So a small **webhook receiver** sits in front (e.g. a Supabase Edge Function). That's good news, because the receiver is where the core does its job:

1. **Verify** the request came from the site (signature check; see [[Form Submission Contract#Delivery]]).
2. **Deduplicate** on the source `event_id` (**idempotency**): a retried delivery is processed once.
3. **Mint the [[Correlation ID]]**.
4. **Save the raw submission** untouched to a staging table.
5. **Trigger the spam-check task** in the orchestrator and **answer WordPress immediately**. Never make the website wait for slow work.

The task then decides spam vs not, and writes business events ([[Two Log Streams]]): `submission received` → `flagged as spam` | `accepted`.

## The spam decision
The plugin *supplies* signals; the **core decides**, so rules change without redeploying WordPress.

Same layered pattern as [[Identity Resolution]]:
- **Simple rules first** catch the obvious cases.
- **An LLM judges only the grey zone.**
- **Two thresholds + a review queue.**

> [!example] Test case (real email, Oct 18)
> 2-character message ("Hy"), a free Gmail address containing "seoexpert", a two-year-old browser (Chrome 109), from Pakistan to a US consulting site, **captcha passed**. Almost certainly spam. That's why captcha alone isn't enough.

## Personal data
IP, country, user-agent and time zone are personal data (e.g. under GDPR). They're fine in the raw staging table with limited access; the [[Telemetry Module]] keeps them **out of technical logs**.

Build ticket: [GTM-Engine #1](https://github.com/galliot-us/GTM-Engine/issues/1).

Why the receiver sits outside the engine: [[Webhook Ingress]]. Adding more sources: [[Source Routing]].

Related: [[Form Submission Contract]], [[Ingestion Models]].
