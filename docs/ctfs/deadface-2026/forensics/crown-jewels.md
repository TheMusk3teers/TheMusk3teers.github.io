---
title: Crown Jewels
tags:
  - DEADFACE CTF 2026
  - Forensics
  - Traffic
---

# Crown Jewels

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Forensics / Traffic (De Monne Breach) |
| **Points** | 75 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> The attacker copied several documents from the executive share. What was the first PDF read from that share? Submit its full filename, including the extension.
>
> Submit the flag as `deadface{file.txt}`.
>
> Use the PCAP from *Uninvited Guest*.

Files given: `demonne.pcap`

## Solution

The attacker (`10.77.10.66`) connects to `\\10.77.10.30\executive` at frame 6734. After that we only care about SMB2 CREATE requests, which carry the file names:

```bash
tshark -r demonne.pcap -Y "smb2.cmd == 5 && frame.number > 6734" -T fields -e frame.number -e smb2.filename
```

| First read (frame) | File |
|---|---|
| 7409 | `annual_report.pdf` |
| 8328 | `executive_risk_register.xlsx` |
| 9092 | `board_transition_timeline.pdf` |
| 9744 | `succession_vote_draft.pdf` |

The first file read is a PDF, `annual_report.pdf`.

## Flag

!!! flag "Flag"
    `deadface{annual_report.pdf}`

## Takeaways

For SMB file theft, find the Tree Connect for the share (`smb2.cmd == 3`) and then list the CREATE (`5`) and READ (`8`) commands after it in frame order.
