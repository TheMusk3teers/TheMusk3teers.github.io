---
title: "DECODE-DLE"
tags:
  - DEADFACE CTF 2026
  - Cryptography
---

# DECODE-DLE

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Logic puzzle |
| **Points** | 25 |
| **Author** | @G2Gh0st |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> One of the junior DEADFACE members left a number puzzle to decipher a 3-digit number combination. Do you have the skills to solve the puzzle and open the lock? Follow the clues to decipher the 3-digit code needed to "pick" the lock. Enter the code as 3 digits, no spaces or other separating characters.
>
> Submit the flag as `deadface{###}`

Files given: `Decoddle.png`, a classic "crack the lock" clue card.

## Solution

The five clues:

| Guess | Clue |
|---|---|
| 863 | One number correct, wrong place |
| 250 | Nothing correct |
| 168 | Two numbers correct, wrong place |
| 735 | One number correct, right place |
| 687 | Two numbers correct, wrong place |

Rather than reasoning by hand, check all 1,000 codes:

```python
clues = [("863", 1, 0), ("250", 0, 0), ("168", 2, 0), ("735", 1, 1), ("687", 2, 0)]
# (guess, digits in the code, of which in the right place)

def fits(code, guess, correct, placed):
    right = sum(a == b for a, b in zip(code, guess))
    common = sum(d in code for d in guess)
    return common == correct and right == placed

print([c for c in (f"{n:03}" for n in range(1000))
       if all(fits(c, *k) for k in clues)])
# ['716']
```

716 is the only code that fits: 6 and 1 are in the code but moved, 7 is in the right place, and none of 2, 5, 0 appear.

## Flag

!!! flag "Flag"
    `deadface{716}`

## Takeaways

With only 1,000 candidates, brute force against the clues is faster and safer than deduction.
