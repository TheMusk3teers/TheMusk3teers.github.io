---
title: Headshot
tags:
  - DEADFACE CTF 2026
  - Steganography
---

# Headshot

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Steganography |
| **Points** | 10 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> DEADFACE compromised a fileshare of corporate headshots for Aurora Health employees. They are tagging the photos with information that tells their group which employees to target. Figure out which image DEADFACE tampered with. Submit the answer as `deadface{flag_text}`.

Files given: `headshot.zip` (SHA256 `5c6572a3ab5a10e682ba479009ee57418d455d7ac678cd6041c54f7a50c40bc9`; the page labels it SHA1, but it is a 64-character SHA256 value)

## Solution

The ZIP holds three 512x512 palette PNGs plus one extra file:

| File | Size (bytes) | Modified |
|---|---|---|
| `headshot001.png` | 169,920 | 02:40:43 |
| `headshot002.png` | 174,949 | 02:41:33 |
| `headshot003.png` | 182,911 | 02:45:53 |
| `headshot003.png_original` | 182,866 | 02:41:53 |

The `_original` suffix is the backup exiftool writes whenever it edits a file in place. So someone ran exiftool on `headshot003.png`, which also explains why that file was modified four minutes after the others.

Dumping all metadata shows the tag immediately:

```bash
exiftool -a -G1 -s -u headshot*
# headshot003.png
# [PNG]  Comment : deadface{b00m_h34Dsh0TTT}
```

None of the other files, including the backup, has a `Comment`. Walking the PNG chunks confirms that the only change is one new `tEXt` chunk (`Comment\0deadface{...}`, 33 bytes) inserted before `IEND`. That accounts for the 45-byte size difference (33 bytes of data plus 12 bytes of chunk header and CRC). The `IDAT` pixel data is unchanged, and no file has data after `IEND`.

## Flag

!!! flag "Flag"
    `deadface{b00m_h34Dsh0TTT}`

## Takeaways

Check the file listing for an exiftool `_original` backup: it tells you which file was edited and gives you a clean copy to diff against. For PNGs, `tEXt`/`zTXt`/`iTXt` chunks are the first place to look.
