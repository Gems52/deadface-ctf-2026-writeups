# Admitted Under Another Name

> **Category:** Database / SQL · **Points:** 15 · **Author:** @syyntax · **Track:** [Database / SQL](README.md)

## Scenario

DEADFACE leaked a hospital's ("Aurora Health") patient registry. Somewhere in that registry is one VIP patient who checked in under a fake name. Direct shell access to the database is given, and the task is to find that patient's real Medical Record Number (MRN).

## Step 1: Get your bearings in the database

When dropped into a database shell never seen before, the very first thing to do is figure out what databases and tables actually exist, not guess table names blindly.

```sql
SHOW DATABASES;
```

This returns several databases, including one named `aurora_ehr`, "EHR" standing for *Electronic Health Record*, exactly the kind of system that would store patient data.

```sql
USE aurora_ehr;
SHOW TABLES;
```

This reveals a `patients` table among others.

## Step 2: Inspect the table structure

Before writing any query, look at the column names with `DESCRIBE`:

```sql
DESCRIBE patients;
```

The output shows columns including `patient_id`, `mrn`, `first_name`, `last_name`, `alias`, and, crucially, `is_vip` (a flag column, 0 or 1).

> **Beginner tip:** when a table literally has a column called `is_vip` and the challenge is about finding a VIP patient, that's not a coincidence; the challenge wants a direct filter on it rather than a manual search through hundreds of rows.

## Step 3: Query for the VIP patient

```sql
SELECT patient_id, mrn, first_name, last_name, alias, is_vip
FROM patients
WHERE is_vip = 1;
```

```
+------------+------------+------------+-----------+---------------+--------+
| patient_id | mrn        | first_name | last_name | alias         | is_vip |
+------------+------------+------------+-----------+---------------+--------+
| 1007       | AH-0044719 | Gregory    | Halloran  | George Harmon | 1      |
+------------+------------+------------+-----------+---------------+--------+
```

This matches the scenario perfectly: the patient's real name is Gregory Halloran, but he checked in under the fake name / alias "George Harmon". His real MRN is `AH-0044719`.

## Step 4: Build the flag

The challenge asked for the format `deadface{MRN}`, for example `deadface{AH-1234567}`.

## Flag

```
deadface{AH-0044719}
```

---
[← Back to index](../../README.md)
