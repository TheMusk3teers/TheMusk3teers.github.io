---
title: Hidden in Plain Task
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# Hidden in Plain Task

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics · Memory (De Monne Breach) |
| **Points** | 75 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> An attacker registered a task to keep activity going after leaving the workstation. What is the task's exact name? Submit the name as it appears in memory.
>
> Submit the flag as `deadface{TaskName}`.
>
> Use the memory dump evidence from *Who Goes There?*

Files given: `memory.7z` (`memory.raw`, 8 GB Windows 11 memory image)

## Solution

By this point we already knew from [Cache Money](cache-money.md) that the attacker built a task around `telemetry.ps1`. Volatility's task plugin couldn't read the registry, so we went back to strings:

```bash
strings -n 8 memory.raw | grep -E "ScheduledTask|telemetry\.ps1"
```

The full set of commands is still in memory:

```powershell
$action    = New-ScheduledTaskAction -Execute 'powershell.exe' -Argument '-NoProfile -File C:\ProgramData\WindowsCache\telemetry.ps1'
$trigger   = New-ScheduledTaskTrigger -Daily -At '23:59'
$principal = New-ScheduledTaskPrincipal -UserId 'SYSTEM' -LogonType ServiceAccount -RunLevel Highest
Register-ScheduledTask -TaskName 'WindowsTelemetryCheck' -Action $action -Trigger $trigger -Principal $principal -Force
Start-ScheduledTask -TaskName 'WindowsTelemetryCheck'
```

The task runs the script as SYSTEM every night at 23:59, and it's called `WindowsTelemetryCheck`. A name that blends in with real Windows tasks.

## Flag

!!! flag "Flag"
    `deadface{WindowsTelemetryCheck}`

## Takeaways

If the task XML or registry keys are paged out, grep memory for `Register-ScheduledTask` or `schtasks`. The `-TaskName` is right there on the command line.
