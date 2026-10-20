---
title: Cereal Killer 03
tags:
  - DEADFACE CTF 2026
  - Reverse Engineering
---

# Cereal Killer 03

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Reverse engineering |
| **Points** | 500 |
| **Author** | @TheZeal0t |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Next up, we have the Reptilians (a.k.a. "Lizard People")... Our Lizard Over-Friends have provided us with a binary for us to analyze. Crack the password, enter it, and look for any other little "gotchas" that our Lizard Over-Pals may have placed as a test of our worthiness.
>
> Submit the answer as `deadface{I-For-One-Welcome-Our-New-Buggy-Overlords}` ... which is not the REAL FLAG, by the way.

Files given: `re04-win-x64.exe` (145 KB 64-bit Windows PE, SHA256: `e92ec049af8451b6e293c388c8ef504b3a8fa0598bb078f1a678904332ae6a7b`)

## Solution

### The password

The binary MD5s your input and compares it to `0d107d09f5bbe40cade3de5c71e9e9b7`. That one is in every hash lookup site: it's `letmein`.

### The gotcha

Typing `letmein` still gets you "Access denied". So we went looking at what runs at startup:

1. A helper thread registers a vectored exception handler and puts a hardware breakpoint (`Dr0`) on the success routine.
2. When the password check passes and execution reaches the success routine, the breakpoint fires `EXCEPTION_SINGLE_STEP` (`0x80000004`).
3. The handler writes `0xD1CEFA11BADC0DE5` into a global, then moves `RIP` to the "Access denied" branch.

So the right password always fails, but the handler still sets the value the decryption needs.

### Decrypting it ourselves

The success routine takes the first 8 bytes of `MD5("letmein")` (`0d 10 7d 09 f5 bb e4 0c` as a 64-bit int), XORs them with `0xD1CEFA11BADC0DE5`, and uses that to seed a `xorshift64` PRNG. The PRNG decrypts 55 bytes from `.rdata`, and the result is checked against another MD5 in the binary.

We redid that in Python, and the output matched the embedded hash:

```text
deadface{Liz4rds_L0ve_Krampuffs_for_some_odd_reason...}
```

## Flag

!!! flag "Flag"
    `deadface{Liz4rds_L0ve_Krampuffs_for_some_odd_reason...}`

## Takeaways

If a binary rejects a password you know is right, look for TLS callbacks, extra threads and vectored exception handlers that touch `Dr0`–`Dr7`. Here the handler both hid the success path and set half the key.
