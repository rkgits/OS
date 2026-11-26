---
tags: [gtm-os, security, privacy, concept]
status: seed
created: 2026-11-25
updated: 2026-11-25
aliases: [PII, data minimisation, retention, right to erasure, GDPR]
---

# Personal Data and Retention

> [!warning] Engineering reasoning, not legal advice
> This note is about how to design the system sensibly. What the law requires for a given case is a question for someone qualified.

## What counts as personal data here
Anything that identifies a person, directly or in combination: **names, email addresses, free-text messages** (people write all sorts of things), **IP addresses**, **user agents** and time zones (they narrow a person down). A **salted hash** of an IP or email is **pseudonymised**, not anonymous: it can still single someone out, and anyone holding the salt could re-create it.

## Where it lives in this system
Copies multiply, which is the first thing to understand:

| Where | What | Notes |
| --- | --- | --- |
| The raw submissions table | The whole submission: fields, context, request headers | The **system of record**; WordPress keeps no copy, so deleting here is final ([[Form Intake Pipeline#Personal data]]) |
| The dashboard view | Name, email, a 200-character message preview, page, score and reasons | Shown to allowlisted users in a browser ([[Dashboard Architecture]]) |
| Notification emails | The full message and sender, to the owner's inbox | Lives in Gmail, outside our control |
| Rate-limit hashes | Salted hashes of IP and email, per submission | Needed for **about an hour**, kept indefinitely |
| Database backups | Everything, as of each night | Deleted rows survive in backups until they age out |
| Technical logs and business events | **Nothing personal, by design** | Enforced by redaction ([[Telemetry Module]], [[Two Log Streams]]) |

## Principles (the vocabulary)
- **Data minimisation:** collect and keep only what you need. The events table is the good example: it holds the *fact* ("classified as spam"), not the person.
- **Purpose limitation:** data kept for one purpose (spotting spam, answering an enquiry) isn't quietly reused for another.
- **Storage limitation (retention):** each kind of data has an end date, decided in advance.
- **Erasure:** a person can ask to be removed, so there must be a way to find and delete everything about them, **across the copies above**.

## Where we are
**Nothing expires.** Every raw body and header is kept forever, and part of it is now displayed in a browser. Nothing is wrong *yet*, but the decision has been deferred, not made ([[Engineering Basics]]: defer, don't skip). Tracked in [GTM-Engine #40](https://github.com/galliot-us/GTM-Engine/issues/40).

## Design questions that fall out
- **Different fields can expire at different times.** The rate-limit hashes need an hour; request headers and IP-like context rarely matter after the spam decision; the email and message are the *enquiry itself* and live as long as the business needs them. Blanking fields beats deleting rows, because the **decision and its events stay** for counting.
- **Erasure has to reach the copies:** the table, the notification emails, and the backups (which age out on their own schedule, so a deletion isn't instant everywhere).
- **Deleting is irreversible while WordPress keeps nothing,** so a scheduled clean-up must start with a dry run and write a business event for each run.

Related: [[Dashboard Architecture]], [[Form Submission Contract]]. Back to [[GTM OS - Map of Content]].
