---
title: Ghost of Squares Past
tags:
  - DEADFACE CTF 2026
  - OSINT
---

# Ghost of Squares Past

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT |
| **Points** | 150 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Intelligence gathered from DEADFACE communications suggests that the group has been using Savannah, Georgia's historic squares to conceal a coded message. Investigators recovered five clues, a combination of riddles and photographs, each pointing to a specific location within the city's historic district.
>
> DEADFACE appears to have arranged these locations into a compass pattern, with the first four representing North, East, South, and West, and the fifth marking the center.
>
> For the first four locations, determine the year each square was established. For the final location, identify the surname engraved on its monument.
>
> Flag Format: `deadface{YEAR-YEAR-YEAR-YEAR-SURNAME}`

## Solution

### North: Johnson Square

> *"First of them all, where the city began, A banker's green heart in Oglethorpe's plan. A Quaker from Rhode Island sleeps under stone here, Mark the year this first square was laid out, and draw near."*

Savannah's first square, in the banking district. General Nathanael Greene, a Quaker from Rhode Island, is buried under the obelisk in the middle. Laid out in **1733**.

### South: Chippewa Square

> *"Just a ways away a battle of a northern frontier Gave its name to this ground eighteen-twelve drawing near. Where a feather once drifted and a founder stands tall, Mark the year it was laid out, the newest of all."*

Named after the Battle of Chippawa in the War of 1812. It's the Forrest Gump feather bench square, and the Oglethorpe statue is there. Laid out in **1815**.

### Center: Wright Square

With North and South both on Bull Street, the center is the square between them, Wright Square. Its monument is for William Washington Gordon, president of the Central of Georgia Railway, so the surname is **gordon**.

### East and West

The squares on either side of Wright Square are Oglethorpe Square to the east, laid out in **1742**, and Telfair Square to the west, first laid out as St. James Square in **1733**.

| Point | Square | Value |
|---|---|---|
| 1 · North | Johnson | 1733 |
| 2 · East | Oglethorpe | 1742 |
| 3 · South | Chippewa | 1815 |
| 4 · West | Telfair | 1733 |
| 5 · Center | Wright (Gordon monument) | gordon |

## Flag

!!! flag "Flag"
    `deadface{1733-1742-1815-1733-gordon}`

## Takeaways

Savannah's squares sit on a grid. Once we had North and South on Bull Street, the center and the east and west squares were just a look at the map.
