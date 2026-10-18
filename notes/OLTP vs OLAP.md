---
tags: [gtm-os, foundations, concept]
status: evergreen
created: 2026-10-15
updated: 2026-10-18
aliases: [OLTP, OLAP, row vs column storage]
---

# OLTP vs OLAP

- **OLTP — Online Transaction Processing.** Many tiny reads and writes, one record at a time. Needs **ACID** guarantees. Stores data **row-oriented**. → Postgres, MySQL, SQL Server, Supabase.
- **OLAP — Online Analytical Processing.** Few huge reads across millions of rows. Stores data **column-oriented**. → Snowflake.

All of them speak SQL and look relational. The differentiator is the **physical storage and execution strategy**.

**History:** OLAP used to mean pre-computed **data cubes** (slice, dice, drill down, roll up; SQL Server Analysis Services), built overnight because live queries were too slow. Columnar warehouses made explicit cubes largely obsolete.

## The key mental correction
**Never pick the engine from how the data arrives. Pick it from the questions you ask and the volume you ask them against.**

GTM events are born as OLTP-shaped writes, but a dashboard asks OLAP-shaped questions. The volume is modest, so Postgres serves it fine. See [[Choosing the Store]].
