---
title: Prying Eyes
tags:
  - DEADFACE CTF 2026
  - Web
  - SQL
---

# Prying Eyes

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Database / SQL (Aurora Health) |
| **Points** | 55 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Aurora Health logs every time a staff member opens a patient chart. Someone was watching the VIP a little too closely. Find the user account that accessed the VIP patient's chart more times than any other account, and report their username.
>
> You'll need the VIP's MRN first from *Admitted Under Another Name*. The audit log references patients by that same MRN.
>
> Use the provided MySQL shell (`https://aurora.dctf.io/`).
>
> Submit the username as `deadface{username}`. Example: `deadface{jsmith}`.

## Solution

The VIP's MRN in `aurora_ehr.patients` is `AH-0044719`. The audit log only stores `user_id`, so join it to `users` and count chart opens per person:

```sql
SELECT u.username, COUNT(*) AS opens
FROM aurora_audit.access_log AS a
JOIN aurora_audit.users AS u
  ON a.user_id = u.id
WHERE a.patient_mrn = 'AH-0044719'
GROUP BY u.username
ORDER BY opens DESC;
```

| `username` | `opens` |
|---|---|
| `rborden` | 47 |
| `mreyes_ehr` | 5 |

The VIP has 58 log rows in total, and 47 of them are `rborden`.

## Flag

!!! flag "Flag"
    `deadface{rborden}`

## Takeaways

Most "who was snooping" questions on an audit log are one `GROUP BY` plus `ORDER BY COUNT(*) DESC`.
