---
title: Name to a Face
tags:
  - DEADFACE CTF 2026
  - Forensics
  - Traffic
---

# Name to a Face

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics / Traffic (pcap) |
| **Points** | 10 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> A poisoned name answer prompted an SMB authentication attempt. Which username
> did the workstation send?
>
> Submit the flag as `deadface{username}`.

Files given: the packet capture from *Uninvited Guest* (`demonne.pcap`), the network side of the same De Monne Financial incident as the [Who Goes There?](who-goes-there.md) memory series.

## Solution

The setup: a rogue Kali box (`10.77.10.66`) on De Monne's LAN running a Responder-style LLMNR/NBT-NS poisoner.

### The poisoning

Filter on `llmnr || nbns`. The workstation `DMF-WS11-042` (`10.77.10.20`) keeps asking for `dmf-field-01`, a name that doesn't exist in DNS, and `10.77.10.66` answers every time saying it's that host:

```
10.77.10.66 → 10.77.10.20   LLMNR response   dmf-field-01  A  10.77.10.66
10.77.10.66 → 10.77.10.20   NBT-NS response  DMF-FIELD-01  →  10.77.10.66
```

### The authentication

The workstation believes it and opens SMB (445) to the attacker. The handshake runs over link-local IPv6, because the poisoner answered the AAAA lookups too. So if you filter only on the attacker's IPv4 address, you'll miss it.

The username is in the NTLMSSP Authenticate message:

```
ntlmssp.messagetype == 3
```

```
fe80::…(DMF-WS11-042) → fe80::…(attacker):445
  NTLMSSP_AUTH
    Domain : dmf-field-01
    User   : nkessler
    Host   : DMF-WS11-042
```

So the workstation sent `nkessler`, the user logged in on it. That matches [Who Goes There?](who-goes-there.md). (DEADFACE then uses the account against the real file server `DMF-FILE-01` at `10.77.10.30`, but that's a later question.)

### Without Wireshark

```python
import struct
from scapy.all import PcapReader, IP, IPv6, TCP

def user(b):                       # parse NTLMSSP Type 3 user field
    if struct.unpack_from("<I", b, 8)[0] != 3: return None
    ln, _, off = struct.unpack_from("<HHI", b, 36)
    return b[off:off+ln].decode("utf-16-le", "replace")

for p in PcapReader("demonne.pcap"):
    raw = bytes(p)
    i = raw.find(b"NTLMSSP\x00")
    if i != -1 and (u := user(raw[i:])):
        ip = p.getlayer(IP) or p.getlayer(IPv6)
        print(ip.src, "->", ip.dst, "user =", u)
```

## Flag

!!! flag "Flag"
    `deadface{nkessler}`

## Takeaways

The username is in the `User` field of the NTLMSSP Type 3 message. When the poisoner also answers AAAA queries, the SMB session can run over IPv6, so don't filter on IPv4 only.
