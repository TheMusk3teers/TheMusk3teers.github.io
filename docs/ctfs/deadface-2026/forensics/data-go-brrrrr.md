---
title: Data Go Brrrrr
tags:
  - DEADFACE CTF 2026
  - Forensics
  - Traffic
---

# Data Go Brrrrr

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Traffic analysis |
| **Points** | 25 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> We captured traffic from an endpoint suspected of leaking sensitive data. Analyze the capture and recover the exfiltrated flag.
>
> Submit the flag as `deadface{flag_text}`.

Files given: `PCAP01.pcap`

## Solution

Most of the capture is normal background noise. The odd thing is a run of ICMP echo requests whose data is 8 bytes of ASCII, not the usual ping filler.

We pulled the data out of every echo request, joined it, and got Base64:

```python
import base64
from scapy.all import rdpcap, ICMP, Raw

pkts = rdpcap("PCAP01.pcap")
chunks = [
    bytes(p[Raw]).decode("ascii", errors="ignore")
    for p in pkts
    if p.haslayer(ICMP) and p[ICMP].type == 8 and p.haslayer(Raw)
]
b64_str = "".join(chunks)
print("Base64:", b64_str)
print("Decoded:", base64.b64decode(b64_str).decode())
```

```text
deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}
```

## Flag

!!! flag "Flag"
    `deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}`

## Takeaways

If HTTP and DNS look clean, check what's inside the pings (`icmp.type == 8`).
