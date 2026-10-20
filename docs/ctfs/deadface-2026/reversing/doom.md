---
title: DOOM
tags:
  - DEADFACE CTF 2026
  - Reverse Engineering
---

# DOOM

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Bonus |
| **Points** | 25 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Alright, so this isn't really cybersecurity-related. You are playing Doom, but I mean how can you be mad when playing Doom?... Simply download the zip file, extract it. Then open a command prompt, change into the extracted directory and run the following command. The game should load, you hit new game and the difficulty doesn't matter. The flag is pretty obvious at the end you can't miss it.
>
> ```bash
> gzdoom -iwad freedoom2.wad -file MAP01.wad
> ```
>
> Submit the flag as `deadface{flag-text}`.

Files given: `DOOM.zip` (32 MB, SHA256: `48c560f08aa36cb421d9afa0619644d184e252f93571e9ae23d231cf6f306a0f`)

## Solution

### 1. Playing (or inspecting) `MAP01.wad`

Extracting `DOOM.zip` and launching the custom level with `gzdoom -iwad freedoom2.wad -file MAP01.wad` (or inspecting the end-of-level sector geometry in a WAD editor / using the `noclip` cheat code in `gzdoom` to walk straight to the exit room) reveals the flag spelled out at the end of `MAP01`:

```text
deadface{Lets_Play_Doom}
```

## Flag

!!! flag "Flag"
    `deadface{Lets_Play_Doom}`

## Takeaways

For custom Doom `.wad` challenges where the flag is built into the level geometry rather than stored as an ASCII string lump, either open the `LINEDEFS`/`VERTEXES` in SLADE / a 2D Python plotter, or use `gzdoom`'s built-in console cheats (`god`, `noclip`) to fly straight to the exit sector.
