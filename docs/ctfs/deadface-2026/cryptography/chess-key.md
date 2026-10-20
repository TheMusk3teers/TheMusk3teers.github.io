---
title: Chess Key
tags:
  - DEADFACE CTF 2026
  - Cryptography
  - OSINT
---

# Chess Key

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Cryptography |
| **Points** | 200 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Looks like DEADFACE compromised a machine and left this cryptic calling card:
>
> `checkmate. nZ3ipU8_yy3p5_hvyKi0`
>
> One of our other analysts also found this image that we believe is tied to this calling card. Use this image and any information you can find in GhostTown to reveal the flag.
>
> Submit the flag as `deadface{flag-text}`.

Files given: `chesskey.png` (SHA256: `583450106b366b832250ee0dc30d57a055b27772387801d6dc77db9515cd354b`)

## Solution

### The forum post

On GhostTown, in Tech Talk, `b1tfr34k` explains the cipher in [Carlsen gave me an idea](https://ghosttown.deadface.io/t/38/carlsen-gave-me-an-idea):

- He put a letter on each of the 64 squares (that's `chesskey.png`).
- He picked a 2026 Magnus Carlsen game where Magnus opened with the English (`1. c4`).
- "Only Magnus's moves matter though. idk about the other player."
- The square each of Magnus's moves lands on gives one letter of a Vigenère key.

### The game and the board

We went through Magnus's (`MagnusCarlsen`) 2026 games looking for `1. c4` and found his Titled Tuesday win as White against GM Aleksandar Indjić (`Beca95`) on January 13, 2026, [game 162638222425](https://www.chess.com/game/live/162638222425):

```text
1. c4 e5 2. g3 Nc6 3. Bg2 h5 4. Nf3 e4 5. Nh4 g5 6. Nf5 h4 7. Nc3 Nf6
8. d3 d6 9. g4 Nxg4 10. Bxe4 Nh6 11. Nxh6 Bxh6 12. Rg1 Ne5 13. d4 ...
```

`chesskey.png` read the normal way (rank 8 at the top, file a on the left):

```text
8 | A  I  W  Y  A  E  I  O
7 | J  B  K  B  F  J  P  U
6 | X  L  C  M  K  Q  V  A
5 | Z  C  N  D  O  W  B  E
4 | D  G  L  P  E  Q  F  I
3 | H  M  R  X  R  F  S  K
2 | N  S  Y  C  G  T  G  U
1 | T  Z  D  H  J  L  V  H
   ------------------------
    a  b  c  d  e  f  g  h
```

### Decrypting

The calling card has 13 letters, so we need White's first 13 moves. Their squares give the key `LSGFIWRXFEAVP`. Digits and underscores pass through and don't use up a key letter:

| Move | Square | Key | Cipher | Plain |
|---|---|---|---|---|
| `1. c4` | c4 | L | n | c |
| `2. g3` | g3 | S | Z | H |
| | | | 3 | 3 |
| `3. Bg2` | g2 | G | i | c |
| `4. Nf3` | f3 | F | p | k |
| `5. Nh4` | h4 | I | U | M |
| | | | 8_ | 8_ |
| `6. Nf5` | f5 | W | y | c |
| `7. Nc3` | c3 | R | y | h |
| | | | 3 | 3 |
| `8. d3` | d3 | X | p | s |
| | | | 5_ | 5_ |
| `9. g4` | g4 | F | h | c |
| `10. Bxe4` | e4 | E | v | r |
| `11. Nxh6` | h6 | A | y | y |
| `12. Rg1` | g1 | V | K | P |
| `13. d4` | d4 | P | i | t |
| | | | 0 | 0 |

`cH3ckM8_ch3s5_cryPt0`, which fits the "checkmate." on the card.

## Flag

!!! flag "Flag"
    `deadface{cH3ckM8_ch3s5_cryPt0}`

## Takeaways

When you get a mystery grid image, look for the forum post that explains it. The "checkmate." prefix also let us check the board orientation fast: if the first few letters didn't come out as `cH3ck`, something was flipped.
