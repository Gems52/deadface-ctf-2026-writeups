# Inside Man

> **Category:** Mobile App · Web / API · **Points:** 20 · **Author:** @syyntax · **Track:** [Mobile](README.md)

## What We Were Given

This challenge reuses the exact same APK and the exact same background story as [Diagnosis](diagnosis.md): De Monne Financial suspects DEADFACE either compromised a developer account, or found something hardcoded in the app that gave them easy access. Where "Diagnosis" was solved purely through static analysis of the app file, "Inside Man" required actually **using** one of the secrets found to talk to the real backend server.

## Step-by-Step Walkthrough

### 1. Reuse the backdoor account discovered earlier

During the "Diagnosis" challenge, a hardcoded backdoor account baked into the app's login screen was already found:

```
email:    audit@demonne-financial.local
password: demonnefinancial
```

The idea for this challenge was to actually log in as this account against the app's real backend API, to prove that this backdoor genuinely grants unauthorized access, exactly how DEADFACE would have used it.

### 2. Call the real login endpoint

By reading the app's networking code, the base API address the app talks to (`http://dmf.dctf.io:8081/`) was already identified, along with the exact login endpoint used by the mobile app (`/api/auth/login`). The exact request the app itself would send was replicated using `curl`:

```bash
$ curl -s -i -X POST \
  -H "Content-Type: application/json" \
  -d '{"email":"audit@demonne-financial.local","password":"demonnefinancial"}' \
  http://dmf.dctf.io:8081/api/auth/login
```

### 3. Read the flag directly from the server's response

The server replied with `HTTP/1.1 200 OK`, confirming the backdoor account is active and accepted, along with a normal login session token (a JWT) and a small JSON object describing the logged-in user. That user object contained an extra field that is clearly not meant to be there in a real production system:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "email": "audit@demonne-financial.local",
    "id": 5,
    "message": "deadface{l3FT_b3Hind!}",
    "name": "Audit System"
  }
}
```

The server itself hands back the flag directly inside the `message` field the moment this backdoor account logs in, proof that this leftover "audit" account is exactly the undocumented door DEADFACE walked through (left behind by the developers, hence the flag text itself).

> **Why this is a serious vulnerability:** a hardcoded account baked directly into a shipped mobile app means anyone who downloads the app can extract these credentials (as done here, through simple static analysis) and log in with full account access, without needing to guess or steal anything from a real customer or developer.

| Step | Technique |
|---|---|
| Reuse the backdoor account | Credentials found during static analysis in "Diagnosis" |
| Identify the real API endpoint | Reading the app's networking code (Retrofit interface) |
| Authenticate as the backdoor account | `curl` POST request to `/api/auth/login` |
| Reveal the flag | Flag returned directly in the server's JSON login response |

## Flag

```
deadface{l3FT_b3Hind!}
```

---
[← Back to index](../../README.md)
