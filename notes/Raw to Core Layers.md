---
tags: [gtm-os, data-model, ingestion, concept]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [medallion architecture, bronze silver gold, crosswalk]
---

# Raw to Core Layers

**Question:** when HubSpot sends you a contact, do you store HubSpot's record as is, or convert it straight into your Person?

> [!warning] Correction
> "Everything needs to be translated into the master data model" is **half right**. Translate, yes, but **store the original first, exactly as it arrived.**

## Why keep the raw record
- **Your translation code will have bugs.** With the raw data saved, you fix the code and **re-run** it. Without it, that data is gone.
- **Fields you ignore today**, you'll want in six months, and they're already sitting there.
- **When numbers look wrong**, you can trace them back to exactly what the source sent.

## Three layers
| Layer | Medallion name | What it is |
| --- | --- | --- |
| **Raw** | Bronze | Exactly as received, per source, never edited |
| **Cleaned** | Silver | Typed and tidied, still per source |
| **Core** | Gold | Your master model ([[Core Data Model]]) |

This is the **medallion architecture**. You're already doing it: in the [[Form Intake Pipeline]], `raw_body` is the bronze layer, stored **before** parsing, and that saved the first real submission.

## The crosswalk
A small table linking **each source's ID to your ID**: *(your ID, source, source's ID)*. It's how a HubSpot contact, a Clay row and a Smart Lead lead all point at the same Person.

Related: [[Webhook Ingress]] (the same "store first" idea at the front door), [[Survivorship]].
