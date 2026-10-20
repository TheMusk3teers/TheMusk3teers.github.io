---
title: Car Crash
tags:
  - DEADFACE CTF 2026
  - OSINT
---

# Car Crash

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT |
| **Points** | 50 |
| **Author** | TODO |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Identify the company that owns the truck involved in the crash and determine the date the company was founded (`MM_DD_YYYY`).
>
> Submit the flag as `deadface{MM_DD_YYYY}`.

## Solution

Zoom in on the side of the truck cab in the crash photo and there's a USDOT number:

```text
USDOT 80806
```

We looked it up in the FMCSA SAFER Company Snapshot, and it belongs to J.B. Hunt Transport, Inc. in Lowell, Arkansas.

J.B. Hunt was founded by Johnnie Bryan Hunt and Johnelle Hunt on August 10, 1961.

## Flag

!!! flag "Flag"
    `deadface{08_10_1961}`

## Takeaways

US commercial trucks have to show their USDOT number on both sides. Put it into FMCSA SAFER and you get the carrier, even if you can't see a logo.
