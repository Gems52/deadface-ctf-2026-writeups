# Too Good to be True

> **Category:** SQL / Statistical Analysis · **Points:** 310 · **Track:** [Database / SQL](README.md)

## Objective

The `aurora_trials` database holds clinical-trial data for a drug called **Veltrexa**. Somewhere in the results, one subject's recorded value looks artificially inflated compared to everyone else. The challenge is to find that outlier record.

## Step 1: Scoping the data

```sql
SHOW TABLES; -- adverse_events, data_changes, results, sites, studies, subjects
SELECT * FROM studies; -- study_id = 1 -> "Veltrexa"
SELECT DISTINCT measure, visit FROM results;
-- measures: primary_efficacy_delta, secondary_qol_score
-- visit: Week 24 (only timepoint recorded)
```

## Step 2: Establishing a statistical baseline

With a single measure and visit to compare, the natural baseline is the sample mean and standard deviation of that measure across all subjects:

```sql
SELECT AVG(value) AS mean_val, STDDEV(value) AS std_val
FROM results
WHERE measure = 'primary_efficacy_delta' AND visit = 'Week 24';
-- mean_val = 2.965, std_val = 2.8067
```

## Step 3: Flagging the outlier

A value more than 3 standard deviations above the mean is a standard statistical threshold for an outlier (`mean + 3σ ≈ 11.39`):

```sql
SELECT subject_id, value
FROM results
WHERE measure = 'primary_efficacy_delta'
  AND visit = 'Week 24'
  AND value > (
    SELECT AVG(value) + 3 * STDDEV(value)
    FROM results
    WHERE measure = 'primary_efficacy_delta' AND visit = 'Week 24'
  );
```

Exactly one record cleared the threshold, with a value of **22.00** against a mean of ~3, the falsified result.

## Answer

The falsified subject record: `VELT-0148`

## Flag

```
deadface{VELT-0148}
```

---
[← Back to index](../../README.md)
