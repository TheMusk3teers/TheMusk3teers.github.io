---
title: "A Pirate's Life for Me: Part 1"
tags:
  - DEADFACE CTF 2026
  - Forensics
  - Traffic
---

# A Pirate's Life for Me: Part 1

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Traffic analysis |
| **Points** | 25 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

A packet capture from a university café's Wi-Fi. Someone there used an unfamiliar browser ("maybe it was just light mode"). Find that device and follow its traffic to the flag.

Files given: `pirateslife.7z` (SHA256 `7bcebd81928b2d01718f2f853c507ea313a44e2b19037f4b7d0cc1c638affe93`). It unpacks to `pirateslife.pcap`, a 565 MB pcapng file with about 445k packets covering 13 minutes.

## Solution

### Finding the odd browser

Most of the capture is TLS and QUIC, but there are 28 cleartext HTTP packets. The hint points at the browser, so we listed every HTTP User-Agent:

```bash
tshark -r pirateslife.pcap -Y http.request -T fields -e ip.src -e http.user_agent | sort | uniq -c
```

```
     13 172.16.14.43	SunstalkerDFBrower/2.0 (Linux; Android 14)
      1 172.16.14.43	curl/8.18.0
```

`SunstalkerDFBrower/2.0` (typo included) is not a real browser. It belongs to **172.16.14.43**. The DHCP traffic gives hostnames for the other café clients (AnneDroids, Maddies-iPhone, drtom and so on), but this device never sends one.

### Following its requests

We listed its HTTP requests with their full header lines:

```bash
tshark -r pirateslife.pcap -Y 'http.request && ip.src==172.16.14.43' \
  -T fields -e frame.number -e tcp.stream -e http.host -e http.request.line
```

Most of them are ordinary page loads of `nvucafe.com`, `nightvaleuniversity.edu`, `aurorahhs.com`, `birdsarejustbirds.gov` and `neverssl.com`, with normal `Accept`, `If-None-Match` and similar headers. Three requests stand out. They send only `Host`, `User-Agent`, `Accept: */*` and one extra header that no browser would add:

| Frame | Stream | Host | Extra header |
|---|---|---|---|
| 19114 | 97 | nightvaleuniversity.edu | `X-1-Trace-Part: deadface{X-M4rk5_` |
| 43516 | 169 | aurorahhs.com | `X-2-Mark: th3_sp0t_` |
| 56543 | 224 | birdsarejustbirds.gov | `X-3-Spot: I_gu355}` |

The number in each header name gives the order. Joined together, the three values form the flag. The flag text itself ("X marks the spot") points at the `X-` headers.

`tshark -q -z follow,tcp,ascii,97` shows the whole exchange for the first part:

```
GET / HTTP/1.1
Host: nightvaleuniversity.edu
User-Agent: SunstalkerDFBrower/2.0 (Linux; Android 14)
Accept: */*
X-1-Trace-Part: deadface{X-M4rk5_
```

The responses are just the "Hacked by DEADFACE" defacement pages. The flag is only in the requests.

### Dead ends

- The large TLS sessions from this host (for example 489 MB to 172.64.145.29) can't be decrypted, because no key log is provided. They aren't needed.
- The curl request to `speakfriend.dctf.io` gets a 301 redirect to HTTPS and contains nothing useful.
- Exporting HTTP objects (`--export-objects http,out/`) only returns the defacement pages and the café's landing page.

## Flag

!!! flag "Flag"
    `deadface{X-M4rk5_th3_sp0t_I_gu355}`

## Takeaways

When a hint points at a client, filter on that client and read its full request headers, not just the URLs. Custom `X-` headers are an easy place to hide data in HTTP traffic. `http.request.line` prints every header of a request in one field, which makes them easy to spot.
