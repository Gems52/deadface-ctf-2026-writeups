# After the Lock

> **Category:** Database / SQL · **Points:** 150 · **Author:** @syyntax · **Track:** [Database / SQL](README.md)

## The Scenario

In a clinical trial, once a study's database is "locked" at a fixed date, any edit made after that date is a serious protocol violation, clear evidence of data tampering. A live MySQL/MariaDB shell is given, and the task is to find the trial subject whose primary efficacy result was edited after the official lock date for the Veltrexa study.

## Step 1: Explore the schema before writing any query

Jumping straight to a complex query without understanding the tables is a common beginner mistake. The very first thing to do in any SQL challenge is simply look around:

```sql
aurora> SHOW DATABASES;
-- aurora_trials, aurora_ehr, aurora_pharmacy, ...

aurora> USE aurora_trials;
aurora> SHOW TABLES;
-- studies, subjects, results, data_changes, ...
```

`DESCRIBE` each table to see its columns. This reveals that `studies` stores a `data_lock_date`, `subjects` links a `subject_code` to a study, and `results` stores each measurement together with a `modified_at` timestamp, everything needed for the task, no extra joins required.

## Step 2: Find the right study

```sql
SELECT study_id, drug_name, data_lock_date
FROM studies
WHERE drug_name LIKE '%Veltrexa%';
-- study_id = 1, data_lock_date = 2025-12-01
```

## Step 3: Join results to subjects and studies, then compare dates

The core idea: for every result row belonging to this study whose measure is the primary efficacy endpoint, check whether its `modified_at` timestamp comes after the study's own `data_lock_date`.

```sql
SELECT sub.subject_code, r.measure, r.modified_at, s.data_lock_date
FROM results r
JOIN subjects sub ON r.subject_id = sub.subject_id
JOIN studies s ON sub.study_id = s.study_id
WHERE s.study_id = 1
  AND r.measure = 'primary_efficacy_delta'
  AND r.modified_at > s.data_lock_date;
```

```
+--------------+-------------------------+---------------------+----------------+
| subject_code | measure                 | modified_at          | data_lock_date |
+--------------+-------------------------+---------------------+----------------+
| VELT-0137    | primary_efficacy_delta  | 2025-12-06 23:14:00  | 2025-12-01     |
+--------------+-------------------------+---------------------+----------------+
```

A single, unambiguous row came back: subject **VELT-0137** had their primary efficacy result edited five days after the study was supposed to be frozen.

## Flag

```
deadface{VELT-0137}
```

> **Lesson:** timestamp columns are often the quiet trail of evidence in a database; comparing them against a business rule (here, a lock date) is enough to catch the tampering.

---
[← Back to index](../../README.md)
