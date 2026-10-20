---
title: Speak Friend and Enter
tags:
  - DEADFACE CTF 2026
  - Web
---

# Speak Friend and Enter

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Web |
| **Points** | 200 |
| **Author** | @Cyb0rgSw0rd |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> EvErYbOdY's So CrEaTiVe... That last PCAP (Pirates Life) had some weird fingerprints in it. Dig into it and see if you can find out what DEADFACE is up to. With MaudeAI rotting some brains, we think they may have tried to find a way to get even sneakier online.
>
> Submit the flag as: `deadface{flag-text}`

This picks up right where [pirateslife](../forensics/pirateslife.md) left off.

## Solution

### The fingerprint in the PCAP

In `pirateslife.pcap`, host `172.16.14.43` makes HTTP requests with an odd User-Agent (typo included):

```text
SunstalkerDFBrower/2.0 (Linux; Android 14)
```

The same host also sends a plain `curl/8.18.0` request to `http://speakfriend.dctf.io/`, which redirects to HTTPS, and then there's TLS 1.3 traffic to that host.

### Same URL, different site

With a normal browser, `https://speakfriend.dctf.io/` is `gh0st404`'s cooking and tech blog, "bytes & bakes". That fits the "EvErYbOdY's So CrEaTiVe" cooking-video joke in the prompt. The `/about` page has a shoutout to "my frend S who wrote her own web brower from scratch".

Send the Sunstalker User-Agent and the site turns into a DEADFACE channel called `deaddrop`:

```bash
curl -sS https://speakfriend.dctf.io/ \
  -A 'SunstalkerDFBrower/2.0 (Linux; Android 14)'
```

There's a banner, `SECURE CHANNEL // sunstalker authenticated // DEADFACE eyes only`, and the recipes are replaced by op plans. A roll-call post at `/post/read-this-sunstalker` (404 for normal browsers) says:

> **the TGRI one is the important one**
>
> d34th needs ur confirmation code from the TGRI keynote post. go read it. the real version.

### The TGRI token

`/post/tgri-innovation-summit-recap` with the same User-Agent shows `tgri op: sunstalker this ones yours`. At the bottom, `gh0st404` writes:

> d34th wants confirmation from everyone on this op. give him this... but ull have to finally hang out with me to get the real flag i guess:
>
> **`pilotnoi{7p-ied-lf6h-6ti-3f0r-dr4e50qx-ob-ag-z44}`**
>
> dont screenshot. dont copy it anywhere else. we all have the same internet. we just dont all see the same thing.

The title is the door riddle from Moria in *The Lord of the Rings*. The answer is `mellon`, Elvish for "friend", and gh0st404 joking that Sunstalker has to hang out with him points the same way. So `mellon` is the key.

Plain a–z Vigenère didn't work because of the digits. Using a 36-character alphabet (`a-z0-9`), only advancing the key on letters and digits, and leaving the hyphens alone:

```python
cipher = "pilotnoi{7p-ied-lf6h-6ti-3f0r-dr4e50qx-ob-ag-z44}"
alpha = "abcdefghijklmnopqrstuvwxyz0123456789"
key = "mellon"

out = []
ki = 0
for ch in cipher:
    if ch in alpha:
        c = alpha.index(ch)
        k = alpha.index(key[ki % len(key)])
        out.append(alpha[(c - k) % len(alpha)])
        ki += 1
    else:
        out.append(ch)

print("".join(out))
# deadface{we-411-h4v3-the-s4me-1nt3rnet-d0-w3-n0t}
```

`pilotnoi` turns into `deadface`, and the rest reads "we all have the same internet, do we not", which matches the last line of the post.

!!! warning "Keep the hyphens"
    Don't swap the hyphens for underscores. CTFd rejected `deadface{we_411_...}`.

## Flag

!!! flag "Flag"
    `deadface{we-411-h4v3-the-s4me-1nt3rnet-d0-w3-n0t}`

## Takeaways

- A server can serve a whole different site based on `User-Agent`. If a PCAP shows a weird one, replay it.
- If the known prefix (`deadface` → `pilotnoi`) only lines up when digits count as letters, the alphabet is `a-z0-9`.
- Copy punctuation from the ciphertext exactly.
