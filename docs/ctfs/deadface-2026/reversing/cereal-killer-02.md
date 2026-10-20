---
title: "Cereal Killer 02"
tags:
  - DEADFACE CTF 2026
  - Reverse Engineering
---

# Cereal Killer 02 (RE03)

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Reverse engineering |
| **Points** | 150 |
| **Author** | @TheZeal0t |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

Find the Nordics' favourite (spooky) breakfast cereal; that answer leads to the flag.

Files given: `re03-ck02.7z` (password `KlippyAI`)
SHA256: `e4742efa2cb7c45353d7f87f4427f9abe3b450eea5469732e7b790647017c7ac`

| File | Type |
|---|---|
| `re03-lin-x86.bin` | ELF 32-bit i386 PIE, dynamically linked, **not stripped** |
| `re03-win-x86.exe` | PE32 console build of the same program |
| `KlippyAI-trained-on-CK02.exe` | Packed .NET "bonus" helper |

We only analysed the Linux build, statically. The Klippy "AI helper" is bait: the program tells you to run it, but it is a packed .NET binary from a CTF archive, so we left it alone.

## Solution

### 1. Triage

```bash
file re03-lin-x86.bin
strings -n 4 re03-lin-x86.bin
```

The imports are only `puts`, `putchar`, `memset`, `memcpy`, `memcmp`. There is no `scanf`, `fgets` or `read`, so **the binary never reads input**: it tries to decrypt the flag by itself. The strings give the theme:

```
[!] ERROR! The key has been tampered with.  It does not correctly decipher the flag!
[!] ERROR! TIME is REVERSIBLE, but the KEY is NOT... Go Back!  GO BACK!!!
...
grab the copy of Klippy AI that is tuned for this challenge, run it, and select "[A]" Analyze File
```

"Tamper", "reversible" and "Go back" are the hint.

### 2. Reading `main`

`nm` gives `main` at `0x133b` and a local (not libc) `memmem` at `0x11ed`. `objdump -d -M intel` on `main` shows:

1. `ebx` = GOT = `0x3fc4` (PIC thunk).
2. Copy `0x101` bytes from `ebx-0x15a4` = **`0x2a20`** (ciphertext) to the stack.
3. Copy `0x100` bytes from `ebx-0x14a0` = **`0x2b24`** (key) to the stack.
4. Print the banner and story.
5. XOR loop, 256 rounds: `out[i] = ct[i] ^ key[i]`.
6. Build the needle on the stack:

    ```
    mov DWORD PTR [ebp-0x325],0x66646162   ; "badf"
    mov WORD  PTR [ebp-0x321],0x3030       ; "00"
    mov BYTE  PTR [ebp-0x31f],0x64         ; "d"
    ```

7. `memmem(0x100, 7, out, "badf00d")`. If it's missing, print the "tampered" / "Go back" errors and the Klippy advert. If it's found, print the "Nordic fleet dispatched" warning and `puts(out)`.

The custom `memmem(hlen, nlen, hay, needle)` is a plain sliding `memcmp`, so the program only checks that the plaintext contains `badf00d`.

### 3. Fixing the tampered key

The plain XOR gives noise. With "Go back", the obvious fix is to **reverse the key**. We tried the key forwards, reversed and every rotation, against the ciphertext forwards and reversed, and kept the results that contain `badf00d`:

```python
import sys
d   = open(sys.argv[1], "rb").read()
ct  = d[0x2a20:0x2a20 + 0x100]
key = d[0x2b24:0x2b24 + 0x100]

pt = bytes(c ^ k for c, k in zip(ct, key[::-1]))
assert b"badf00d" in pt
print(pt.decode())
```

Only `key[::-1]` with the ciphertext as stored works:

```
****************
* f e e d . . .*
* f a c e . . .*
* d e a d b eef*
* d e a d . . .*
* f a c e { . .*
* N 0 r d i c s*
* -R3ally-like-*
* Frank3nb3rry-*
* cuz-bro-is- .*
* tall-like-. .*
* they-are!}. .*
* badf00d......*
* d a 7e4107 . * 
****************
```

### 4. Reading the flag out of the box

The rules say the flag has no spaces, periods or asterisks, so drop the frame and the padding. The `feed face deadbeef` lines are hex-speak filler, and the flag starts at `dead` + `face{`:

```
dead face{ N0rdics -R3ally-like- Frank3nb3rry- cuz-bro-is- tall-like- they-are!}
```

The Nordics' favourite cereal is **Franken Berry** (`Frank3nb3rry`). `badf00d` is the marker the program looks for, and `da7e4107` is trailing filler.

| Step | Technique |
|---|---|
| No input read | Import list from `strings` / `nm` |
| Find the buffers | PIC GOT maths in `main`: ciphertext `0x2a20`, key `0x2b24` |
| Success check | XOR, then search for `badf00d` |
| "Tampered" key | Reverse the key ("Go back") |
| Flag | Read the ASCII box without the frame and padding |

## Flag

!!! flag "Flag"
    `deadface{N0rdics-R3ally-like-Frank3nb3rry-cuz-bro-is-tall-like-they-are!}`

## Takeaways

Check the import table first: no input functions means the "answer" is computed inside the binary, and you only need to repair it. The error strings are the hint ("Go back" means reverse). Searching for a known marker (`badf00d`) over a few candidate keys solves it quickly. And you don't need the "AI helper" from the archive: never run unknown packed bonus binaries just because the challenge tells you to.
