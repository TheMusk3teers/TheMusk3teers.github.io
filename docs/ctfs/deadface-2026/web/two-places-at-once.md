---
title: Two Places at Once
tags:
  - DEADFACE CTF 2026
  - Web
  - SQL
---

# Two Places at Once

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Database / SQL (Aurora Health) |
| **Points** | 450 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Aurora Health's incident-response team knows what DEADFACE did. They still don't know how they got in — until DEADFACE bragged about it. One legitimate account, used from two places at once. Find the compromised account — the one whose login shows up on two different workstations at the very same timestamp, over and over. Report that account's `username`.
>
> Submit the username as `deadface{username}`. Example: `deadface{jsmith}`.

## Solution

`aurora_audit.access_log` has `user_id`, `workstation` and `accessed_at`. "Two places at once" means the same user and the same timestamp with more than one workstation. Group on that, then count how often it happens per user:

```sql
USE aurora_audit;

SELECT
  u.username,
  COUNT(*) AS collision_timestamps
FROM (
  SELECT user_id, accessed_at
  FROM access_log
  GROUP BY user_id, accessed_at
  HAVING COUNT(DISTINCT workstation) > 1
) AS dup
JOIN users AS u ON dup.user_id = u.id
GROUP BY u.username
ORDER BY collision_timestamps DESC;
```

```text
+----------+----------------------+
| username | collision_timestamps |
+----------+----------------------+
| kbhatt   |                    8 |
+----------+----------------------+
```

Only one account shows up: `kbhatt`, eight times.

## Flag

!!! flag "Flag"
    `deadface{kbhatt}`

## Takeaways

`GROUP BY user, time HAVING COUNT(DISTINCT workstation) > 1` is worth keeping around for spotting shared or stolen credentials in logs.
