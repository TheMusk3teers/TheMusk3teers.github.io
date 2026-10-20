---
title: Necromancy
tags:
  - DEADFACE CTF 2026
  - Exploitation
  - Reverse Engineering
---

# Necromancy

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | PWN / Binary Exploitation |
| **Points** | 300 |
| **Author** | @SpiffyLich |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Yo, Charles here. We finally got it! This is the last one. After digging through, it seems like solving this final calling card program will net you a chance to talk with DEADFACE members, via some encrypted message channel. If you can crack this program, we can help edit it and hopefully track them down!
>
> Submit the flag as `deadface{flag-text}`.

Files given: `Necromancy.zip` (513 MB padded ELF, SHA256: `78a39a5df5a1d4672d10a268e71f407e87b7fd4b828b1eb5baf9ebf590967df4`)
Target: `159.65.228.107:5005`

## Solution

### 1. Binary triage

`Necromancy` is the final binary in @SpiffyLich's 5-part pwn series (following [The Crypt](the-crypt.md), [Skeleton Key](skeleton-key.md), [Cursed Mirror](cursed-mirror.md), and [Cursed Library](cursed-library.md)). It is an x86-64 non-PIE ELF with NX enabled and no stack canary:

- The vulnerable routine reads up to `1024` bytes into a `256`-byte stack buffer (`0x100` bytes to the saved return address).
- The binary provides a minimal set of gadgets including `syscall` and a helper that performs a `read` into `.bss`, making it ideal for **Sigreturn-Oriented Programming (SROP)**.

### 2. Two-stage SROP chain to `execve("/bin/sh", 0, 0)`

1. **Stage 1 (Stack Pivot / `.bss` staging):** Overflow the `256`-byte buffer to trigger a `read` into a known writable `.bss` address where we place `/bin/sh\x00` and a second `SigreturnFrame`.
2. **Stage 2 (`rt_sigreturn` → `execve`):** Trigger a `15`-byte `read` (`SYS_rt_sigreturn = 15` in `rax`) followed by `syscall` to load the `SigreturnFrame` registers (`rax = 59`, `rdi = bss_binsh_addr`, `rsi = 0`, `rdx = 0`, `rip = syscall_addr`), spawning `/bin/sh` on the remote container.

Once the shell opens in `/app`, reading `/app/flag.txt` returns the flag:

```bash
cat /app/flag.txt
```

```text
deadface{P13c3_By_P13c3_T#3_D34D_SH477_R153}
```

## Flag

!!! flag "Flag"
    `deadface{P13c3_By_P13c3_T#3_D34D_SH477_R153}`

## Takeaways

When a stripped or minimal non-PIE binary lacks `pop rsi` / `pop rdx` gadgets but includes `syscall` and a way to set `rax = 15` (such as a `read(0, buf, 15)` return value), `pwntools`'s `SigreturnFrame()` turns a single `syscall` gadget into full control over every general-purpose register.
