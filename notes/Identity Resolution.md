---
tags: [gtm-os, identity, concept]
status: growing
created: 2026-10-15
updated: 2026-10-18
aliases: [entity resolution, deduplication, identity graph]
---

# Identity Resolution

Seeing one human's journey across every system requires **identity resolution** (a.k.a. **entity resolution**). Collapsing duplicates is **deduplication**; the result is a **unified customer profile**; the linking structure is an **identity graph**.

## Rules
- Mint your own **canonical internal ID**. Never rely on a vendor's ID.
- The hard part is deciding *which records deserve the same ID*, not assigning it.
- **Deterministic matching** first (verified email, LinkedIn URL), then **probabilistic matching** (weak signals like name + company + city, scored).
- **Never merge in place.** Keep raw records intact; identity is a separate layer linking back, so bad merges can be unwound.

## Two thresholds and a review queue
Every candidate pair gets a **match score**. Set *two* thresholds:
- above the high one → **auto-link**
- below the low one → **keep separate**
- in between → **review queue**, where a human decides

Those human decisions become labelled data that improves the rules over time.

## Building it with AI
- AI can write the code easily, and open-source libraries exist: **Splink**, **dedupe**.
- The hard part is **judgment**: which signals count, where thresholds sit, what "same person" means. An AI assistant will happily produce confident logic that quietly fuses two different people.
- **LLM as reviewer**: let a model judge the grey-zone pairs and explain its verdict. Run it *only* on uncertain pairs, because running it on everything is slow and expensive.

Next level up: [[Person vs Account]].
