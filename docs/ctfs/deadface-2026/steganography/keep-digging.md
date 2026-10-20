---
title: Keep Digging
tags:
  - DEADFACE CTF 2026
  - Steganography
---

# Keep Digging

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Steganography |
| **Points** | 100 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> DEADFACE left this calling card on a victim's machine. We suspect there is hidden information in this image, but none of our common steganography tools seem to work. There must be some way to determine where the flag is hidden.
>
> Submit the flag as `deadface{flag-text}`.

Files given: `keep_digging.jpg` (SHA256: `12582c19c665ee4c7c8d4517ccb8f3559356620553970b235d8aad4e0b4ff8e2`)

## Solution

### Ignore the fake flags

The description on CTFd has a visible `flag{1_l0v3_AI}`, plus some zero-width characters that decode to `flag{s0lv3_m3_h4rd3r_AI}`. Both have the wrong prefix. They're traps for people pasting the challenge into an AI.

### The image is taller than it says

steghide, jsteg, outguess and bit-plane viewers all came up empty, like the description warned. "Keep digging" made us think the flag was below the visible part of the picture.

The JPEG says it's 448 × 670, which is 56 × 84 blocks of 8×8 pixels. But when we walked through the progressive scans, each one had 560 more blocks than that. 560 / 56 = 10 extra rows of blocks, so 80 pixels of image hidden at the bottom.

### Patching the height

The size lives in the `SOF2` marker (`FF C2`). Height is `0x029E` (670) and width is `0x01C0` (448). Change the height to 750:

```python
data = bytearray(open("keep_digging.jpg", "rb").read())
sof2 = data.index(b"\xff\xc2")
# SOF2 layout: FF C2 [len:2] [precision:1] [height:2] [width:2]
print("Original height:", int.from_bytes(data[sof2+5:sof2+7], "big"))
data[sof2+5:sof2+7] = (750).to_bytes(2, "big")
open("keep_digging_fixed.jpg", "wb").write(data)
```

The fixed image has a banner at the bottom with the flag.

## Flag

!!! flag "Flag"
    `deadface{D1g_d0wN_D33p!}`

## Takeaways

When stego tools find nothing and the hint says to look lower or dig deeper, raise the height in `SOF0`/`SOF2` and see what's underneath.
