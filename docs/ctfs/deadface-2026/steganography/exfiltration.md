---
title: "Exfiltration"
tags:
  - DEADFACE CTF 2026
  - Steganography
---

# Exfiltration

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Steganography |
| **Points** | 75 |
| **Author** | @G2Gh0st |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> DEADFACE frequently uses unconventional methods to exchange information and avoid detection. Analysts recovered a set of images from a compromised communications channel that appear unrelated at first glance. We suspect the collection contains a hidden message intended for other members of the group. Recover the message and identify what DEADFACE is trying to say.
>
> Submit the answer as `deadface{flagtext}`.

Files given: `exfiltration.zip` with 20 JPEGs, `image_01.jpg` to `image_20.jpg`. (The hash labelled SHA1 is really a SHA-256.)

## Solution

Each JPEG has an EXIF image name like `IMG_6465` or `IMG_7b`. The part after `IMG_` is hex ASCII, one or two characters per image. The file numbers are shuffled; the real order is the EXIF timestamp, which is what the flag hints at.

```bash
exiftool -T -DateTimeOriginal -ImageDescription -FileName image_*.jpg | sort
```

| Time (2026-06-01) | File | Hex | Text |
|---|---|---|---|
| 14:02:17 | image_07 | 6465 | de |
| 14:04:52 | image_14 | 61 | a |
| 14:08:11 | image_03 | 6466 | df |
| 14:11:36 | image_19 | 61 | a |
| 14:13:09 | image_01 | 6365 | ce |
| 14:18:44 | image_12 | 7b | { |
| 14:21:03 | image_08 | 7469 | ti |
| 14:24:58 | image_20 | 6d | m |
| 14:29:12 | image_05 | 696e | in |
| 14:31:47 | image_16 | 67 | g |
| 14:35:20 | image_02 | 2d69 | -i |
| 14:39:06 | image_18 | 73 | s |
| 14:42:55 | image_09 | 2d65 | -e |
| 14:46:13 | image_13 | 76 | v |
| 14:51:38 | image_04 | 6572 | er |
| 14:55:04 | image_15 | 79 | y |
| 14:59:27 | image_10 | 7468 | th |
| 15:03:10 | image_06 | 69 | i |
| 15:07:42 | image_17 | 6e67 | ng |
| 15:12:29 | image_11 | 7d | } |

## Flag

!!! flag "Flag"
    `deadface{timing-is-everything}`

## Takeaways

For a set of "unrelated" files, dump every metadata field side by side. If file names are shuffled, sort by timestamps.
