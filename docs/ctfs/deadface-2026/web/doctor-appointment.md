---
title: Doctor Appointment
tags:
  - DEADFACE CTF 2026
  - Web
---

# Doctor Appointment

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Web & SQL |
| **Points** | 26 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Web applications are tricky these days and we are seeing an influx of vibe coded platforms. Put your black hat on and see if you can gain access to this website. Don't just brute force a massive dictionary. Take a look at GhostTown for potential resources.

Target: `http://45.79.200.170:3000/`, a patient-portal login page.

## Solution

"Vibe coded" plus "don't brute force" read like a hint that the login code itself is broken. GhostTown had nothing useful for this site, so we went straight at the login form.

The page's JavaScript posts to `POST /api/login`. We tried the oldest trick there is:

```
username: ' OR '1'='1'--
password: anything
```

The server came back with HTTP 200, no session token, and an odd error: "Login failed. That's weird... check the network traffic!" So we did. The raw response in the browser's Network tab had a whole user row for `john.smith`, flag included.

We tried a few other payloads and always got `john.smith` back, so this looks like a canned response the challenge serves on any injection, not the query actually picking that user.

This flag is for part 1 of the HealthSpark series. The fax document challenge has its own flag.

## Flag

!!! flag "Flag"
    `deadface{typical_and_classic}`

## Takeaways

Throw `' OR '1'='1'--` at a login form before you open a wordlist. And if the page tells you to check the network traffic, read the raw response.
