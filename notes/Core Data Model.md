---
tags: [gtm-os, data-model, concept]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [data model, master data model, canonical model]
---

# Core Data Model

A **data model** is your decision about **which things exist in your business and how they relate**. It's layer 2 of the [[System Map]].

## The core nouns (entities)
- **Person**: a human you might sell to.
- **Account**: the company they work at ([[Person vs Account]]).
- **Touchpoint / event**: anything that happened (email sent, reply, form filled in, meeting).
- **Campaign**: the effort that caused it.
- **Deal**: the money outcome.

## Your model is the master
Clay, HubSpot and Smart Lead each have **their own version of a person**. Your model is the **master version**; each tool's record only *links* to it. That's why you mint your **own ID**.

## Identifiers
> [!warning] Correction
> "A person has a LinkedIn ID, and that stays the same." A LinkedIn ID is a strong **identifier**, but not your **primary key**. Some people have no LinkedIn, and URLs change.

- **Primary key** = your own ID, always.
- **External IDs** (LinkedIn URL, HubSpot contact ID, Clay ID…) are stored side by side in a **crosswalk** table: one row per *(your ID, source, source's ID)*. They don't compete, so none of them replaces another.

> [!warning] Correction
> Asked "which field should HubSpot always win?", the answer "person ID" was a trap. **The person ID is yours**, so no tool gets a say. HubSpot's contact ID is just another row in the crosswalk.

## The deeper notes
- Relationships that change over time: [[Temporal Modelling]]
- History vs the current picture: [[Events vs State]]
- From a tool's raw record to your model: [[Raw to Core Layers]]
- When two sources disagree: [[Survivorship]]
- Keeping the model healthy over years: [[Data Model Management]]
- Matching records to the same person: [[Identity Resolution]]

> [!todo] Not covered yet
> How accounts roll up (parent company, subsidiaries). See [[Open Threads]].
