---
title: Bad Calibration
tags:
  - DEADFACE CTF 2026
  - Medical Device
  - Linux
---

# Bad Calibration

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Medical Device |
| **Points** | 210 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

The description is the same text as [Live Boot](live-boot.md): a live shell on the device, and keys that aren't in the Field Manual. The title and the much bigger point value are the real hint. This one is about the scanner's calibration check.

## Solution

### The calibration files

From the Live Linux shell, with the data partition mounted (see [Live Boot](live-boot.md)):

```
cd /mnt/opt/scanworks/calibration
cat README
```

The README explains the whole scheme:

- `calib.json` holds the thresholds (`gain`, `anomaly_threshold`, `report_above`)
- `calib.json.sig` is the signature over `calib.json`
- the device won't scan unless the two match
- `scanctl verify` checks the signature, `scanctl scan` checks it again and runs the study

It also says how the signature is made:

```
sha256( <calib.json bytes> + <integrity.salt from config.yaml> )
```

### The bug

That's not a signature, it's a salted hash, and the salt is in a file anyone can read on the same partition:

```yaml
# /mnt/opt/scanworks/config.yaml
integrity:
  salt: EM-2019          # calibration signature salt
```

The developers knew. From `verify.py`:

```python
# salt is read from config.yaml integrity.salt (NOT a device secret).
# FIXME: replace with HMAC using the per-device key.
```

`EM-2019` is also the string the BLE maintenance log calls the vendor default ([Signals from the Skull](signals-from-the-skull.md)), so it's the same on every device. If you can write `calib.json`, you can compute a matching `.sig`, and the device says `signature verified`. The check catches corruption, not tampering.

### Hiding a finding

With the stock calibration, the loaded study gets flagged:

```
CALIBRATION OK (sig verified)  report_above=0.35
Patient: Gideon Marsh (MRN 100413)  study: Cranial perfusion
RESULT: ANOMALY DETECTED (score 0.72) — refer for radiologist review
```

We raised `report_above` above 0.72, recomputed the hash with the salt, and `scanctl verify` passed. The finding is gone from the report, and the simulator gives up the flag:

```
CALIBRATION OK (sig verified)  report_above=0.99
RESULT: No abnormality detected   [UNSAFE: finding 0.72 >= safe 0.35 suppressed]
integrity-audit: deadface{c4l1br4t10n_f0rg3ry_h4rms_l1v3s}
```

!!! warning "Why this one is worse"
    The earlier flags in the series are data leaks. This one changes what the doctor sees: the 0.72 perfusion finding just doesn't get reported, so the patient misses a diagnosis.

## Flag

!!! flag "Flag"
    `deadface{c4l1br4t10n_f0rg3ry_h4rms_l1v3s}`

## Takeaways

`sha256(data + salt)` with the salt sitting next to the data protects nothing. It needs an HMAC or a real signature with a key the attacker can't read. Also, read the `FIXME` comments in whatever code the vendor left on the device. That's how we found this one.

This challenge writes to the device's calibration files, and the target is shared, so put the original `calib.json` and `calib.json.sig` back when you're done.
