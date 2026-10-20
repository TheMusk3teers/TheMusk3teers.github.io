---
title: Cereal Killer 04
tags:
  - DEADFACE CTF 2026
  - Reverse Engineering
---

# Cereal Killer 04

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Reverse engineering |
| **Points** | 250 |
| **Author** | @TheZeal0t |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Finally, we have the Greys... Unlike the rest of our Far-Flung Friends, the Greys communicate with each other using "Assembly Codes"... We have recovered a binary from one of their crashed ships, and we'd like you to analyze it. Once we figure out their favorite spooky cereal, we feel like we can better bond with our new friends!
>
> Submit the answer as `deadface{I-For-One-Welcome-Our-New-Buggy-Overlords}` ... which is not the REAL FLAG, by the way.

Files given: `re05.exe` (14 KB .NET executable, SHA256: `045f72d2027c0c9397d136375341b4c5d6b9bb6765a73aa75fd3955bbef2f96f`)

## Solution

### The broken loader

`re05.exe` is a 32-bit .NET program that XOR-decrypts an embedded byte array and loads it with `Assembly.Load`. Running it just crashes, because the hardcoded 2-byte key is `[0x01, 0x01]`, which is wrong.

A PE file is full of zero bytes (after the DOS stub, in section padding). Anything XORed with zero is the key itself, so those areas of the blob should just show the key repeating. They do: `0x41 0x33`, or `"A3"`. XOR with that and the blob starts with a proper `MZ` header (`4d 5a 90 00...`):

```python
# Recover the inner .NET assembly by XORing the embedded blob with b"A3"
inner_pe = bytes(b ^ (0x41 if i % 2 == 0 else 0x33) for i, b in enumerate(embedded_blob))
open("inner_stage.dll", "wb").write(inner_pe)
```

### The inner assembly

Decompiling `inner_stage.dll` and looking at `Run`:

1. It MD5s your input and compares it to the hash of `booberry`, the cereal.
2. `booberry` is then used to decrypt the inner ciphertext, which gives the flag:

```text
deadface{Before_Consenting_to_Med_Exam_Ascertain_Destination_of_Finger!}
```

## Flag

!!! flag "Flag"
    `deadface{Before_Consenting_to_Med_Exam_Ascertain_Destination_of_Finger!}`

## Takeaways

If someone XORs a PE with a short key, look at where the zero bytes should be. The key shows up there in plain sight.
