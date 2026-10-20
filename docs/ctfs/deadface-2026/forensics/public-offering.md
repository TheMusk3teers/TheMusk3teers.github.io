---
title: Public Offering
tags:
  - DEADFACE CTF 2026
  - Forensics
  - Traffic
---

# Public Offering

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics / Traffic (De Monne Breach) |
| **Points** | 75 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Kali sent a ZIP archive outside the corporate network. What public IPv4 address and TCP destination port received the archive?
>
> Submit the flag as `deadface{IP_PORT}`.
>
> Use the PCAP from *Uninvited Guest*.

Files given: `demonne.pcap`

## Solution

From [Borrowed Script](borrowed-script.md) and [Crown Jewels](crown-jewels.md) we knew the Kali box is `10.77.10.66` and that it pulled about 725 KB of documents off `\\10.77.10.30\executive`.

To find where it went, we looked for a ZIP header (`PK\x03\x04`) in anything Kali sent:

```bash
tshark -r demonne.pcap -Y "ip.src == 10.77.10.66 && tcp.payload contains 50:4b:03:04" \
  -T fields -e ip.src -e tcp.srcport -e ip.dst -e tcp.dstport
```

One hit. A raw TCP connection from `10.77.10.66:34174` to `167.172.132.224:4815`, carrying around 710 KB of ZIP. The first entry in it is `annual_report.pdf`.

That's the same IP and port the persistence script keeps testing in [False Telemetry](false-telemetry.md).

## Flag

!!! flag "Flag"
    `deadface{167.172.132.224_4815}`

## Takeaways

Search the payloads for file magic bytes (`50:4b:03:04` for ZIP). It works even when the exfil runs over a raw socket that Wireshark can't decode.
