---
title: Return to Sender
tags:
  - DEADFACE CTF 2026
  - Forensics
  - Traffic
---

# Return to Sender

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Traffic analysis |
| **Points** | 80 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Something on this network is stealing data and sneaking it out disguised as ordinary web traffic. You've been handed a packet capture from the affected machine. Buried, a compromised host is quietly shipping stolen data out to a server it shouldn't be talking to. Find the traffic that doesn't belong, figure out what's being smuggled, and piece it back together.
>
> Submit the flag as `deadface{flag_text}`.

Files given: `PCAP03.pcap` (63,104 bytes, SHA256: `23cc1638...a78bc`)

## Solution

The capture is 348 packets of HTTP. Almost all of it is GETs. The exceptions are five POSTs from `10.20.30.40` to `45.66.77.88` with `Host: telemetry.deadface-cc.net`.

Each POST body is a short Base64 string. We decoded and joined them:

```python
import base64, re

raw = open("PCAP03.pcap", "rb").read().decode("latin1")
posts = re.findall(r"POST [^\r\n]+ HTTP/1\.1\r\nHost: telemetry\.deadface-cc\.net.*?\r\n\r\n([A-Za-z0-9+/=]+)", raw, re.S)
session_ids = re.findall(r"X-Session-Id:\s*([A-Za-z0-9+/=]+)", raw)

prefix = b"".join(base64.b64decode(chunk) for chunk in posts)
suffix = base64.b64decode(session_ids[0])
print((prefix + suffix).decode())
```

The bodies only give `deadface{http_exfil_sneaky_`, with no closing brace. We went back to the headers on those POSTs, and `X-Session-Id` is Base64 too. It decodes to `business}`.

## Flag

!!! flag "Flag"
    `deadface{http_exfil_sneaky_business}`

## Takeaways

If the pieces you've rebuilt are missing the end, check the custom headers on the same requests (`X-Session-Id`, `Cookie` and the like).
