# Diagnosis

> **Category:** Mobile App · Android · **Points:** 100 · **Author:** @syyntax · **Track:** [Mobile](README.md)

## What We Were Given

A ZIP file containing an Android application package (`app-release.apk`) for "De Monne Financial", a fictional banking app. The story: DEADFACE managed to access customer data that normal users should never see, and De Monne's own server logs were turned off, so nobody knows exactly how the attackers got in. The task is to figure out whether DEADFACE compromised a developer account, or whether something was simply hardcoded into the app that gave them easy access.

## Step-by-Step Walkthrough

### 1. Unpack the APK

An Android APK file is really just a ZIP archive. Unzipping it exposes the app's compiled Java/Kotlin bytecode (in one or more `classes.dex` files), its resources, and its manifest:

```bash
$ unzip app-release.apk -d extracted/
$ ls extracted/
classes.dex classes2.dex AndroidManifest.xml resources.arsc ...
```

### 2. Build a minimal DEX parser

Normally, Android apps are analyzed with a decompiler like `jadx` or `apktool`, which turn the raw bytecode back into readable Java-like source code. Since those tools were not available in this environment, a small Python script was written that understands the raw **DEX file format** well enough to: read the list of all text strings embedded in the app, list every class, method and field name, and disassemble individual method bodies instruction by instruction, looking specifically for the `const-string` instruction (which loads a literal piece of text), to see exactly which hardcoded strings are used by which piece of code.

> **Why this works:** a compiled Android app still contains the full list of all the text strings it might ever use, in a shared "string table". By reading that table and then checking which strings each function loads, a function's behaviour can be reconstructed without a full decompiler, much like reading someone's shopping list to guess what they are cooking.

### 3. Search for hardcoded secrets

Searching the extracted string table for security-sensitive keywords (`token`, `password`, `secret`, `admin`, `internal`...) immediately surfaced a very suspicious field name: `INTERNAL_API_TOKEN`, used inside a class called `InternalTokenInterceptor`. This class adds an HTTP header called `X-Internal-Token` to every network request the app makes.

The class that constructs this interceptor (`ApiClient.getInstance()`) passes the token value in directly as a literal string, with no decryption step:

```java
new InternalTokenInterceptor(
    "zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR"
)
```

This is a real, usable secret API token, hardcoded directly into the app and shipped to every user who installs it, exactly the kind of mistake the challenge was hinting at.

### 4. Find a second hardcoded secret: a backdoor account

Continuing the same search through `LoginActivity.attemptLogin()` revealed a second hardcoded secret: a full email/password pair, apparently left behind for internal testing or auditing:

```
email:    audit@demonne-financial.local
password: demonnefinancial
```

This is a backdoor account baked directly into the compiled app, with a simple, guessable password, a textbook way for an attacker to get in without needing to "hack" anything technical at all.

### 5. Find the hidden diagnostic screen

Both secrets above are genuine vulnerabilities, but neither directly contains this challenge's flag. Continuing to search the app's classes for anything unusual found a class called `DiagnosticActivity`, tucked away inside an internal package, the kind of screen developers build for debugging purposes and normally remove before shipping to the public, but which was accidentally left inside this build.

Disassembling this activity's code and reading the literal strings it loads revealed that it builds and displays a debug report directly on-screen, including build version, environment, device info, and this exact block of text:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
diagnostic.code : deadface{flaG_ManIfesT3d}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
status           : internal build — not for distribution
```

The flag was simply hardcoded as plain text inside this forgotten debug screen, clearly never meant to reach a production release.

| Step | Technique |
|---|---|
| Unpack the app | `unzip` on the APK file |
| Read bytecode without a decompiler | Custom Python DEX file parser |
| Find hardcoded secrets | Searching the DEX string table for sensitive keywords |
| Confirm a hardcoded API token | Disassembly of `ApiClient.getInstance()` |
| Confirm a hardcoded backdoor account | Disassembly of `LoginActivity.attemptLogin()` |
| Reveal the flag | Disassembly of a leftover debug screen, `DiagnosticActivity` |

The same APK and backdoor account are reused directly against the live backend in [Inside Man](inside-man.md).

## Flag

```
deadface{flaG_ManIfesT3d}
```

---
[← Back to index](../../README.md)
