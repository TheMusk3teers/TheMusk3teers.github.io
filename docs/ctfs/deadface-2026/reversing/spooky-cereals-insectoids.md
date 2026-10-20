---
title: "Cereal Killer 01"
tags:
  - DEADFACE CTF 2026
  - Reverse Engineering
---

# Cereal Killer 01

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Reverse engineering |
| **Points** | 50 |
| **Author** | @TheZeal0t |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[chaimae098](https://github.com/chaimae098) · Lyna · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) |

## Challenge

Files given: `re01-win-x86.exe` (PE32, Intel 80386, console app)
SHA256: `5d5e9dccf7179e1d7cac1dbe2376b8001137d04516ce885a8dbec4e27231458d`

## Solution

### 1. Triage

```bash
file re01-win-x86.exe
# PE32 executable (console) Intel 80386, for MS Windows, 17 sections
```

`strings` shows an ASCII-art banner, flavor text and the key UI strings:

```
Please enter the password:
Access granted
Flag decode error
Good job! You are ALMOST there... :
ACCESS DENIED... HACKER TRACKING ENABLED
```

It's built with MinGW-w64 GCC (`GNU C99 9.3-win32 ... -g -O2 -m32`) and keeps its DWARF debug sections. `objdump --dwarf=decodedline` even gives the source filename, `re01.c`.

### 2. Reading `main`

The symbol table (`objdump -t`) gives the addresses:

```
0x4095b0 <_main>
0x401630 <_base64_decode>
```

Disassembling `main` (`objdump -d -M intel --start-address=0x4095b0`) shows the flow:

1. Copy two embedded strings onto the stack.
2. Print the banner.
3. `getline()` the password.
4. `SHA1(input)`, turned into a 40-character lowercase hex string with `sprintf("%02x")`.
5. `strcmp()` it against the first embedded blob, decoded by a custom `_base64_decode`.
6. On a match, print "Access granted", base64-decode the second blob, and **RC4-decrypt it using the password text you typed as the key** (not the hash).
7. Otherwise, print the "ACCESS DENIED" troll message.

The RC4 call:

```
lea eax,[esp+0x380]       ; RC4 output buffer
mov [esp+0xc], eax        ; arg: dst
lea eax,[esp+0x280]
mov [esp+0x4], eax        ; arg: src (ciphertext)
mov eax,[esp+0x24]        ; pointer to the ORIGINAL input buffer
mov [esp], eax            ; arg: key = password text
call RC4
```

### 3. The custom base64 alphabet

`_base64_decode` uses its own 64-character table in `.rdata` at `0x40b860`:

```
ZKj9n+yf0wDVX1s/5YbdxSo=ILaUpPBCHg8uvNO4klm6iJGhQ7eFrWczAMEq3RTt
```

Here `=` is a normal alphabet character, not padding. The two embedded strings are UTF-16LE (`X\x00` pairs in a hex dump), so decode them from UTF-16 first, then with this alphabet:

```python
alpha = "ZKj9n+yf0wDVX1s/5YbdxSo=ILaUpPBCHg8uvNO4klm6iJGhQ7eFrWczAMEq3RTt"

def dec(s):
    out = bytearray()
    for i in range(0, len(s), 4):
        a, b, c, d = [alpha.find(ch) for ch in s[i:i+4]]
        out.append(((a << 2) | (b >> 4)) & 255)
        if c < 0: break
        out.append(((b << 4) | (c >> 2)) & 255)
        if d < 0: continue
        out.append(((c << 6) | d) & 255)
    return bytes(out)
```

| Blob | Decoded |
|---|---|
| `0x40a020` (56 chars) | `68158678a3f9d76a3d1ca12d20b2d6af9bd3f290`, the SHA1 of the correct password |
| `0x40a0a0` (80 chars) | 59 bytes of RC4 ciphertext |

### 4. Cracking the SHA1

The RC4 key is the password itself, so matching the hash isn't enough: we need the plaintext. A small themed guess list and `wamerican` failed, so we ran `rockyou.txt`:

```python
import hashlib
target = "68158678a3f9d76a3d1ca12d20b2d6af9bd3f290"
for line in open("rockyou.txt", encoding="latin1", errors="ignore"):
    w = line.rstrip("\r\n")
    if hashlib.sha1(w.encode()).hexdigest() == target:
        print(w)
        break
```

Result: **`cookiecrisp`**, which fits the breakfast-cereal theme.

### 5. Decrypting the payload

```python
def rc4(key, data):
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    out, i, j = bytearray(), 0, 0
    for c in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        out.append(c ^ S[(S[i] + S[j]) % 256])
    return bytes(out)

ct = bytes.fromhex("00bf0865fe0878b54b446aab09aa727e7a25296efdc4e8d"
                   "00712254a8e05916b47e04a9b1cb67902f8350b91d7eef4"
                   "3e5a84c00137c050579395ce")
print(rc4(b"cookiecrisp", ct))
# https://cerealkiller.lyttonlabs.org/unwanted-guests/ick.png
```

The payload is a URL, not the flag ("You are ALMOST there").

### 6. The image

Running the binary from a Windows console (double-clicking closes the window instantly) and entering `cookiecrisp` gives "Access granted" and the same URL. The image is a mock Lytton Labs personnel photo of "Dr. Charles K. Gesch-ICK-ter, Director", with the flag printed above it.

| Step | Technique |
|---|---|
| Find the secrets | Disassemble `main`, symbol table, DWARF info |
| Decode them | Custom base64 alphabet + UTF-16 |
| Beat the hash check | SHA1 wordlist search with rockyou → `cookiecrisp` |
| Decrypt | RC4 with the password as key → image URL |
| Flag | Printed on the image |

## Flag

!!! flag "Flag"
    `deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}`

## Takeaways

Check what the decryption key really is: here it was the typed password, so cracking the hash was required. Try rockyou early on unsalted SHA1. Keep debug symbols in mind: `objdump -t` gave `main` and the decoder straight away.
