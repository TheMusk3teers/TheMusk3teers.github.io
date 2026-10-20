---
title: Chatty Cathy
tags:
  - DEADFACE CTF 2026
  - OSINT
---

# Chatty Cathy

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT |
| **Points** | 15 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

Follow a chatty DEADFACE associate from the Ghost Town forum to whatever they overshared online, and find the flag.

## Solution

### Ghost Town

In `/t/20/travel-destinations` on the Off-Grid board, the poster drops a link to their own site:

> i put them all up on my little travel blog `https://170.187.156.131:8443/`

Someone replies "It has your first name on it." So: a self-hosted blog that leaks personal info.

### The blog

It's called "Wander with Alice". On a small self-hosted site, the first thing we check is `robots.txt`:

```
https://170.187.156.131:8443/robots.txt
```

```
Disallow: /secret_members
# deadface{y0u_m3ddling_k!ds}
```

The flag is a comment right there, under a hidden path.

## Flag

!!! flag "Flag"
    `deadface{y0u_m3ddling_k!ds}`

## Takeaways

Read `robots.txt` on small targets. The blog also says "I will keep posting these on both profiles", so there's another profile out there, which matters later in the series.
