---
title: The Mark
tags:
  - DEADFACE CTF 2026
  - OSINT
  - Steganography
---

# The Mark

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT / Crypto / Stego |
| **Points** | 350 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> This is a hybrid challenge that spans multiple categories. Think logically about this challenge and treat it like an investigation.
>
> DEADFACE bragged about gaining access to sensitive information belonging to TGRI. We need to find out what DEADFACE uncovered. We think they're targeting a person - but who? Search for intel where DEADFACE posts and follow the trail.
>
> Submit the flag as `deadface{first_last}`. Example: `deadface{John_Smith}`.

## Solution

### 1. GhostTown thread 59

In "Who knew doomscrolling could be so useful?", daem0n asks for "our mark's sensitive file" and d34th posts `bigdawg-1.jpg`. The next page only says "You need to look deeper."

### 2. Hidden rows in the JPEG

The JPEG's SOF0 header says the image is 516 px tall, but the file holds scan data for more rows. Patching the height field to 1024 reveals a QR code under the dog:

```python
d = bytearray(open('bigdawg-1.jpg', 'rb').read())
i = d.find(b'\xff\xc0')
d[i + 5:i + 7] = (1024).to_bytes(2, 'big')   # height
open('bigdawg-tall.jpg', 'wb').write(d)
```

The QR points to `https://tinyurl.com/yru2dna5`, which redirects to `tgri_records.csv.gpg`.

### 3. DoomScroll's hidden metadata

In the same thread, mirveal lists DoomScroll API paths. `GET /api/posts/<id>/meta` returns entries the site never shows. Each value is written in three kinds of Unicode space, U+2002, U+2003 and U+2009, read as base-3 digits, 6 digits per character.

- Posts 223, 224, 226, 227 and 228 hold a d34th / lilith conversation: *"TGRI's CFO, Santos Staudenmaier, has my attention… Staudenmaier is our priority. He may be the key to bringing TGRI down."*
- Post 235 has a "passcode" entry linking to `pastebin.com/GBcnwhvZ`, which holds the GPG passphrase.

### 4. Confirming

The passphrase decrypts `tgri_records.csv.gpg`: TGRI finance records for Santos Staudenmaier, CFO (bonus holdback, a "Glass Harbor" research budget, ledger discrepancies).

!!! danger "Don't submit"
    - Post 274's meta holds `flag{sUbm1T_m3_1f_Im_Ch34TiNG}`, labelled "AI Check". It's a canary for AI-assisted solving.
    - The `flag{08a7cdddc7dcbba8}` in the challenge text doesn't match the required format.

## Flag

!!! flag "Flag"
    `deadface{Santos_Staudenmaier}`

## Takeaways

Check JPEG dimensions against the data size; a shrunk height hides content. Call API routes the UI never uses. Unused leads that may matter later: DoomScroll post 252's emoji cipher ("The lookup table is live…") and GhostTown thread 51's logic-gate puzzle (`BINARYLOGIC`). Continued in [Get That Bag](get-that-bag.md).
