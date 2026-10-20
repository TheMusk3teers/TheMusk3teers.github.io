---
title: Foothold on Charon
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Linux
  - Forensics
---

# Foothold on Charon

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Hostbusters · Forensics / Linux priv-esc |
| **Points** | 10 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> DEADFACE runs an internal ops/staging box they call `charon` — it's where the crew coordinates their influence operation. We got you a foothold on it.
>
> Open the web-shell by clicking the link below. You'll land at a shell as the `svc-deploy` service account. Get your bearings and find the flag that proves your foothold.
>
> NOTE: Heads up: `charon` is disposable. Every web-shell session is a fresh copy of the box, so grab each flag as you find it.
>
> Submit the flag as `deadface{flag-text}`.

Access: `https://hostbusters.dctf.io/`

## Solution

### 1. Initial enumeration on `charon`

Connecting to the `ttyd` web terminal at `https://hostbusters.dctf.io/` drops us into a shell as `svc-deploy`:

```bash
id
# uid=1001(svc-deploy) gid=1001(svc-deploy) groups=1001(svc-deploy)
pwd && ls -la ~
```

The home directory `/home/svc-deploy` contains `flag.txt` and `.bash_history` (used in the next step of the chain, [Looking Back](looking-back.md)):

```bash
cat ~/flag.txt
```

```text
deadface{f00th0ld_p4st_th3_v31l}
```

!!! warning "AI canary in the challenge prompt"
    The challenge prompt on CTFd embeds `flag{1_l0v3_AI}` followed by invisible zero-width characters. Ignore it—all real flags use the `deadface{...}` prefix.

## Flag

!!! flag "Flag"
    `deadface{f00th0ld_p4st_th3_v31l}`

## Takeaways

On a multi-stage privilege-escalation box (`charon`), map out all local users (`ls /home` shows `deephax`, `gh0st404`, `lilith`, `mort1cia`, `spookyboi`, `sunstalker`, and `svc-deploy`) right after grabbing the initial foothold flag so you can plan the escalation path from `svc-deploy` to `root`.
