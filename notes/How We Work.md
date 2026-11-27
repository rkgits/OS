---
tags: [gtm-os, process, learning, rules]
status: growing
created: 2026-11-26
updated: 2026-11-26
aliases: [rules and procedures, working agreement, how we build]
---

# How We Work

The rules and procedures we've set while building, **in one place so you can learn them and hold me to them**. If a rule here is wrong or missing, that's a correction to make.

## The loop for every feature
1. **Talk architecture first.** Before any building I explain the shape (what talks to what), each decision with its **reason**, the **alternatives**, and what I'm **least sure about**. You question and change it. *No building until that conversation has happened.*
2. **One ticket per task**, as GitHub issues in `galliot-us/GTM-Engine`, linked to an epic. Concepts and reasoning go in this vault, never tickets.
3. **Build on a branch**, with tests. **Prove each safeguard by breaking it** ([[Dashboard Architecture#Proving it: a safeguard you've never seen fail is only a belief]]).
4. **Open a PR and wait for the checks to go green.** CI here is a *signal*, not a gate, so the discipline is ours ([[Engineering Basics]]).
5. **Merge only when you say so**, for that specific PR.
6. **Deploy by hand with named commands**, then do a live check against the real thing.
7. **Update the README** if behaviour changed, and **capture the reasoning here** ([[Recurring Lessons]]).

## Design rules (written in the repo's `CLAUDE.md`)
| Rule | Why | Read more |
| --- | --- | --- |
| **Logic in plain functions; tools are thin wrappers** | The tool is replaceable; the rules are yours | [[Engine vs Core]] |
| **Store first, process later** | Bad input must never lose data | [[Webhook Ingress]], [[Raw to Core Layers]] |
| **The source comes from how a request authenticated, never from its body** | A body can claim anything | [[Source Routing]] |
| **One correlation ID on everything** | One search shows a whole journey | [[Correlation ID]] |
| **Technical logs and business events are separate; no personal data in either** | Different readers, different lifetimes | [[Two Log Streams]], [[Telemetry Module]] |
| **The dashboard reads only through narrow views** | The browser is public; the database is the boundary | [[Dashboard Architecture]] |
| **Assume every action runs twice; guard side effects** | Retries repeat things | [[Idempotency]], [[Message Triage and Sender Research#One task or several? A rule of thumb]] |
| **Fail open when a miss costs more than a false alarm** | Make the cheap mistake the one that happens | [[Message Triage and Sender Research]] |

## Habits and limits
- **Ask before** creating a repository, a database project, any cloud resource, or changing settings on a shared system.
- **Live-table changes in two steps:** add, deploy the code that uses it, then clean up. *And not before the design is agreed.*
- **Secrets** never go in the repo, in chat, or in client code. You type them yourself where they're stored.
- **Security is proportional.** This is an internal tool with low-importance data: do the basics (login, no secrets in the browser), not an audit.
- **Say plainly where a rule was broken**, and what changed because of it.

## Where each kind of rule lives
- **The repo's `CLAUDE.md`:** rules for the code.
- **This vault:** the reasoning, so you can learn it.
- **The assistant's memory:** preferences about how I work with you (for example "design first", "merges need your yes").

## Slips, and what changed
- **Nov 25:** merged without asking, and turned a review into too many tickets. → merges need your explicit yes; security stays proportional.
- **Nov 26:** asked product questions, then started building (and migrated production) without walking through the architecture. → rule 1 above; nothing is applied to a live system before the design is agreed.

Back to [[GTM OS - Map of Content]].
