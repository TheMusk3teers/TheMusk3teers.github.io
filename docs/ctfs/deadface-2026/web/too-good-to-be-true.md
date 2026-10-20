---
title: Too Good to be True
tags:
  - DEADFACE CTF 2026
  - Web
  - SQL
---

# Too Good to be True

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Database / SQL (Aurora Health) |
| **Points** | 310 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> The post-lock edit wasn't the only fabrication in the Veltrexa trial. One subject's recorded efficacy improvement is a statistical impossibility — it sits more than three standard deviations above the study-wide mean for the primary endpoint, a telltale sign that a number was invented rather than measured. Identify that outlier subject and report their subject code.
>
> Submit the subject code as `deadface{subject_code}`. Example: `deadface{ABCD-0101}`.

## Solution

In `aurora_trials`, `results` has each subject's `primary_efficacy_delta` and links to `subjects` for the `subject_code`. We worked out the mean and standard deviation in a subquery and kept anyone above mean + 3σ:

```sql
USE aurora_trials;

SELECT
  s.subject_code,
  r.primary_efficacy_delta,
  stats.mean_val,
  stats.sd_val,
  (stats.mean_val + 3 * stats.sd_val) AS threshold_3sd
FROM results AS r
JOIN subjects AS s ON r.subject_id = s.id
CROSS JOIN (
  SELECT
    AVG(primary_efficacy_delta) AS mean_val,
    STDDEV_POP(primary_efficacy_delta) AS sd_val
  FROM results
) AS stats
WHERE r.primary_efficacy_delta > stats.mean_val + 3 * stats.sd_val;
```

The mean is 2.965 and the 3σ cutoff is 11.3851. Only one subject is over it: `VELT-0148`, with an improvement of 22. That's around 7σ out, so it's obviously made up.

## Flag

!!! flag "Flag"
    `deadface{VELT-0148}`

## Takeaways

MySQL has `AVG()` and `STDDEV_POP()`, so you can do a z-score check in one query instead of exporting to Python.
