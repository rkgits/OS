---
tags: [gtm-os, architecture, workers, ai, decision]
status: growing
created: 2026-11-26
updated: 2026-11-26
aliases: [triage, AI classification, sender enrichment, Freckle, conductor and instruments]
---

# Message Triage and Sender Research

Layer 3 of the [[System Map]] (workers), built on the [[Form Intake Pipeline]]. Whenever someone writes through the website's contact form, an AI reads the message and labels what it is, and for the right senders a research service finds out who they are. GTM-Engine [epic #43](https://github.com/galliot-us/GTM-Engine/issues/43) holds the work; this note holds the reasoning.

## The mental model: a conductor and its instruments
The worker is a **conductor**. It doesn't do everything itself; it calls separate services, each of which plays one part:

```
                          ┌──▶ the database   (store and read)
the worker (our rules) ───┼──▶ Claude         (label the message)
                          ├──▶ Gmail          (email the owner)
                          └──▶ Freckle        (research the sender): its own system, its own servers, its own workflow
```

The services don't live inside the worker. **We hand a service a job and get an answer back**; the *rules about what to do with the answer* are ours. That separation is [[Engine vs Core]] again.

## What happens to a message
1. The **spam rules** run first (cheap, instant, no outside call).
2. For a non-spam contact message, **Claude labels it**: `inquiry` (wants to hire us), `promotion` (selling to us), `acquisition` (a buyer or family office), or `other`.
3. **Only a confident promotion is kept out of the inbox.** Everything else is emailed with the label in the subject.
4. For an inquiry or acquisition from a sender whose email is **not** a free provider, the worker asks Freckle to research the sender, waits up to three minutes, and emails once, with or without the research.
5. Promotions are not forgotten: a **weekly summary** email lists them.

## Decisions, with the reasons
> [!info] Fail open
> **A missed inquiry costs far more than a shown promotion.** When the costs of two mistakes are unequal, you design so the *cheap* mistake is the one that happens. So anything unusual (a model error, a weak answer, a missing key) ends with the owner seeing the message. An *unsure* promotion is treated as an inquiry. This is why promotions are the only thing ever withheld, and only when the model is confident. The confidence threshold (0.8) is a **guess until measured**; the evaluation script exists to replace the guess with evidence.

- **The answer is forced into a fixed list of labels** (a schema), not free text. Combined with fencing the message as *untrusted data*, this limits what a malicious email can do to "mislabel one message". It can't make the system do anything, because the model is given no tools and its answer can only be one of four words. **You limit the damage of an attack by limiting what the thing it fools is able to do.** The same idea sits behind [[Bot Protection]].
- **The model sees only the message and the sender's email domain.** The name and full address stay home. Data you never send can't leak.
- **The label is saved as columns on the submission, and also recorded as an event** ([[Events vs State]]): the columns are the *present* (what is it now), the event is the *history* (it was labelled at this time by this model). The event holds no personal data.
- **Model choice (Haiku) is a bet, not a fact.** It's cheap and fast; whether it's good enough on subtle cases is **measured on a set of realistic messages** (including a disguised pitch, a prompt-injection attempt, and an inquiry that mentions "SEO" as the *project*) before it is trusted.

## One task or several? A rule of thumb
> [!warning] A retry re-runs the whole task
> When a step fails, the engine runs the whole task again. In one big task, a failure at the *last* step (the email) can repeat the *earlier* steps. That's harmless for cheap steps, but two of ours **cost money** (the model call, the research credits) and one is **irreversible** (an email). Repeating a side effect is the real danger, and it's the same lesson as [[Idempotency]] and [[Source Routing#Side effects]].

There are two ways to protect against it:
- **A guard:** before a paid step, check "did I already do this?" (triage reuses its saved label this way).
- **A boundary:** put the step in its **own task**, so it retries on its own and is never repeated because something *else* failed.

**Decision: split only where a step differs in kind.** Triage is quick, cheap and guarded, so it stays inside the main task. The research step waits minutes, spends credits and depends on an outside system's mood, so it becomes **one child task**. That is *orchestration* (a conductor calls the part and waits for the result), the simpler of the two styles in [[Worker Coordination]]. The parent treats a failed or slow child as "no research" and carries on, so an outside outage can never block the email.

## Talking to a service that works in the background
Freckle's research runs on its own servers and takes a while, so we can't just call it and read the answer. The pattern is common and worth recognising:
1. **Hand over the job** (a webhook: we post a small message to a private address).
2. **The service works and spends its own time.**
3. **It calls us back** with the result, and we store it.
4. Meanwhile **we wait with a deadline**, checking whether the result has arrived. If the deadline passes we proceed without it, and a late answer is still saved and shown later.

The private address is treated like a password: it lives only in a secret store, never in the code.

## The weekly summary
A **scheduled task** reads the week's stored promotions and sends one email, once per week (it records that it did, so a rerun doesn't send a second). No model call. Nothing is ever invisible; it's just batched.

## Open
- **The 0.8 threshold and the model choice**: settled by the evaluation, not by opinion ([[Open Threads]]).
- **Should the email wait for research at all?** It adds a failure mode. Sending at once and showing the research in the dashboard is simpler; we chose to wait because the owner wants it in the email.

Related: [[Dashboard Architecture]] (where the labels and research are shown), [[Two Log Streams]]. Back to [[GTM OS - Map of Content]].
