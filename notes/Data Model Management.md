---
tags: [gtm-os, data-model, practices]
status: seed
created: 2026-10-28
updated: 2026-10-28
aliases: [schema management, data model best practices]
---

# Data Model Management

Best practices for keeping the [[Core Data Model]] healthy as it grows:

1. **Every schema change is a migration in Git.** Never edit tables by hand in the Supabase dashboard. Then you can always rebuild the database and see who changed what.
2. **Consistent conventions:** one naming style everywhere, your own ID on every table, and `created_at` / `updated_at` on every row.
3. **Let the database enforce the rules.** Required fields, uniqueness and links between tables are **constraints**, so bad data can't get in, whatever code is writing it.
4. **Soft deletes:** mark records as deleted instead of erasing them, so history stays intact ([[Events vs State]]).
5. **A planned place for extra fields.** Plugins will want custom fields; give them a flexible **JSON column** or an **extension table**, so core tables don't change for every add-on. This is how extensibility shows up in the data layer ([[System Map]]).
6. **Only your core code writes to core tables.** Tools and plugins go *through* it, never around it ([[Engine vs Core]]).
7. **A data dictionary:** a short written definition of every table and field.
8. **Protect personal data:** restrict who can read what. Supabase's **row-level security** does this.

> [!todo] Next
> Pick one of these to go deeper on. See [[Open Threads]].
