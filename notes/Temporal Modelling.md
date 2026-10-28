---
tags: [gtm-os, data-model, concept]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [slowly changing dimensions, SCD, employment history, effective dating]
---

# Temporal Modelling

**Relationships that change over time are records with a start and an end, not fields you overwrite.**

## The case: a person moves from Company A to Company B
> [!warning] Correction
> The instinct was to *update* the person's company from A to B, and "should we keep the history or not?" **Don't update. Always keep the history.**

The person→company link is **its own record**, an **employment**: *(person, account, title, start date, end date)*. When they move:
1. **Close** the Company A employment (set its end date).
2. **Open** a new employment at Company B.

Nothing is overwritten. Every email and meeting stays tied to the **person**, and to the **company they were at when it happened**, so Company A's account history stays truthful.

**Names:** **temporal modelling**, **slowly changing dimensions** (SCD), effective dating.

**Why it's commercially valuable:** a past champion who joins a new company is one of your hottest leads. You can only spot that if the history exists.

## Another case: deals
> [!warning] Correction
> "A company is a prospect, then a lead, then closed won or closed lost, then maybe another deal" mixes **two different things**.

| | Lifecycle stage | Deal stage |
| --- | --- | --- |
| Belongs to | The **person or account** | One specific **deal** (an opportunity with money attached) |
| Describes | Your relationship with them | How far that deal has progressed |
| Examples | Prospect, lead, customer | Proposal, negotiation, **closed won**, **closed lost** |

- One account can have **many deals over time**: a **one-to-many** relationship.
- Each deal moves through its own stages, and you keep **every stage change with a timestamp**, not just the current stage. That stage history answers "how long do deals sit in negotiation?" and "where do we lose them?"
- Same principle: **don't overwrite, record the change.** How that's stored: [[Events vs State]].

Related: [[Person vs Account]], [[Core Data Model]].
