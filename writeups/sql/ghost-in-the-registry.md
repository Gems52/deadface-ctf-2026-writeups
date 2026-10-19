# Ghost in the Registry

> **Category:** Database / SQL · **Points:** 240 · **Author:** @syyntax · **Track:** [Database / SQL](README.md)

## The Scenario

Investigators suspect a fabricated patient record is being used to divert controlled substances: a "ghost" patient who receives Schedule II prescriptions but has never actually had a single clinical visit. Since the hospital's systems share one enterprise `patient_id` across every database, records can be joined across the pharmacy and the clinical record systems to catch the inconsistency.

## Step 1: Explore both databases involved

```sql
aurora> USE aurora_pharmacy;
aurora> SHOW TABLES;
-- prescriptions, drugs, ...

aurora> USE aurora_ehr;
aurora> SHOW TABLES;
-- patients, encounters, ...
```

`DESCRIBE` of each relevant table shows: `drugs.dea_schedule` stores the DEA controlled-substance schedule (e.g. `'II'`), `prescriptions` links a `patient_id` to a `drug_id`, `encounters` records every clinical visit by `patient_id`, and `patients` stores the human-readable `mrn` (Medical Record Number) to report.

## Step 2: One query, two databases

Because MySQL lets us reference tables from different databases in the same query using the `database.table` syntax, there is no need to export or combine anything manually; a single query joins across `aurora_pharmacy` and `aurora_ehr` directly. The logic: find every patient with at least one Schedule II prescription who does **not** appear anywhere in the `encounters` table.

```sql
SELECT DISTINCT p.patient_id, pt.mrn, pt.first_name, pt.last_name
FROM aurora_pharmacy.prescriptions p
JOIN aurora_pharmacy.drugs d ON p.drug_id = d.drug_id
JOIN aurora_ehr.patients pt ON p.patient_id = pt.patient_id
WHERE d.dea_schedule = 'II'
  AND p.patient_id NOT IN (
    SELECT patient_id FROM aurora_ehr.encounters
    WHERE patient_id IS NOT NULL
  );
```

```
+------------+------------+------------+-----------+
| patient_id | mrn        | first_name | last_name |
+------------+------------+------------+-----------+
| 1060       | AH-0090001 | Dana       | Kessler   |
+------------+------------+------------+-----------+
```

Exactly one patient matched both conditions: **Dana Kessler**, MRN `AH-0090001`, a patient with controlled-substance prescriptions on file but zero recorded clinical encounters. This is precisely the "ghost patient" pattern described in the challenge: a fabricated identity used purely to divert medication.

## Flag

```
deadface{AH-0090001}
```

> **Lesson:** a `NOT IN` subquery across two related tables, even across two separate databases, is a simple and powerful way to find records that should logically never exist together.

---
[← Back to index](../../README.md)
