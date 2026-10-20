---
title: Heavy Hand
tags:
  - DEADFACE CTF 2026
  - Web
  - SQL
---

# Heavy Hand

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Database / SQL (Aurora Health) |
| **Points** | 100 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> One Aurora prescriber dispensed far more Schedule II medication than anyone else in the final quarter of 2025 — the classic signature of drug diversion. Using the `aurora_pharmacy` database, find the prescriber with the highest total quantity of Schedule II medication dispensed between `2025-10-01` and `2025-12-31`, and report their DEA number.
>
> Use the provided MySQL shell (`https://aurora.dctf.io/`).
>
> Submit the DEA number as `deadface{DEA}`. Example: `deadface{AB1234567}`.

## Solution

`dispenses` points to `prescriptions`, and each prescription points to a drug (which has `dea_schedule`) and a prescriber (which has `dea_number`). Join all four, keep Schedule II in Q4 2025, and sum the quantity per prescriber:

```sql
USE aurora_pharmacy;

SELECT
  pr.dea_number,
  CONCAT(pr.first_name, ' ', pr.last_name) AS prescriber,
  SUM(di.quantity_dispensed) AS total_qty,
  COUNT(*) AS dispenses
FROM dispenses AS di
JOIN prescriptions AS rx ON di.prescription_id = rx.id
JOIN drugs AS d ON rx.drug_id = d.id
JOIN prescribers AS pr ON rx.prescriber_id = pr.id
WHERE d.dea_schedule = 'II'
  AND di.dispensed_at >= '2025-10-01'
  AND di.dispensed_at < '2026-01-01'
GROUP BY pr.id, pr.dea_number, prescriber
ORDER BY total_qty DESC
LIMIT 5;
```

| `dea_number` | `prescriber` | `total_qty` | `dispenses` |
|---|---|---|---|
| `BR2938471` | Marcus Reyes | 5220 | 29 |
| `BR2340466` | Dorothy Hill | 150 | 3 |
| `RZ6578160` | Michelle Wilson | 90 | 3 |

Not close. Marcus Reyes dispensed 5,220 units, about 35 times the next prescriber.

## Flag

!!! flag "Flag"
    `deadface{BR2938471}`

## Takeaways

For a date range on a timestamp column, use `>= start AND < the day after the end`. `BETWEEN '2025-10-01' AND '2025-12-31'` quietly drops anything after midnight on Dec 31.
