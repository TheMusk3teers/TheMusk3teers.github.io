---
title: Git Good
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Linux
  - Forensics
---

# Git Good

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Hostbusters · Forensics / Linux priv-esc |
| **Points** | 100 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> As `deephax`, you can see this member keeps a little tooling project in his home directory. `deephax` swears he scrubbed the credentials out of it.
>
> He's wrong. Recover the password for the next member, become that user, and read their flag.
>
> Submit the flag as `deadface{flag-text}`.

Access: `https://hostbusters.dctf.io/` (continuing from [Looking Back](looking-back.md))

## Solution

### 1. Inspecting `deephax`'s Git repository

After switching to `deephax` using the password `Sk4t3rB0y_S1nce_2011` from [Looking Back](looking-back.md), listing `/home/deephax` reveals a reminder note and a Git repository at `/home/deephax/kingmaker-scraper`:

```bash
su - deephax
# Password: Sk4t3rB0y_S1nce_2011
cd ~/kingmaker-scraper
git log --oneline
git show HEAD~1
```

The repository has two commits. While the latest commit removes `SERVICE_PW` from `config.py`, inspecting the initial commit (`git log -p`) exposes the hardcoded password in plaintext:

```python
SERVICE_USER = "gh0st404"
SERVICE_PW   = "Gh0stInTh3Sh3ll_404"
```

### 2. Switching to `gh0st404`

Using `Gh0stInTh3Sh3ll_404` to authenticate as `gh0st404` and reading their home flag:

```bash
su - gh0st404
# Password: Gh0stInTh3Sh3ll_404
cat ~/flag.txt
```

```text
deadface{g1t_l0g_sp1lls_s3cr3ts}
```

## Flag

!!! flag "Flag"
    `deadface{g1t_l0g_sp1lls_s3cr3ts}`

## Takeaways

Deleting a secret in a new commit does not remove it from `.git/objects`. Unless the repository history is rewritten (and reflogs/unreachable blobs are pruned) and the leaked credential is rotated, `git log -p` will immediately recover it.
