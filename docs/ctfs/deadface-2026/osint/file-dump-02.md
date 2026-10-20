---
title: File Dump 02
tags:
  - DEADFACE CTF 2026
  - OSINT
  - Forensics
---

# File Dump 02

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT |
| **Points** | 75 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Now we know *who* is responsible for impersonating Daniel. Can you dig through these documents and maybe find a *where*? We are pretty sure Daniel had a computer stolen so where could that have happened?
>
> Submit the flag as `deadface{store_name}`. Example: `deadface{texas_tires}`.

Files given: `File_Dump_Challenge.zip` (same archive as [File Dump 01](../steganography/file-dump-01.md) and [File Dump 03](../steganography/file-dump-03.md))

## Solution

Three files in the dump narrow it down:

1. `reciept_1.png`: a water-stained receipt for computer repair, dated August 14, 2026, from store/suite 17.
2. `snip.png`: a cropped piece of a store directory that says `#17 · Apple Specialist`.
3. `businesscard_1.png`: Daniel's phone number is in the 907 area code (Anchorage, Alaska), and there's an ad for Golden Donuts in Anchorage. `notes.docx` and `reminders.docx` also mention grabbing coffee nearby.

So we wanted an Apple Specialist in suite 17 in Anchorage, close to Golden Donuts. That's Alaska MacStore, 2217 E. Tudor Rd., Ste. 17, Anchorage, AK 99507, phone `907-272-7697`. It's just down Tudor Road from Golden Donuts.

!!! note "Wrong answers we tried"
    `deadface{apple_specialist}` (the tagline from the crop) and `deadface{golden_donuts}` (the donut shop) were both rejected. It wants the store's actual name.

## Flag

!!! flag "Flag"
    `deadface{alaska_macstore}`

## Takeaways

Put all the small clues together (suite number, tagline, landmark, area code) and look for the one real business that fits all of them. Don't submit the generic text from the image.
