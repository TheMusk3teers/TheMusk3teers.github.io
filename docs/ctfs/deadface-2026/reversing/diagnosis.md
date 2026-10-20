---
title: Diagnosis
tags:
  - DEADFACE CTF 2026
  - Mobile
  - Reverse Engineering
---

# Diagnosis

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Mobile reverse engineering (De Monne Slayer) |
| **Points** | 100 |
| **Author** | @syyntax |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> Analyze the De Monne Financial Android application for an exposed diagnostic component left enabled in the build.
>
> Submit the flag as `deadface{flag_text}`.

Files given: `app-release.zip` (`app-release.apk`, same APK as [Hard Kept Secrets](hard-kept-secrets.md), [Inside Man](inside-man.md), and [No Holding Back](../osint/no-holding-back.md))

## Solution

### 1. Inspecting `AndroidManifest.xml`

Decompiling `app-release.apk` with `jadx` and inspecting `AndroidManifest.xml` reveals an internal activity that was explicitly marked `android:exported="true"`:

```xml
<activity
    android:name="com.demonne.financial.internal.DiagnosticActivity"
    android:exported="true" />
```

### 2. Inspecting `DiagnosticActivity`

Opening `com.demonne.financial.internal.DiagnosticActivity` in `jadx` (or launching it directly via `adb shell am start -n com.demonne.financial/.internal.DiagnosticActivity`) reveals the hardcoded diagnostic flag:

```text
deadface{flaG_ManIfesT3d}
```

## Flag

!!! flag "Flag"
    `deadface{flaG_ManIfesT3d}`

## Takeaways

Always check `AndroidManifest.xml` for activities, services, or broadcast receivers with `android:exported="true"` that aren't reachable from the normal UI navigation flow—any app on the device (or `adb shell am start`) can invoke an exported activity directly.
