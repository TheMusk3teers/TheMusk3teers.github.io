---
title: Who Goes There?
tags:
  - DEADFACE CTF 2026
  - Forensics
---

# Who Goes There?

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics (memory) |
| **Points** | 25 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[chaimae098](https://github.com/chaimae098) · Lyna · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) |

## Challenge

> DEADFACE managed to slip a rogue Kali device onto De Monne Financial's corporate network. Shortly afterward, someone used a legitimate employee account to access the company file share and steal sensitive documents. One of those documents later surfaced in Ghost Town. ZeroTrace recovered a packet capture and a memory image from the affected environment. Your first task is to identify the De Monne Financial employee account that DEADFACE compromised.

Files given: `memory.7z` (1.3 GB). It unpacks to `memory.raw`, an 8 GB raw memory image. This is the first challenge in the De Monne memory series, so keep the image for the later ones.

## Solution

### Setup

We checked the archive's SHA256 against the published value, extracted it to its own folder and installed Volatility 3 in a venv:

```bash
python -m venv vol3venv
vol3venv/Scripts/python -m pip install volatility3 pycryptodome
vol3venv/Scripts/vol -q -f memory.raw windows.info
```

`windows.info` identifies a 64-bit **Windows 11 22H2** (NT 10.0, build 22621) machine with one CPU. The system time is **2026-09-27 20:40:44 UTC**. The first run downloads symbols and builds a cache, so it takes a few minutes.

We then dumped the useful plugins to text files so we could search them:

```bash
for p in windows.sessions windows.pstree windows.cmdline windows.netscan \
         windows.envars windows.hashdump windows.lsadump; do
  vol -q -f memory.raw $p > $p.txt
done
```

### Who is logged on

`windows.sessions` shows one interactive user. `explorer.exe` (PID 4868) runs in console session 1 as `DMF-WS11-042/nkessler`. Edge, OneDrive and the other desktop apps belong to the same account.

### The remote logon

Session 0 holds something more interesting: an OpenSSH logon as the same user, about 20 minutes before the capture.

```
4996  sshd.exe        -                  2026-09-27 20:21:51
3532  sshd.exe        WORKGROUP/nkessler 2026-09-27 20:21:55   "sshd.exe" -z
5424  conhost.exe     WORKGROUP/nkessler 2026-09-27 20:21:55   --headless ... cmd.exe
8120  cmd.exe         WORKGROUP/nkessler 2026-09-27 20:21:55
3300  powershell.exe  WORKGROUP/nkessler 2026-09-27 20:21:58   powershell
```

`sshd.exe -z` is the per-connection child process that OpenSSH for Windows starts for each logon. `windows.envars` on those processes shows where the connection came from:

```
3532  sshd.exe  SSH_CONNECTION  10.77.10.66 50742 10.77.10.20 22
3532  sshd.exe  USERNAME        nkessler
3532  sshd.exe  COMPUTERNAME    DMF-WS11-042
```

The workstation is `10.77.10.20`. Someone at `10.77.10.66`, almost certainly the rogue Kali device, logged in with nkessler's valid credentials and got a cmd and a PowerShell shell. That makes **nkessler** the compromised employee account.

The other late processes are the responders: PowerShell 7472 started `go-winpmem` (PID 9032) at 20:39 to write `E:\Acquisition\memory.raw`, which is this image.

### Other findings

- `windows.hashdump` lists two real local accounts: `nkessler` (RID 1001) and `dmf_support` (RID 1002). Administrator, Guest and DefaultAccount have the empty-password NT hash `31d6cfe0...`. `lsadump` found no LSA secrets.
- The host is in a **WORKGROUP**, not a domain, so this is a local account. If `windows.getsids` shows SID `...-513` as "Domain Users", that label is generic: on a non-domain machine RID 513 is the local "None" group.
- `windows.netscan` only recovered two stale TCP 443 sockets, so the SSH and SMB connections aren't in it. We found no UNC share paths in the memory strings either. The file-share logon should show up in the packet capture (`ntlmssp.auth.username`, SMB2 Session Setup from 10.77.10.66).

## Flag

!!! flag "Flag"
    `deadface{nkessler}`

## Takeaways

Check every logon to the machine, not just the interactive one. Here the console user and the attacker were the same account, and the source IP was in the SSH process's environment variables (`SSH_CONNECTION`). Run `windows.envars` early: it holds the username, hostname and sometimes the client IP.
