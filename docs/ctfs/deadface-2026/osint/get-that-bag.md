---
title: Get That Bag
tags:
  - DEADFACE CTF 2026
  - OSINT
---

# Get That Bag

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT / Crypto / Stego |
| **Points** | 350 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> DEADFACE discovered critical information about *the mark*. How much is their victim's salary?
>
> Note: this is a continuation of The Mark.
>
> Submit the flag as `deadface{$#}`. Example: `deadface{$1000}`.

## Solution

The answer is in `tgri_records.csv`, decrypted in [The Mark](the-mark.md) with the pastebin passphrase. In record `SYN-TGRI-001`, Santos Staudenmaier (Chief Financial Officer) has `annual_base_salary_usd` = **540000**. It's his only salary entry.

Other figures in his rows that are not the salary:

| Value | What it is |
|---|---|
| $243,000 | bonus target |
| $201,000 | bonus actually paid after a $42,000 board holdback |
| $620,000 | Mara Quenley's salary (different person) |

## Flag

!!! flag "Flag"
    `deadface{$540000}`

## Takeaways

Keep decrypted artifacts from earlier challenges; follow-ups often only ask you to read one more field.
