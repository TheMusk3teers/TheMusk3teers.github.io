---
title: Keypad Oracle
tags:
  - DEADFACE CTF 2026
  - Mobile
  - Reverse Engineering
---

# Keypad Oracle

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | iOS reverse engineering |
| **Points** | 200 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> An iOS app where the right keypad code reveals a secret message.

Files given: `Keypad-Oracle.app.zip`
Zip SHA256: `7bfda8f6fa3bebbe29a6e18c4312a29ec9a3cdfda394eb01d39a59df849af49e`

## Solution

### 1. Triage

```bash
unzip Keypad-Oracle.app.zip
file Keypad-Oracle.app/*
```

The bundle holds:

| File | What it is |
|---|---|
| `Info.plist` | Binary plist. Bundle ID `com.deadface.keypadoracle.rick`, platform `iPhoneSimulator`, SDK `iphonesimulator26.4` |
| `Keypad-Oracle` | Small arm64 Mach-O stub (58 KB) |
| `Keypad-Oracle.debug.dylib` | arm64 dylib (343 KB) with all the Swift code. Xcode's debug builds put the app here |
| `KeypadConfig.json` | One 27-byte hex blob per digit `0` to `9` |
| `__preview.dylib` | SwiftUI previews helper, not useful |

```json
{
  "digits": {
    "0": "8e07513d09e6379a5b14f845137456c5f4974967c678c07e1a9025",
    "1": "d179fa8cf908d9ed599ab5967c4f8d362de70c1a791babc9b75bcc",
    ...
    "9": "5992f0ee2c8c682daa6038ef393108b28a40548fc97f99b33f1d88"
  }
}
```

The strings in the dylib give us the rules of the UI:

```text
Enter a 4-digit PIN. Digits may not repeat.
Repeated digits are not allowed.
The answer you seek is...
Missing KeypadConfig.json
```

### 2. Symbols

It's a debug build, so the Swift symbols are still there. `llvm-nm` shows a `PinCipherEngine` class:

```bash
llvm-nm-19 Keypad-Oracle.debug.dylib | grep PinCipherEngine
```

```text
PinCipherEngine.outputLength : Int
PinCipherEngine.binarySalt   : [UInt8]
PinCipherEngine.seedTable    : [Int : [UInt8]]
PinCipherEngine.loadSeedTable() -> [Int : [UInt8]]
PinCipherEngine.encode(pin: [Int]) -> String
PinCipherEngine.derive(seed: [UInt8], digit: Int, position: Int) -> [UInt8]
PinCipherEngine.xor(_: [UInt8], _: [UInt8]) -> [UInt8]
PinCipherEngine.rotateLeft(_: UInt8, by: UInt8) -> UInt8
[UInt8].toHexString() -> String
```

`seedTable` is `KeypadConfig.json` decoded from hex. Each digit has its own seed.

### 3. Disassembly

The build isn't optimised, so `llvm-objdump-19 -d` gives readable code, with an overflow check after every arithmetic operation.

**Constants.** The initialiser for `outputLength` returns `0x1b` (27). The initialiser for `binarySalt` stores 28 bytes:

```text
3a 91 44 2f d0 18 6b c3 72 0e a9 5d 11 be 80 27 f4 66 39 9a 05 cc 70 1b 88 42 e1 53
```

**`encode(pin:)`**

1. Return an empty string unless `pin.count == 4` and `Set(pin).count == 4`, so 4 digits with none repeated.
2. Start with `acc = [UInt8](repeating: 0, count: 27)`.
3. For each `(position, digit)` in the PIN, set `acc = xor(acc, derive(seedTable[digit], digit, position))`.
4. Set `acc = xor(acc, binarySalt)`. `xor` zips the two arrays, so the 28th salt byte is dropped.
5. Return `acc.toHexString()`.

**`derive(seed:digit:position:)`** loops `i` over `0..<27`. The key instructions are `mul #0x21` (33), `mul #0x13` (19), `mul #0x7`, three `eor`s, `sdiv #8` before `rotateLeft`, then `mul #0xb` (11) and a last `eor`:

```swift
var v = seed[i % seed.count]
      ^ UInt8((position + 1) * 33)
      ^ UInt8(truncatingIfNeeded: digit * 19)
      ^ UInt8(truncatingIfNeeded: i * 7)
v = rotateLeft(v, by: UInt8((digit + position + i) % 8))
out[i] = v ^ UInt8(truncatingIfNeeded: i + digit + position * 11)
```

Each digit and its position give a 27-byte keystream. The four keystreams and the salt are XORed together, and the result is shown as hex. A wrong PIN still gives output, just garbage bytes. The right PIN gives readable ASCII.

### 4. Brute force

There are only 10 × 9 × 8 × 7 = 5040 PINs with no repeated digit. So reimplement the cipher and keep any PIN whose output is all printable:

```python
import json, sys, itertools

cfg = json.load(open(sys.argv[1]))          # KeypadConfig.json
seeds = {int(k): bytes.fromhex(v) for k, v in cfg["digits"].items()}
SALT = bytes.fromhex("3a91442fd0186bc3720ea95d11be8027f466399a05cc701b8842e153")
N = 27

def rotl(v, r):
    return ((v << r) | (v >> (8 - r))) & 0xff if r else v

def derive(seed, d, pos):
    out = []
    for i in range(N):
        v = seed[i % len(seed)] ^ ((pos + 1) * 33) ^ ((d * 19) & 0xff) ^ ((i * 7) & 0xff)
        v = rotl(v, (d + pos + i) % 8)
        out.append(v ^ ((i + d + pos * 11) & 0xff))
    return bytes(out)

def xor(a, b):
    return bytes(x ^ y for x, y in zip(a, b))

def encode(pin):
    acc = bytes(N)
    for pos, d in enumerate(pin):
        acc = xor(acc, derive(seeds[d], d, pos))
    return xor(acc, SALT)

for pin in itertools.permutations(range(10), 4):
    m = encode(pin)
    if all(32 <= c < 127 for c in m):
        print("".join(map(str, pin)), m.decode())
```

```text
$ python3 solve.py Keypad-Oracle.app/KeypadConfig.json
3719 deadface{your-pin-is-valid}
```

Only one PIN gives printable output. The keypad code is **3719**. In the app it shows as the hex string `toHexString()` returns, and decoding that hex gives the flag.

## Flag

!!! flag "Flag"
    `deadface{your-pin-is-valid}`

## Takeaways

Xcode debug builds put the real code in `*.debug.dylib` and keep the Swift symbols, so start there. When a check only runs on a small keyspace like a 4-digit PIN, reimplement it and brute-force it rather than reversing it by hand.
