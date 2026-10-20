---
title: The Workstation in the Middle
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# The Workstation in the Middle

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics (memory) |
| **Points** | 40 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> The compromised De Monne employee was working at a Windows device. What is that device's hostname? Preserve the capitalization shown in memory.

Files given: the `memory.raw` image from [Who Goes There?](who-goes-there.md).

## Solution

We already had the Volatility 3 output from the previous challenge, so this was a matter of searching it. The hostname appears in several places, all with the same capitalization:

**1. Environment variables.** `windows.envars` on nkessler's processes, including the attacker's SSH shell:

```
3532  sshd.exe  COMPUTERNAME  DMF-WS11-042
3532  sshd.exe  PROMPT        nkessler@DMF-WS11-042 $P$G
```

**2. Process owners.** `windows.sessions` shows the user as `DMF-WS11-042/nkessler`, and system services run as the machine account `WORKGROUP/DMF-WS11-042$`.

The name fits De Monne's naming scheme: **DMF** (De Monne Financial), **WS11** (Windows 11 workstation), **042** (asset number). The machine is at `10.77.10.20` and runs Windows 11 22H2. The attacker reached it over SSH from `10.77.10.66`.

## Flag

!!! flag "Flag"
    `deadface{DMF-WS11-042}`

## Takeaways

Save the plugin output to text files on the first memory challenge. Later challenges in the series become a grep instead of a new Volatility run.
