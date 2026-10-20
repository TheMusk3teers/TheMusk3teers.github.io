---
title: Exposing Myself Through Bad OPSEC
tags:
  - DEADFACE CTF 2026
  - OSINT
  - Web
---

# Exposing Myself Through Bad OPSEC

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT / Web |
| **Points** | 25 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Now that you've found the DEADFACE member's travel blog, what else can we determine about this site and this member? Perhaps the site's certificate can tell us more.
>
> The IP hosts other challenge ports; only the website at 8443 is relevant.

Target: https://170.187.156.131:8443/

## Solution

The [Ghost Town travel thread](https://ghosttown.deadface.io/t/20/travel-destinations) ties the blog to lamia415 and says the site shows the poster's first name. The challenge tells you to look at the certificate, so we did:

```bash
openssl s_client -connect 170.187.156.131:8443 -showcerts </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

It's self-signed (Subject and Issuer are the same), and the Subject is full of personal info:

- CN: `170.187.156.131`
- OU: `Operations`
- O: `Deadface{c@tch_m3_if_y0u_c@n}`
- L: `New Orleans`
- ST: `Louisiana`
- C: `US`
- emailAddress: `alicehankerty@gmail.com`

The SAN repeats the IP and email and adds `deadface.ctf`. The email gives the name Alice Hankerty, which matches the "first name" comment in the thread.

The flag is in the Organization field. It's written with a capital D there, but the flag format is lowercase, so we submitted it as `deadface{...}`.

## Flag

!!! flag "Flag"
    `deadface{c@tch_m3_if_y0u_c@n}`

## Takeaways

A self-signed cert says whatever its owner typed in when making it. Check the Subject and SAN on any small HTTPS target.
