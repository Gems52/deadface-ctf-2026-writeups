# Locked... or Not?

> **Category:** Mobile · **Points:** 150 · **Author:** @G2Gh0st · **Track:** [Mobile](README.md)

## The Scenario

A junior DEADFACE developer built an Android app ("df-securenotes") to store a secure note. The note is protected by a password nobody can guess. Rather than trying to brute-force the password, the challenge asks to "tear the app apart", i.e. reverse-engineer it.

## Step 1: An APK is just a ZIP file

Android application packages (`.apk`) are regular ZIP archives. Unzipping one gives the app's manifest, resources, and, most importantly, its compiled code in one or more `classes.dex` files (Dalvik EXecutable, Android's bytecode format).

```bash
$ sha256sum df-securenotes_apk.zip
b5f6e9a0a75f404103fc2d08542165015f8934e8d76433a513ca1a4705f006ca

$ unzip df-securenotes_apk.zip
$ unzip -o df-securenotes.apk -d x
$ ls x
AndroidManifest.xml classes.dex classes2.dex resources.arsc ...
```

## Step 2: Look for secrets before looking for the password check

Rather than disassembling the whole app, a fast first move on any mobile challenge is to run `strings` on every DEX file. Hard-coded secrets, debug leftovers, and API keys are extremely common mistakes in real-world apps, and this challenge's title, "Locked... or Not?", is a strong hint that the lock is not as solid as it looks.

```bash
$ strings classes2.dex | grep -E '^[A-Za-z0-9+/=]{16,}$'
ZGVhZGZhY2V7ZXotbW9iaWxlLXRvLXN0YXJ0fQ==
```

This string sits next to method and field names like `getSecureBackupText` and `apiKeyBackup`, which are then passed into `android.util.Base64.decode()`. In other words: the developer added a "backup" code path that keeps the real secret sitting in plain Base64 inside the app, completely bypassing the password check.

## Step 3: Decode it

Base64 is not encryption, just an encoding, decoding it requires no key at all:

```bash
$ echo 'ZGVhZGZhY2V7ZXotbW9iaWxlLXRvLXN0YXJ0fQ==' | base64 -d
deadface{ez-mobile-to-start}
```

## Flag

```
deadface{ez-mobile-to-start}
```

> **Lesson:** never trust client-side "secure" storage; if the secret has to be readable by the app, it is readable by an attacker too.

---
[← Back to index](../../README.md)
