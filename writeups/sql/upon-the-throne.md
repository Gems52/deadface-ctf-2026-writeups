# Upon the Throne

> **Category:** SQL / Database Forensics · **Points:** 120 · **Track:** [Database / SQL](README.md)

## Objective

Access is given to a MariaDB server hosting several DEADFACE-related databases. The challenge asks to identify a specific individual by cross-referencing records that are split across two separate databases on the same server.

## Step 1: Mapping the databases

```sql
MariaDB> SHOW DATABASES;
-- aurora_audit, aurora_ehr, aurora_pharmacy, aurora_trials,
-- information_schema, performance_schema
```

Two databases stood out: `aurora_audit` (containing a `users` table) and `aurora_ehr` (containing a `providers` table). Describing both tables showed they shared a common linking column, `handler_ref`.

## Step 2: Cross-database join

MariaDB allows fully-qualified `database.table` references, making a join across two separate schemas possible in a single query:

```sql
SELECT u.*, p.*
FROM aurora_audit.users AS u
JOIN aurora_ehr.providers AS p
  ON u.handler_ref = p.handler_ref;
```

The join resolved to a single matching record, identifying the handler's alias.

## Answer

The handler's alias resolved by the join: `REVENANT-7`

## Flag

```
deadface{REVENANT-7}
```

---
[← Back to index](../../README.md)
