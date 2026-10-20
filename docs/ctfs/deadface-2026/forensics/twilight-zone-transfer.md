---
title: Twilight Zone Transfer
tags:
  - DEADFACE CTF 2026
  - Forensics
  - Traffic
---

# Twilight Zone Transfer

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Traffic analysis |
| **Points** | 50 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> A misconfigured DNS server allowed an unauthorized zone transfer (AXFR). The attacker enumerated hostnames, then pulled the ENTIRE zone in one request. The capture already contains that transfer, find it and read the records.
>
> Submit the flag as: `deadface{flag-text}`

Files given: `PCAP04.pcap` (~4.36 KB, SHA256: `5c4b3bf978bcde5e7813e37be93e85b9374bd7874cc1563a5402fcd7934d4483`)

!!! note "Wrong file linked"
    At first the challenge page linked `PCAP03.pcap`, which has no DNS in it at all. The file that matches the published size and hash is `PCAP04.pcap`.

## Solution

AXFR is query type 252, so filter on that:

```bash
tshark -r PCAP04.pcap -Y "dns.qry.type == 252" -V
```

The response dumps all 14 records for `deadface-cc.net`. One of them is a TXT record on `_flag`:

```text
_flag.deadface-cc.net: type TXT, class IN, txt "deadface{dns_zone_transfer}"
```

## Flag

!!! flag "Flag"
    `deadface{dns_zone_transfer}`

## Takeaways

Check the hash of what you downloaded. We spent a while looking for DNS in the wrong PCAP before noticing.
