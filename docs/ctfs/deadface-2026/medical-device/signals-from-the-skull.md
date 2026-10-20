---
title: Signals from the Skull
tags:
  - DEADFACE CTF 2026
  - Medical Device
  - BLE
---

# Signals from the Skull

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Medical Device |
| **Points** | 300 |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

The CS-9000 chatters constantly over Bluetooth Low Energy for remote maintenance. Most of the traffic is boring vitals telemetry, but one characteristic is quietly dribbling out a maintenance log a few bytes at a time. Capture it, reassemble it, and decode it.

## Solution

### Capturing

Plug the BLE Sniffer into USB-A and read the capture:

```bash
curl -s -X POST http://emedra.dctf.io:8080/api/connect \
  -H 'content-type: application/json' -d '{"tool":"ble-sniffer","port":"usb-a"}'
curl -s http://emedra.dctf.io:8080/api/capture -H "authorization: Bearer <TOKEN>"
```

Each packet has a `handle`, a `seq` and a hex `payload`:

```json
{"handle":"0x2a","seq":69,"payload":"9fdb212e1da3"}
{"handle":"0x37","seq":0,"payload":"5533517552486c7463476875"}
```

### Which handle

There are two handles mixed together:

- `0x2a`: lots of packets, 6 random-looking bytes each. That's the "boring vitals" noise, and it doesn't decode to anything.
- `0x37`: only 24 packets of 12 bytes, and every byte is printable ASCII. That's the log.

### Putting it back together

The packets are shuffled, which is what `seq` is for. Sort the `0x37` packets by `seq`, join the payloads and un-hex them. You get ASCII that's clearly Base64:

```python
import json, base64
d = json.load(open('ble.json'))
p = sorted((x for x in d['packets'] if x['handle'] == '0x37'), key=lambda x: x['seq'])
b64 = bytes.fromhex(''.join(x['payload'] for x in p)).decode()
print(base64.b64decode(b64 + '==').decode())
```

```
St.Dymphna CS-9000 maint log | site=stdymphna-rmc-01 | tech=emedra-svc
last_patient: Gideon Marsh (MRN 100413)
note: telemetry unencrypted per EM-2019 default; flag deadface{un3ncrypt3d_b4bbl3_fr0m_th3_sku11}
```

The log ties into the rest of the series too: the `emedra-svc` account, the patient from the HDMI dump, and the `EM-2019` default that comes back as the calibration salt.

## Flag

!!! flag "Flag"
    `deadface{un3ncrypt3d_b4bbl3_fr0m_th3_sku11}`

## Takeaways

Split a BLE capture by handle before decoding anything, and sort by sequence number. Payloads that are all printable ASCII are the ones worth decoding.
