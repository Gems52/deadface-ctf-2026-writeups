# Two Places at Once

> **Category:** SQL / Anomaly Detection · **Points:** 450 · **Track:** [Database / SQL](README.md)

## Objective

An `access_log` table records badge and workstation logins across a facility. The task is to find a user who was recorded logging in from **more than one workstation at the exact same timestamp**, a physical impossibility that points to a shared or cloned credential.

## Step 1: Inspecting the schema

```sql
DESCRIBE access_log;
-- log_id | user_id | workstation | access_time | ...
```

## Step 2: Detecting simultaneous logins

Grouping by timestamp and counting the number of distinct workstations used at that exact moment surfaces any login that happened in two places at once:

```sql
SELECT user_id, access_time, COUNT(DISTINCT workstation) AS station_count
FROM access_log
GROUP BY user_id, access_time
HAVING COUNT(DISTINCT workstation) > 1;
```

One user (`user_id = 16`) stood out, with **8 separate occurrences** of simultaneous logins from different workstations, far beyond coincidence or clock drift.

## Step 3: Identifying the user

```sql
SELECT username FROM users WHERE user_id = 16;
-- kbhatt
```

## Answer

The username tied to the duplicated badge logins: `kbhatt`

## Flag

```
deadface{kbhatt}
```

---
[← Back to index](../../README.md)
