---
title: The Typo That Bit
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# The Typo That Bit

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics (memory) |
| **Points** | 75 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Nathan's workstation tried to reach an executive share through a misspelled server name. Identify one mistyped server hostname visible in memory. Submit the hostname, without slashes or the share name.

Files given: the `memory.raw` image from [Who Goes There?](who-goes-there.md).

## Solution

Volatility doesn't help much here: `windows.netscan` holds no SMB sockets, and a plain ASCII grep for `\\server\share` finds nothing useful. Windows stores most paths as **UTF-16LE**, so we scanned the raw image for UNC paths in ASCII, UTF-16LE, and the `##server#share` form that Explorer uses in the MountPoints2 registry key:

```python
import mmap, re, sys
from collections import Counter

pat_a = re.compile(rb'\\\\([A-Za-z0-9][A-Za-z0-9.\-]{1,40})\\([A-Za-z0-9$ _.\-]{1,40})')
pat_w = re.compile(rb'\\\x00\\\x00((?:[A-Za-z0-9.\-]\x00){2,41})\\\x00((?:[A-Za-z0-9$ _.\-]\x00){1,40})')
pat_m = re.compile(rb'#\x00#\x00((?:[A-Za-z0-9.\-]\x00){2,41})#\x00((?:[A-Za-z0-9$ _.\-]\x00){1,40})')

c = Counter()
with open(sys.argv[1], 'rb') as f:
    mm = mmap.mmap(f.fileno(), 0, access=mmap.ACCESS_READ)
    for kind, pat, enc in (('A', pat_a, 'ascii'), ('W', pat_w, 'utf-16le'), ('M', pat_m, 'utf-16le')):
        for m in pat.finditer(mm):
            c[(kind, m.group(1).decode(enc), m.group(2).decode(enc))] += 1
for k, n in c.most_common():
    print(n, *k)
```

Most hits are noise (Defender signatures, `\\?\` paths, IPs from threat intel). Filtering for De Monne's `DMF-` naming scheme leaves:

```
6  W  DMF-WS11-042  ROOT          <- the workstation itself
3  W  DMF-FIELD-01  executive
2  W  DMF-FIELD-01  execuHome     <- fragment of a longer path
1  W  DMF-FIELS-01  exec          <- the typo
```

To tell the real server from the typo, we pulled the UTF-16 text around each name. Both names appear in NTLM authentication data, with a FQDN, a NetBIOS name and the same GUID:

```
dmf-field-01.local  DMF-FIELD-01  dmf-field-01.local  NTLM  {28bfc32f-10f6-4738-98d1-1ac061df716a}
dmf-fiels-01.local  DMF-FIELS-01  dmf-fiels-01.local  NTLM  {28bfc32f-10f6-4738-98d1-1ac061df716a}
```

A second scan for any `dmf-`-style name in either case shows the lowercase forms are by far the most common, and turns up a second typo:

```
218 W dmf-field-01    <- real server
 97 W dmf-fiels-01    <- typo: D -> S
  2 W dmf-fiel-01     <- typo: missing D
  1 W DMF-FIELS-01
```

`DMF-FIELD-01` also shows up in the workstation's ASCII name table next to `DMF-WS11-042`, and the `executive` share is on it, so it's the real file server. `dmf-fiels-01` has **D** swapped for **S**, its neighbor on the keyboard. That's the misspelled server name Nathan's machine tried to reach.

The flag is case-sensitive. Our first scan uppercased every hostname, and `deadface{DMF-FIELS-01}` was rejected. The accepted answer uses the lowercase form, which is how the name appears in most of its 97 copies.

## Flag

!!! flag "Flag"
    `deadface{dmf-fiels-01}`

## Takeaways

Search memory in UTF-16LE as well as ASCII: Windows paths, registry values and SMB/NTLM names are almost always wide strings. Once a naming scheme like `DMF-` is known, filtering hits by that prefix cuts the noise. Never normalize case while counting: submit the form that appears most often in memory.
