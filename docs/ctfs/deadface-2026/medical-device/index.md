---
title: Medical Device · DEADFACE CTF 2026
tags:
  - DEADFACE CTF 2026
  - Medical Device
---

# Medical Device

Writeups for the DEADFACE CTF 2026 medical device series. Every challenge targets the same simulated Emedra CerebraScan CS-9000 brain scanner at St. Dymphna's RMC (`http://emedra.dctf.io:8080/`). Put each writeup in this folder as `challenge-name.md`.

The device page is a 3D scene with clickable hotspots and a toolkit of attack tools (HDMI cable, Live Linux USB, Rubber Ducky, BLE sniffer). Everything is driven by a small JSON API, so the whole series can be solved with `curl` once you read the frontend bundle `/assets/index-CbamKmo7.js`:

| Endpoint | Purpose |
|---|---|
| `GET /api/device`, `GET /api/hotspot/{id}` | Device info, tools and hotspots |
| `POST /api/connect` `{tool, port}` | Plug a tool into a port, returns a session `token` |
| `GET /api/kiosk/login`, `POST /api/kiosk/auth` | ScanWorks touchscreen login |
| `GET /api/external-display?level=N` | HDMI mirror (Bearer token) |
| `POST /api/shell` `{token, line}` | Live Linux terminal |
| `POST /api/kiosk/hid` `{token, macro}` | Rubber Ducky payload |
| `GET /api/capture` | BLE sniffer capture (Bearer token) |

- [ScanWorks Login Source](scanworks-login-source.md)
- [Factory Settings](factory-settings.md) · 25 pts
- [Second Opinion](second-opinion.md) · 50 pts
- [Live Boot](live-boot.md) · 50 pts
- [Dump It](dump-it.md) · 150 pts
- [Bad Calibration](bad-calibration.md) · 210 pts
- [Signals from the Skull](signals-from-the-skull.md) · 300 pts
