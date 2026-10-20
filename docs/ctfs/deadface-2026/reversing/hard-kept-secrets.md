---
title: Hard Kept Secrets
tags:
  - DEADFACE CTF 2026
  - Mobile
  - Reverse Engineering
---

# Hard Kept Secrets

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Mobile reverse engineering |
| **Points** | 20 |
| **Author** | TODO |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

Recover the internal API token hardcoded in the De Monne Financial Android app. The flag is `deadface{<token>}`, and the token looks like base64.

Files given: `app-release.zip`, which contains `app-release.apk`.
Zip SHA256: `5530926b2e564615285d15674ddb21c316f673d7d240f9fcfa36c6afb99f5c0c`

The same APK is used for [Inside Man](inside-man.md).

## Solution

### 1. Decompile

```bash
unzip app-release.zip
jadx -d jadx_out "$PWD/app-release.apk"       # Java sources
apktool d -o apktool_out app-release.apk      # manifest, res/, assets/
```

On Kali, the `jadx` wrapper changes into its install directory before it runs, so pass it absolute paths.

The app code is in `com.demonne.financial`. The usual places for secrets turned up nothing useful:

- `res/values/strings.xml` only has UI strings.
- `assets/` only has `dexopt/baseline.prof`.
- `BuildConfig` only has the API base URL `http://dmf.dctf.io:8081` and build metadata.

### 2. Follow the networking code

The token has to be attached to requests somewhere, so look at the Retrofit setup in `api/ApiClient.java`:

```java
public class ApiClient {
    private static final String INTERNAL_API_TOKEN =
        "zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR";

    public static Retrofit getInstance() {
        ...
        .client(new OkHttpClient.Builder()
            .addInterceptor(new InternalTokenInterceptor(INTERNAL_API_TOKEN))
            .addInterceptor(new AuthInterceptor())
            ...
```

`InternalTokenInterceptor` adds the token to every outgoing request:

```java
return chain.proceed(chain.request().newBuilder()
        .header("X-Internal-Token", this.token).build());
```

So every copy of the app sends the same static "internal" credential in an `X-Internal-Token` header. Anyone with the APK, or anyone watching the cleartext HTTP traffic (`usesCleartextTraffic="true"`), can read it.

The value is 64 base64 characters (48 bytes once decoded) and doesn't decode to readable text. The challenge asks for the token as it appears in the app, so no decoding is needed.

```bash
grep -rn "INTERNAL_API_TOKEN\|X-Internal-Token" jadx_out/sources/com/demonne
```

## Flag

!!! flag "Flag"
    `deadface{zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR}`

## Takeaways

When `strings.xml` and `BuildConfig` are clean, follow the HTTP client. Look at OkHttp interceptors and Retrofit builders for static headers like `X-Api-Key`, `X-Internal-Token` or `Authorization`. A secret shipped in the client is not a secret.
