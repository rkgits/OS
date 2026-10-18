---
tags: [gtm-os, ingestion, concept]
status: evergreen
created: 2026-10-15
updated: 2026-10-18
aliases: [push vs pull, webhooks vs polling]
---

# Ingestion Models

The axis is **push vs pull**. A webhook *is* push; **polling** (not "pulling") is the scheduled form of pull.

## 1 — Scheduled batch pull (polling)
- Pro: simple, predictable, debuggable.
- Con: latency; wasted quota asking "anything new?"

## 2 — Webhooks (push)
- Pro: near real-time.
- Con: needs an always-listening endpoint, **idempotency** (a repeated event must be harmless) and **retries**.

## 3 — Hybrid ✅
Webhooks for what must feel live (a hot lead, an inbound reply); batch for slow bulk (nightly campaign stats).

**Rules:**
- Vendor capability often decides: Clay, Smart Lead and HeyReach have **patchy webhook coverage**.
- Always keep a nightly batch pull as a **backstop**, because webhooks silently drop events.
- Keep the **system-of-insight** separate from the **system-of-action**. The analytics layer does not take business actions.

How to decide per source: [[Per-Source Decision Framework]]. What runs the pulls: [[Ingestion Layer Options]].
