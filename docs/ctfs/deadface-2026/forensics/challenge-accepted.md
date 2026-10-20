---
title: Challenge Accepted
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# Challenge Accepted

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics (PCAP) |
| **Points** | 100 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Find Nathan's first SMB NTLMv2 authentication after the poisoned name response. What is its 16-byte NTProofStr?
>
> Submit the flag as `deadface{16-byte string}`.

Files given: none new. It uses `demonne.pcap` from Uninvited Guest.

## Solution

An NTLMv2 response (in the NTLMSSP Type 3 / AUTHENTICATE message) starts with the 16-byte NTProofStr, followed by the client blob. We needed the first one sent after the attacker poisoned name resolution.

We had no tshark, so a short scapy script walked the capture in order and printed:

- every LLMNR (UDP 5355) and NBT-NS (UDP 137) response, and
- every NTLMSSP Type 3 message, with the user name and the first 16 bytes of its NT response.

```python
from scapy.all import rdpcap, Raw, IP, IPv6, TCP, UDP
import struct

pk = rdpcap('demonne.pcap')
for i, p in enumerate(pk, 1):
    if UDP in p and p[UDP].sport in (5355, 137):
        print(i, 'name response from', p[IPv6].src if IPv6 in p else p[IP].src)
    if TCP in p and Raw in p:
        d = bytes(p[Raw])
        j = d.find(b'NTLMSSP\x00\x03\x00\x00\x00')
        if j < 0: continue
        m = d[j:]
        ln, _, off = struct.unpack('<HHI', m[20:28])     # NtChallengeResponse
        ulen, _, uoff = struct.unpack('<HHI', m[36:44])  # UserName
        user = m[uoff:uoff + ulen].decode('utf-16le')
        print(i, user, p[TCP].sport, '->', p[TCP].dport, m[off:off + 16].hex())
```

What it shows:

- **Frames 1463–1476:** the first poisoned responses, from the Kali box over IPv6 (`fe80::5054:ff:fe77:6601`) and IPv4 (`10.77.10.66`) to the workstation `10.77.10.20`.
- **Frame 1489:** the first SMB NTLMv2 authentication after that, workstation → attacker on TCP 445 (source port 50692), user `nkessler`. NTProofStr `5a2342014fe7f28997ce4dcd09f6b7f8`.
- No NTLM authentication appears before frame 1463. Later frames are retries with new proofs (`edb05acc…` at 1532, `7d015b79…` at 1575), so 1489 is the first.

## Flag

!!! flag "Flag"
    `deadface{5a2342014fe7f28997ce4dcd09f6b7f8}`

## Takeaways

NTProofStr = the first 16 bytes of the NTLMv2 response. When asked for the "first" one, list every attempt in frame order; the victim retries several times.
