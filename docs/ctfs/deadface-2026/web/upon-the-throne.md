---
title: Upon the Throne
tags:
  - DEADFACE CTF 2026
  - Web
  - SQL
---

# Upon the Throne

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Database / SQL (Aurora Health) |
| **Points** | 120 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Pull the threads together. Aurora's breach was not random: an insider clerk snooped a gubernatorial candidate's chart to build kompromat, and a diversion-flagged physician was coerced into faking the Veltrexa trial that would decide a drug's approval. Both insiders were being run by the same DEADFACE handler, whose codename was recovered in a compromised-account field (`handler_ref`) on each side of the house: the audit-system `users` and the EHR `providers`.
>
> Many handler codenames appear across those two tables — around fifteen on each side, so there is no obvious one to guess — but only one appears in both `users` and `providers`: the single operative managing an insider on each side. Find that codename.
>
> Submit the codename as `deadface{CODENAME}`. Example: `deadface{JOKER-4}`.

## Solution

`users` is in `aurora_audit` and `providers` is in `aurora_ehr`, but they're on the same MySQL server, so you can join them directly by prefixing the database name:

```sql
SELECT DISTINCT u.handler_ref
FROM aurora_audit.users AS u
JOIN aurora_ehr.providers AS p
  ON p.handler_ref = u.handler_ref
WHERE u.handler_ref IS NOT NULL;
```

```text
+-------------+
| handler_ref |
+-------------+
| REVENANT-7  |
+-------------+
1 row in set (0.00 sec)
```

One row, one handler.

## Flag

!!! flag "Flag"
    `deadface{REVENANT-7}`

## Takeaways

You don't need to `USE` a database to query it. `database.table` works across databases on the same server.
