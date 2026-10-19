---
tags: [gtm-os, build, ingestion]
status: seed
created: 2026-10-19
updated: 2026-10-19
aliases: [website form webhook, first build]
---

# Form Intake Pipeline

**The first real build:** website form submissions (Galliot Forms, a custom WordPress plugin on galliot.us) → spam decision → business events. Everything else gets built around the patterns proven here.

## Today
The plugin has a webhook option (*Settings > Galliot Forms*). None is configured, so submissions fall back to **email**. The plugin also has a **Send test event** button.

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

Related: [[Form Submission Contract]], [[Ingestion Models]].
