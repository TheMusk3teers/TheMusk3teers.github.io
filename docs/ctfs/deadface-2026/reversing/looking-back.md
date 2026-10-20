---
title: Looking Back
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Linux
---

# Looking Back

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics / Linux priv-esc |
| **Points** | 30 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Whichever member of DEADFACE who used the svc-deploy account last was sloppy. There are many DEADFACE members who use this machine. Access one of their accounts and read the flag in their home directory.
>
> NOTE: Heads up: charon is disposable. Every web-shell session is a fresh copy of the box, so grab each flag as you find it.

Access: the Hostbusters web shell (`hostbusters.dctf.io`), which drops you onto the `charon` box with a foothold as `svc-deploy`.

## Solution

The title and the "sloppy" hint both point at history. Start with the account we land on:

```bash
id
# uid=1001(svc-deploy) ...
ls -la ~
# .bash_history, flag.txt (owned by svc-deploy)
ls /home
# deephax  gh0st404  lilith  mort1cia  spookyboi  sunstalker  svc-deploy
```

`flag.txt` in our own home is not the target. The task says to get into *another member's* account. Read the shell history:

```bash
cat ~/.bash_history
```

```
cd /opt/deploy
cat README.md
systemctl status charon-collector.service || true
# ticket OPS-4471: rotate member pw after onboarding
echo "deephax:Sk4t3rB0y_S1nce_2011" | sudo chpasswd
history -c
```

The previous user set `deephax`'s password in plaintext with `chpasswd`, then ran `history -c` to clean up, but that only clears the *current* session's in-memory history. The line was already written to `.bash_history` on disk.

Switch to that account and read the flag:

```bash
su - deephax
# Password: Sk4t3rB0y_S1nce_2011
id
# uid=1002(deephax) ...
cat ~/flag.txt
```

```
deadface{h1st0ry_r3m3mb3rs_4ll}
```

`deephax`'s home also leaves a note that spells out the lesson:

> reminder: I committed the scraper service pw by accident and "removed" it in the next commit. must scrub git history properly at some point.

## Flag

!!! flag "Flag"
    `deadface{h1st0ry_r3m3mb3rs_4ll}` *(or `deadface{sh3ll_h1st0ry_t3lls_4ll}` on later container builds)*

## Takeaways

`history -c` doesn't scrub `~/.bash_history` on disk; it only clears the running shell's buffer. Always read `.bash_history` (and shell rc files, and `/opt` deploy dirs) first on a foothold box. Credentials "removed" in a later step are usually still recoverable—which leads directly to [Git Good](git-good.md).

