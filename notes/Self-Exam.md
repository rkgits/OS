---
tags: [gtm-os, learning]
status: seed
created: 2026-10-28
updated: 2026-10-28
aliases: [exam, test my understanding]
---

# Self-Exam

**Goal:** prove you understand a concept by explaining it **in your own words**, not by repeating terms back. Repeating vocabulary isn't the same as understanding it.

## Three formats
1. **Explain it to a non-engineer.** "Tell me why we keep raw data, as if I'm your sales manager."
2. **Scenarios.** A situation is described (a company gets acquired, a sync re-runs, a webhook arrives twice) and you say what breaks and how you'd design for it.
3. **Spot the flaw.** A bad design is described, and you find the problem.

## Grading
- Graded on whether the **reasoning holds up**, not on using the exact words from these notes.
- The **proper term** is given *after* the answer, so you learn to map your own words onto it.

## Example questions (data model)
- Why does a person's employer get start and end dates instead of being updated? ([[Temporal Modelling]])
- A sync re-reads the last 10 minutes every run. Why doesn't that double-count replies? ([[Idempotency]], [[Pull-Based Sync]])
- Alphabet sells YouTube. What happens to Alphabet's revenue report for last year, and why? ([[Account Hierarchy]])
- A colleague adds an `intent_score` column to the person table. What's your objection? ([[Core Plus Extensions]])
- Clay and HubSpot disagree on a job title. How do you decide? ([[Survivorship]])

Track rounds in [[Open Threads]].
