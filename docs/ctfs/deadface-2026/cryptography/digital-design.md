---
title: Digital Design
tags:
  - DEADFACE CTF 2026
  - Cryptography
---

# Digital Design

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Cryptography |
| **Points** | 100 |
| **Author** | @G2Gh0st |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Investigators found a post by lilith on Ghosttown that looks like some new crazy encryption scheme but they can't make heads or tails of it. Find the post with a large binary string and an alphabetic string that contains the letters A, O, N, and X and see if you can break the code.
>
> Submit the answer as `deadface{FLAGTEXT}`.

## Solution

### The post

lilith's thread in Tech Talk is [Four letters. Still too many for you?](https://ghosttown.deadface.io/t/51/four-letters-still-too-many-for-you). It has two strings, each split into 11 groups:

```text
001100000 10110011 001110101 100111010 010101000 101111101 100101100 00001100 10001010 100000010 0100001110
```

```text
XXONO NXNOA OAOON NXXAO NAAXO XANXO ANOXA ANOXN NONOX ANXAX AOAAX
```

### Working out the rules

The replies basically give it away:

- "This has suspiciously little to do with language and a lot to do with circuits." So the letters are logic gates: `A` = AND, `O` = OR, `X` = XOR, `N` = NOT.
- "Some like company. One prefers to work alone." AND/OR/XOR take two bits, NOT takes one.
- "Start on the left. Take what you need, then move on." Go through each group's five gates left to right, eating bits from the matching binary group.
- "...even someone who can count through the alphabet should be able to read it." Each group gives a 5-bit number, and 1 = A, 2 = B and so on.

What convinced us: count the bits each gate group needs and it matches the length of its binary group every time (8, 9 or 10).

### Decoding

```python
bits = "001100000 10110011 001110101 100111010 010101000 101111101 100101100 00001100 10001010 100000010 0100001110".split()
gates = "XXONO NXNOA OAOON NXXAO NAAXO XANXO ANOXA ANOXN NONOX ANXAX AOAAX".split()

out = []
for b_chunk, g_chunk in zip(bits, gates):
    idx = 0
    val = ""
    for g in g_chunk:
        if g == "N":
            a = int(b_chunk[idx]); idx += 1
            val += str(1 - a)
        else:
            a, b = int(b_chunk[idx]), int(b_chunk[idx + 1]); idx += 2
            if g == "A":
                val += str(a & b)
            elif g == "O":
                val += str(a | b)
            elif g == "X":
                val += str(a ^ b)
    out.append(chr(ord("A") + int(val, 2) - 1))

print("deadface{" + "".join(out) + "}")
```

The first group worked by hand, to show the idea: `001100000` with `XXONO` is X(0,0)=0, X(1,1)=0, O(0,0)=0, N(0)=1, O(0,0)=0. That's `00010` = 2 = B.

| # | Bits | Gates | Output | Letter |
|---|---|---|---|---|
| 1 | `001100000` | `XXONO` | `00010` (2) | B |
| 2 | `10110011` | `NXNOA` | `01001` (9) | I |
| 3 | `001110101` | `OAOON` | `01110` (14) | N |
| 4 | `100111010` | `NXXAO` | `00001` (1) | A |
| 5 | `010101000` | `NAAXO` | `10010` (18) | R |
| 6 | `101111101` | `XANXO` | `11001` (25) | Y |
| 7 | `100101100` | `ANOXA` | `01100` (12) | L |
| 8 | `00001100` | `ANOXN` | `01111` (15) | O |
| 9 | `10001010` | `NONOX` | `00111` (7) | G |
| 10 | `100000010` | `ANXAX` | `01001` (9) | I |
| 11 | `0100001110` | `AOAAX` | `00011` (3) | C |

## Flag

!!! flag "Flag"
    `deadface{BINARYLOGIC}`

## Takeaways

Binary chunks of different lengths paired with fixed-length keys usually mean the key symbols eat different numbers of bits.
