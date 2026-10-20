---
title: Old Habits
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Linux
  - OSINT
---

# Old Habits

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Hostbusters · Linux / Password cracking / OSINT |
| **Points** | 350 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> `spookyboi` handled decryption for the crew and kept notes on the rest of the team. One of them: the DEADFACE second-in-command, `mort1cia` — who, apparently, *still* hasn't changed her password, and reuses the same personal one everywhere.
>
> `mort1cia` has a past, and it was exposed elsewhere. Track down the password she's been assigned, become her, and read the flag.
>
> Submit as `deadface{...}`.

Access: `https://hostbusters.dctf.io/` (continuing from [Dangerous Capability](dangerous-capability.md))

## Solution

### 1. OSINT clue on GhostTown

Checking `spookyboi`'s notes and the GhostTown thread [**yo mort1cia we need to talk about ur past**](https://ghosttown.deadface.io/t/50/yo-mort1cia-we-need-to-talk-about-ur-past) reveals two key pieces of information:

- `gh0st404` set up `mort1cia`'s account on `charon` and teases her by calling her **"princess"**.
- The thread attaches an old Halloween throwback photo of her dated **`2015-10-31`** (`2015`).

Combining the nickname and year matches the `<word><year>` password pattern noted in `spookyboi`'s cache: **`princess2015`**.

### 2. Switching to `mort1cia` on `charon`

Authenticating as `mort1cia` with `princess2015` on `charon` and reading `/home/mort1cia/flag.txt`:

```bash
su - mort1cia
# Password: princess2015
id
# uid=1007(mort1cia) gid=1007(mort1cia) groups=1007(mort1cia),1008(ops)
cat ~/flag.txt
```

```text
deadface{0s1nt_n3v3r_f0rg3ts}
```

## Flag

!!! flag "Flag"
    `deadface{0s1nt_n3v3r_f0rg3ts}`

## Takeaways

When on-box enumeration hints at password reuse and personal history, pivot to the target's forum posts and image timestamps (`princess` + `2015` → `princess2015`) rather than brute-forcing blind wordlists over a slow web shell.
