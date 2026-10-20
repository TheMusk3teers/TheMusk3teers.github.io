---
title: Secure Notes Lite
tags:
  - DEADFACE CTF 2026
  - Mobile
  - Reverse Engineering
---

# Secure Notes Lite (MOB06)

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Mobile reverse engineering |
| **Points** | TODO |
| **Author** | @G2Gh0st |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[chaimae098](https://github.com/chaimae098) · Lyna · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) |

## Challenge

Files given: `df-securenotes.apk.zip`, which contains `df-securenotes.apk`.
Zip SHA256: `b5f6e9a0a75f404103fc2d08542165015f8934e8d76433a513ca1a4705f006ca`

## Solution

### 1. Triage

```bash
unzip df-securenotes_apk.zip
file df-securenotes.apk
# Android package (APK), with AndroidManifest.xml, with APK Signing Block
```

The APK is a small, standard package: `AndroidManifest.xml`, three layouts (`activity_main`, `activity_note_detail`, `dialog_password`), `resources.arsc`, `classes.dex` and `classes2.dex` (multidex, mostly the `R` resource classes).

### 2. The manifest

Decoded with `androguard`:

```xml
<manifest package="com.deadface.securenoteslite" ...>
  <application android:label="Secure Notes Lite"
               android:debuggable="true"
               android:allowBackup="false" ...>
    <activity android:name="...NoteDetailActivity" android:exported="false"/>
    <activity android:name="...MainActivity" android:exported="true">
      <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
      </intent-filter>
    </activity>
  </application>
</manifest>
```

There are only two real classes, `MainActivity` and `NoteDetailActivity`. With a surface that small, it's quicker to decompile both in full than to hunt for strings.

### 3. Decompiling

```python
from androguard.misc import AnalyzeAPK
a, d, dx = AnalyzeAPK("df-securenotes.apk")

for cls_name in ["Lcom/deadface/securenoteslite/MainActivity;",
                 "Lcom/deadface/securenoteslite/NoteDetailActivity;"]:
    cls = dx.get_class_analysis(cls_name).get_vm_class()
    print(cls.get_source())
```

The interesting parts of `MainActivity`:

```java
private String getSecureBackupText() {
    return new String(
        android.util.Base64.decode("ZGVhZGZhY2V7ZXotbW9iaWxlLXRvLXN0YXJ0fQ==", 0),
        java.nio.charset.StandardCharsets.UTF_8
    );
}

private synthetic void lambda$onCreate$1(android.view.View p3) {
    this.openNote("Meeting Notes",
        "Review mobile app release checklist.\n" +
        "Confirm backup process.\n" +
        "Do not store secrets in client-side code.");
}

private synthetic void lambda$showLockedNoteDialog$3(
        android.content.DialogInterface p3, int p4) {
    android.widget.Toast.makeText(this, "Access denied", 0).show();
}

private void showLockedNoteDialog() {
    android.widget.EditText v0_1 = new android.widget.EditText(this);
    v0_1.setHint("Password");
    new android.app.AlertDialog$Builder(this)
        .setTitle("Authentication Required")
        .setMessage("This note is protected. Enter the password to continue.")
        .setView(v0_1)
        .setPositiveButton("Unlock",
            new MainActivity$$ExternalSyntheticLambda0(this))   // -> lambda$showLockedNoteDialog$3
        .setNegativeButton("Cancel", 0)
        .show();
}
```

Two things stand out:

1. **The password dialog is a red herring.** The Unlock handler never reads the `EditText`. It always shows "Access denied", whatever you type. The app has no password check at all.
2. **`getSecureBackupText()` is dead code.** Nothing calls it. It just base64-decodes a hardcoded string, like debug code someone forgot to remove.

The "Meeting Notes" note even says it: "Do not store secrets in client-side code." `NoteDetailActivity` only displays the title and body passed to it, and the locked-note flow never reaches it.

### 4. Getting the flag

No password or dynamic analysis needed. Decode the string:

```python
import base64
print(base64.b64decode("ZGVhZGZhY2V7ZXotbW9iaWxlLXRvLXN0YXJ0fQ==").decode())
# deadface{ez-mobile-to-start}
```

| Step | Technique |
|---|---|
| Unpack | `unzip` the zip, then the APK |
| Read manifest | `androguard` (AXML decoding) |
| Decompile | `AnalyzeAPK` → `get_vm_class().get_source()` |
| Spot the bug | Unlock never reads input; an unused method decodes a hardcoded string |
| Recover flag | Base64-decode the literal in `getSecureBackupText()` |

## Flag

!!! flag "Flag"
    `deadface{ez-mobile-to-start}`

## Takeaways

When an app has only a couple of classes, decompile everything before poking at the UI. Look for methods nothing calls: leftover "backup" or debug code often holds the secret.
