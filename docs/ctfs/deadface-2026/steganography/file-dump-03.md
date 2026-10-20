---
title: File Dump 03
tags:
  - DEADFACE CTF 2026
  - Steganography
  - Forensics
---

# File Dump 03

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Steganography / Network |
| **Points** | 200 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Now that we know the who, what, when of it all, we want to embarrass the DEADFACE member responsible for this whole thing. Think you can do some digging and find their deepest darkest secret? I am pretty sure it is hidden on some external file location, but we don't know where!
>
> Submit the flag as `deadface{flag_text}`.

Files given: `File_Dump_Challenge.zip` (same archive as [File Dump 01](file-dump-01.md) and [File Dump 02](../osint/file-dump-02.md))

## Solution

### Steghide

Back in [File Dump 01](file-dump-01.md) we'd found this in the `dc:description` of `reminders.docx`:

```text
backup passphrase: n0rth_st4r
```

"Backup passphrase" on a dump full of JPEGs made us think steghide. We tried it on the images and `office_memory_meme.jpg` gave up a file:

```bash
steghide extract -sf "office_memory_meme.jpg" -p "n0rth_st4r"
```

That's `ftp_creds.txt`, with a login for `ghost404`. `technical_details.docx` in the dump mentions an FTP server at `45.79.200.170:21`.

### The FTP server

The creds work. The FTP root has `README.txt` and `ghost_repo.tar.gz`, and the tarball is a Git repo.

### Git history

The working tree only has boring notes. The log is more interesting, because one commit deletes `notes/personal.txt`:

```bash
cd ghost_repo
git log --stat --all
git log -p -n 2
```

The commit before that one added the file, and that's where the secret is:

```text
deadface{i_sleep_with_a_nightlight}
```

## Flag

!!! flag "Flag"
    `deadface{i_sleep_with_a_nightlight}`

## Takeaways

Passphrase in Office metadata, then steghide in a JPEG, then FTP creds, then a deleted file in Git history. When a file dump series has more parts, keep notes of every odd string you find, because the later parts will need them.
