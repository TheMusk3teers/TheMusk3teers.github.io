---
title: "Art Heist: The Forgery"
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# Art Heist: The Forgery

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics |
| **Points** | 25 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

A ZIP of five paintings. One is a forgery planted by a DEADFACE member; find it and the flag.

Files given: a ZIP (SHA256 `f4d4d118…cb12`) holding five 1024×1024 RGB PNGs, plus macOS junk (`.DS_Store`, `__MACOSX`).

## Solution

Five images that look the same, one of them fake. We figured the metadata would give it away, so we dumped all of it:

```bash
exiftool -a -G1 *.png
```

`Painting_002.png` ("Mona Lisa Loading") doesn't match the others anywhere:

- Software is `LAMIA Restoration Utility 0.0.13`. The other four say `Atelier Archive`.
- It has an extra PNG `tEXt` comment: "checksum mismatch; palette recovered from cache".
- XMP Creator is "Leonardo da Vinci / LAMIA", Rights is "LAMIA temporary acquisition", Description is "Authenticity status: pending…".
- XMP ModifyDate is `2026:09:01`. On the other four it's the same as their period CreateDate. Its ZIP mtime is Oct 15, the others Sep 1.

So it's `Painting_002.png`, and the DEADFACE member is LAMIA. The flag is just sitting in the XMP `dc:identifier`:

```bash
exiftool -a -G1 Painting_002.png | grep -i identifier
# XMP-dc  Identifier  deadface{L4MIA_SIGNS_HER_WORK}
```

!!! note "Decoys checked"
    Nothing after `IEND` in any file, no readable LSB text (all of RGB or per channel), and the AppleDouble files only have `com.apple.macl`.

## Flag

!!! flag "Flag"
    `deadface{L4MIA_SIGNS_HER_WORK}`

## Takeaways

When files should match, diff their metadata and look for the odd one out. Use `exiftool -a -G1`, since normal image viewers don't show XMP fields.
