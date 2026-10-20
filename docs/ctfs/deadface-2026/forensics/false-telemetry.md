---
title: False Telemetry
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# False Telemetry

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics (memory) |
| **Points** | 250 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> WindowsTelemetryCheck runs a PowerShell script that repeatedly tests a connection. Identify the destination IPv4 address, TCP port, and delay between attempts.
>
> Submit the flag as `deadface{IP_PORT_SECONDS}`.

Files given: none new. It uses the `memory.raw` image from [Who Goes There?](who-goes-there.md).

## Solution

[Cache Money](cache-money.md) found the script path, `C:\ProgramData\WindowsCache\telemetry.ps1`. The script body is still in the file cache, next to its name. A Python `mmap` search of the raw image for `Test-NetConnection` and `Start-Sleep` near `telemetry` recovered it:

```python
import mmap, re
f = open('memory.raw', 'rb')
m = mmap.mmap(f.fileno(), 0, access=mmap.ACCESS_READ)
for h in re.finditer(rb'[\x20-\x7e\r\n\t]{0,300}Test-NetConnection[\x20-\x7e\r\n\t]{0,300}', m):
    print(h.start(), h.group().decode())
```

```powershell
$ErrorActionPreference = 'SilentlyContinue'
for ($i = 0; $i -lt 24; $i++) {
    Test-NetConnection -ComputerName 167.172.132.224 -Port 4815 -InformationLevel Quiet | Out-Null
    Start-Sleep -Seconds 30
}
```

The script tries `167.172.132.224` on TCP `4815` 24 times, 30 seconds apart.

Two other hits look similar but aren't the answer:

- `Test-NetConnection -ComputerName 167.172.132.224 -Port 22` followed by `Write-Output "drop_tcp22=$ok"`: a one-off reachability check, not the scheduled script.
- `Test-NetConnection - 10.77.10.1:22`: a PowerShell progress record from an unrelated test.

!!! note "Windows Defender"
    Defender blocked the first search because its output contained these strings. Spacing out the output characters got around it.

## Flag

!!! flag "Flag"
    `deadface{167.172.132.224_4815_30}`

## Takeaways

Small scripts often survive whole in memory. Search for the cmdlets they must use (`Test-NetConnection`, `Start-Sleep`), and tell the loop apart from one-off commands typed in the console.
