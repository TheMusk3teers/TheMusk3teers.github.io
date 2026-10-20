---
title: It But A Wee Bit Reverse
tags:
  - DEADFACE CTF 2026
  - Reverse Engineering
---

# It But A Wee Bit Reverse

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Reverse engineering |
| **Points** | 15 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> We pulled this binary off a DEADFACE member's machine. It's guarding something, and it wants a password. Strings won't save you this time, they were just clever enough to not leave it lying in plaintext. Figure out what it wants, and it'll give up its secret.
>
> Submit the flag as `deadface{flag-text}`.

Files given: `re02.bin` (16 KB x86-64 ELF, SHA256: `fa76583b6f4c9f6414fb79c3ffe717709ba3d74ce62ce7043d59807e8ca538f8`)

## Solution

### 1. Password check (`XOR 0x42`)

Disassembling `main` in `re02.bin` shows a three-step verification:

1. **Length check:** Rejects any input whose length is not exactly `10` characters.
2. **Single-byte XOR check:** XORs each input byte with `0x42` and compares the result against the 10-byte array `pw_enc`:
   ```text
   31 37 2c 31 2a 2b 2c 27 76 70
   ```
   XORing `pw_enc` with `0x42` recovers the password **`sunshine42`**.
3. **Flag decryption:** When the password matches, the binary XORs the 31-byte `flag_enc` buffer with the repeating 10-byte password `sunshine42` and prints the result:

```python
pw_enc = bytes.fromhex("31372c312a2b2c277670")
pw = bytes(b ^ 0x42 for b in pw_enc)
print("Password:", pw.decode())
# Password: sunshine42
# Flag: deadface{x0r_1s_not_encryption}
```

## Flag

!!! flag "Flag"
    `deadface{x0r_1s_not_encryption}`

## Takeaways

When `strings` doesn't show the plaintext password or flag, check `.rodata`/`.data` for short byte arrays referenced right before a comparison loop—a single-byte XOR (`0x42`) hides strings from `strings` while remaining trivial to invert statically.
