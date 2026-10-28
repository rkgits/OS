---
tags: [gtm-os, operations, practices]
status: seed
created: 2026-10-28
updated: 2026-10-28
aliases: [running it, CI/CD, environments, WIP limits]
---

# Engineering Basics

Layer 6 of the [[System Map]]: the practices v0 skipped, which start to hurt as the system grows.

## Finish what you shipped before starting something new
Two v0 gaps mattered more than the next feature:
- **Verify what you already receive.** The plugin signs requests but the receiver ignored the signature ([[Form Submission Contract]]).
- **Fail loudly, not silently.** If the hand-off to the engine fails and nobody is told, the row just sits there. Your old n8n setup failed loudly; a system that can fail silently is worse ([[Form Intake Pipeline#Lessons from the first real submission]]).

## Basics
- **Automatic deploys (CI/CD).** Push to `main` → tests run → deploy. Deploying by hand from a laptop lets what's running **drift** from what's in Git.
- **Separate test and live environments.** v0's test rows ended up in the live tables. That's the symptom; the fix is a dev Supabase project or branch.
- **Secrets management.** Supabase, Trigger.dev and Gmail credentials already live in three places. Write down where each lives and who can change it, before there are forty.
- **An event catalog.** A written list of event names and naming rules (`form_submission.*`, `subscriber.*`…). Without it, names drift by the tenth workflow and dashboards break ([[Two Log Streams]]).

## Limit work in progress
> [!warning] The trap
> Twelve parallel threads is how 30 fragile n8n flows happened. **Limit how much is in progress at once**: finish, then start.

**Defer, don't skip:** postponing a big item is fine; skipping a cheap safeguard isn't ([[Source Routing#Side effects]]).

Related: [[Observability]].
