---
title: Schedule of Concern
tags:
  - DEADFACE CTF 2026
  - Database
---

# Schedule of Concern

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Database |
| **Points** | 20 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Aurora Health's pharmacy formulary was in the DEADFACE leak. Before investigators can assess how exposed the controlled-substance supply is, they need a baseline: how many of the formulary's drugs are the most tightly regulated class — DEA Schedule II?
>
> Use the provided MySQL shell to count the Schedule II drugs in the drugs table.
>
> Submit the number as deadface{#}. Example: deadface{42}.

Given: a browser MySQL shell (ttyd) at `https://aurora.dctf.io/`, logged in as `player`.

## Solution

Small gotcha first: the "Access Hostbusters" link on the challenge page goes to `hostbusters.dctf.io`, which is another challenge. The MySQL shell for this one is `aurora.dctf.io`. You land at an `aurora>` prompt with no database selected:

```sql
SHOW DATABASES;
-- aurora_audit, aurora_ehr, aurora_pharmacy, aurora_trials, ...
USE aurora_pharmacy;
SHOW TABLES;
-- dispenses, drugs, pharmacists, prescribers, prescriptions
DESCRIBE drugs;
```

The column is `dea_schedule varchar(3)`. Before counting we wanted to see how the values were written, since data like this often mixes `II`, `2`, `C-II` and `CII`. The brackets and `HEX()` would show stray spaces:

```sql
SELECT CONCAT('[',dea_schedule,']') s, HEX(dea_schedule) h, COUNT(*)
FROM drugs GROUP BY dea_schedule;
```

| s | h | COUNT(*) |
|---|---|---|
| [II] | 4949 | 9 |
| [III] | 494949 | 4 |
| [IV] | 4956 | 5 |
| [V] | 56 | 2 |
| [N] | 4E | 12 |

Clean data. Schedule II only shows up as `II`, and the groups add up to 32, same as `SELECT COUNT(*) FROM drugs;`, so no NULLs either. `N` is non-controlled.

```sql
SELECT COUNT(*) FROM drugs WHERE dea_schedule = 'II';
-- 9
```

## Flag

!!! flag "Flag"
    `deadface{9}`

## Takeaways

`GROUP BY` the column before you count so you see every spelling of a value at once. "No database selected" means `SHOW DATABASES;` and then `USE`.
