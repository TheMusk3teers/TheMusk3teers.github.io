---
title: Inside Man
tags:
  - DEADFACE CTF 2026
  - Mobile
  - Reverse Engineering
---

# Inside Man

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Mobile reverse engineering |
| **Points** | 20 |
| **Author** | TODO |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

Find what's hardcoded in the De Monne Financial app that gives elevated or developer access, for example a dev account, a role flag or a debug endpoint. Flag format: `deadface{flag_text}`.

Files given: `app-release.zip`, which contains `app-release.apk` (the same file as [Hard Kept Secrets](hard-kept-secrets.md)).
Zip SHA256: `5530926b2e564615285d15674ddb21c316f673d7d240f9fcfa36c6afb99f5c0c`

## Solution

### 1. Decompile

```bash
unzip app-release.zip
jadx -d jadx_out "$PWD/app-release.apk"
apktool d -o apktool_out app-release.apk
```

### 2. The backdoor in the login screen

`LoginActivity` has two interesting constants:

```java
public class LoginActivity extends AppCompatActivity {
    private static final String INTERNAL_ACCOUNT = "audit@demonne-financial.local";
    private static final String INTERNAL_PASS    = "demonnefinancial";
```

`attemptLogin()` checks for them before doing a normal login:

```java
if (strTrim.equalsIgnoreCase(INTERNAL_ACCOUNT) && string.equals(INTERNAL_PASS)) {
    performInternalLogin(strTrim, string);
} else {
    ApiClient.getService().login(new LoginRequest(strTrim, string))...
}
```

`performInternalLogin()` also calls `POST /api/auth/login`, so this is a real server-side account. The developers left an internal "audit" account and its password in the client.

### 3. Use it

`ApiService` shows the endpoint, and `BuildConfig.API_BASE_URL` gives the host. Send the same request the app makes:

```bash
curl -s -X POST http://dmf.dctf.io:8081/api/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"audit@demonne-financial.local","password":"demonnefinancial"}'
```

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "email": "audit@demonne-financial.local",
    "id": 5,
    "message": "deadface{l3FT_b3Hind!}",
    "name": "Audit System"
  }
}
```

The app would show this `message` on the dashboard after login. Here we just read it from the JSON.

### Red herrings in the same APK

The APK has two other `deadface{...}` strings. They're the answers to other challenges, not this one:

- `internal/DiagnosticActivity` is `exported="true"` in the manifest and prints `deadface{flaG_ManIfesT3d}`.
- `SecureVaultActivity` hides `deadface{Ur_N0t_A110wEd_h3r3}` behind a root and emulator check.

## Flag

!!! flag "Flag"
    `deadface{l3FT_b3Hind!}`

## Takeaways

Search login code for `equals`/`equalsIgnoreCase` comparisons against constants. A client-side "if email == X" branch almost always means a backdoor account. If it calls the real API, the credentials work against the backend too.
