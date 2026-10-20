---
title: Bedside Manner (ScanWorks Login Source)
tags:
  - DEADFACE CTF 2026
  - Medical Device
---

# Bedside Manner (ScanWorks Login Source)

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Medical Device |
| **Points** | 15 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> DEADFACE is taking credit for an attack that compromised patient data at St Dymphna's Medical Center... Your first task is to identify any lingering strings developers may have left in the device's kiosk web browser.

Access: `http://emedra.dctf.io:8080/`

## Solution

Clicking the touchscreen hotspot loads the ScanWorks login form. The frontend gets that form as raw HTML from `/api/kiosk/login`, so we just read the source:

```bash
curl -s http://emedra.dctf.io:8080/api/kiosk/login
```

```html
<div class="sw-status">Active study: <b>G.M.</b> · MRN ****13 · device locked</div>
<!-- TODO(dev): REMOVE before go-live — temporary QA technician PIN: deadface{v13w_s0urc3_sav3s_l1v3s} -->
```

Someone's "remove before go-live" note made it to go-live.

## Flag

!!! flag "Flag"
    `deadface{v13w_s0urc3_sav3s_l1v3s}`

## Takeaways

View source on kiosk UIs first. The BLE, HDMI and default creds flags you'll run into on the same device are for the other challenges in the series, not this one.
