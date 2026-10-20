---
tags: [gtm-os, ingestion, architecture, concept]
status: growing
created: 2026-10-19
updated: 2026-10-19
aliases: [inbox pattern, store-then-process, receiver vs processor]
---

# Webhook Ingress

**The receiver and the processor are two different things. They always were; n8n just hid it.**

## The question
With n8n, a webhook URL lived *inside* n8n: the payload arrived and the workflow started right there. In [[Form Intake Pipeline|GTM-Engine v0]] the payload arrives at a **Supabase Edge Function**, which then hands off to Trigger.dev. Was that move intentional?

## Corrections
1. **The receiver doesn't "log" the payload, it *stores* it.** Logging is a diagnostic trail for maintainers. Storing (persisting) makes the row the real, owned record of the submission. Mixing the two words blurs the [[Two Log Streams]] model.
2. **n8n didn't have a different architecture.** Its webhook node *is* a receiver, bundled inside the engine, so the separation was invisible.
3. **The split is receiver vs *processor*, not receiver vs store.** The store is where the receiver puts things; the two work together.

## Intentional or accidental?
Both. It started as a **constraint**: Trigger.dev has no public URL for a website to post to, and calling its API from WordPress would put the Trigger.dev secret key in the plugin. But it's the design you'd choose anyway:

1. **The front door belongs to the core.** Verifying the sender, deduplicating, minting the [[Correlation ID]] and storing the raw data happen in code you own. Swap the engine and no website needs a new URL. That's [[Engine vs Core]] applied to the entry point.
2. **Store first, process later.** If the engine is down, the submission is already safe in Postgres. With n8n down, it was simply lost.
3. **Acknowledge fast, process asynchronously.** The website gets an answer in milliseconds, whatever the processing does.

## Names for it
- **Webhook ingress layer**: the part that receives inbound webhooks.
- **Store-then-process**, or the **inbox pattern**: requests land in a durable inbox table and workers pick them up.
- It's the standard pattern: Stripe, GitHub and Shopify all advise receiving webhooks this way. Some teams buy the ingress as a product (e.g. Hookdeck, or Svix's ingest service) or build it from API Gateway plus a queue.

## Shape
```
website ──POST──▶ receiver (core) ──stores──▶ inbox table
                      │
                      └──triggers──▶ processor (engine task)
```

Next step up, many sources through one front door: [[Source Routing]].
