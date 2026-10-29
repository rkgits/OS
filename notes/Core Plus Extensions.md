---
tags: [gtm-os, data-model, architecture, decision]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [extension tables, plugin data, core plus extensions]
---

# Core Plus Extensions

**How a new add-on stores its own data without changing your core tables.** This is what makes "add more plugins later" work at the data layer ([[System Map]]).

## The case
Next year you add a plugin that scores leads from intent signals. For each person it needs a **score**, a **reason** and a **date**. Where does that live?

**Answer:** in the plugin's own table, an **extension table** (e.g. `intent_scores`), linked back to Person by person ID. Person itself stays untouched.

```
person (core)               intent_scores (owned by the intent plugin)
─────────────               ───────────────────────────────────────────
id  ◀───────────────────────  person_id
name                          score
…                             reason
                              scored_at
```

## Rules
1. **Name it after the plugin** (`intent_scores`, not `scores`), so ownership is obvious.
2. **Only that plugin writes to it.** The core and other plugins may read it.
3. **Removing the plugin = dropping its table.** The core never notices.
4. **Tiny, loose bits of data** can go in a flexible **JSON column** instead. Anything you'll query, filter or chart deserves a real table.
5. **Need it all in one place?** A database **view** joins core and extensions back into one table *for reading*, without merging them in storage.

## Why not just add columns to Person?
> [!warning] Correction
> The instinct "just add a score column to the person table" works at first. After ten plugins, the core table is a **junk drawer**.

1. **Ownership gets blurry.** With 60 columns on Person, nobody knows which plugin owns which field, or whether it's still used, so nobody dares delete anything.
2. **Changes get risky.** Every new plugin is a migration on your most important table, the one everything reads. One mistake breaks the whole system, not just the new feature.
3. **Plugins collide.** Two add-ons both wanting a `score` column step on each other. Separate tables give each its own lane.
4. **Removal becomes surgery.** Turning off a plugin should be "drop its table", not editing the core.
5. **The core loses meaning.** Person should describe a person. A plugin's *opinion about* that person is a different thing, so it lives somewhere else.

## The idea underneath
The same one as [[Engine vs Core]]: **a stable centre, with add-ons attached by a link, never welded in.** The pattern is called **core plus extensions**.

Related: [[Core Data Model]], [[Data Model Management]] (practice 5).
