---
title: Death
tags:
  - DEADFACE CTF 2026
  - OSINT
---

# Death

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT |
| **Points** | 100 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> What is `d34th`'s first name?
>
> Submit the flag as `deadface{John}`

## Solution

### GhostTown

In the thread [d34th pls explain the glasses 😭💛](https://ghosttown.deadface.io/t/48/d34th-pls-explain-the-glasses), `luciafer` makes fun of d34th's yellow blue-light glasses, and then adds at the bottom:

> *"UPDATE: IT'S POSTEDDD 💛📸 no tag, as promised!!"*

So there's a photo somewhere, posted without tagging him.

### DoomScroll

On luciafer's DoomScroll profile (`https://doomscroll.dctf.io`), post #200 is a photo of d34th in the yellow glasses: "when the cybersecurity grind is so intense you gotta see life in 4K nacho cheese".

The site also has a metadata endpoint for each post:

```bash
curl -s https://doomscroll.dctf.io/api/posts/200/meta
```

```json
[
  {
    "id": 1,
    "key": "filename",
    "value": "travis_001.png",
    "is_encrypted": 0,
    "created_at": "2026-10-14 20:11:43",
    "decrypted_value": null
  }
]
```

luciafer didn't tag him, but she didn't rename the file either. His name is Travis.

## Flag

!!! flag "Flag"
    `deadface{Travis}`

## Takeaways

Even if the site strips EXIF, look for an API that keeps the original upload filename.
