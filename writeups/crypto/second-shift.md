# Second Shift

> **Category:** Crypto · **Points:** 90 · **Author:** @syyntax · **Track:** [Cryptography](README.md)

## Scenario

Files were recovered from a DEADFACE relay server. One "heartbeat" message was captured in **both** its original (plaintext) form and its encrypted ("processed") form. A second message, a "tasking" instruction, was only recovered in encrypted form. The task is to recover its contents.

## Step 1: Inspect what was given

The ZIP contained three files:

- `relay-heartbeat.json`: the **plaintext** (unencrypted) version of a routine message.
- `relay-heartbeat.enc`: the **same message**, but after the relay encrypted it.
- `relay-tasking.enc`: a **second, different** message, only available encrypted.

> **Beginner tip:** whenever both the plaintext *and* the ciphertext of the same message are available, think "known-plaintext attack". If the encryption scheme reuses the same key stream for more than one message (a mistake called **nonce/key reuse**, common in stream ciphers like RC4 or AES-CTR when implemented carelessly), that key stream can be recovered and used to decrypt anything else encrypted with the same stream; this is sometimes called a "two-time pad" attack.

## Step 2: Recover the key stream

A stream cipher encrypts by XOR-ing the plaintext with a pseudo-random key stream, byte by byte: `ciphertext = plaintext XOR keystream`. This also means: `keystream = ciphertext XOR plaintext`. Since both the plaintext and the ciphertext for the heartbeat message are available, the exact key stream used can be recovered:

```python
data_enc = open('relay-heartbeat.enc', 'rb').read()
data_json = open('relay-heartbeat.json', 'rb').read()[:-1]  # drop trailing newline
keystream = bytes(a ^ b for a, b in zip(data_enc, data_json))
print(len(keystream), 'keystream bytes recovered')
```

## Step 3: Reuse the key stream on the second message

If the relay made the classic mistake of reusing the same key stream for every message it sent (instead of generating a fresh one each time), then XOR-ing the tasking ciphertext with this same recovered key stream should reveal its plaintext too:

```python
data_enc = open('relay-heartbeat.enc', 'rb').read()
data_json = open('relay-heartbeat.json', 'rb').read()[:-1]
keystream = bytes(a ^ b for a, b in zip(data_enc, data_json))

tasking = open('relay-tasking.enc', 'rb').read()
plaintext = bytes(a ^ b for a, b in zip(tasking, keystream))
print(plaintext)
```

This immediately printed a readable JSON object:

```json
{"type":"tasking","target":"aurora-vpn","action":"stage-access",
 "window":"00:30Z","operator":"mort1cia",
 "note":"Use the backup tunnel. Archive bundle under
 DEADFACE{ctr_nonce_reuse_breaks_confidentiality} before exfil."}
```

The attack worked on the first try, confirming the relay really did reuse its key stream between messages, exactly the vulnerability the challenge title ("Second Shift", i.e. a second message reusing the same cryptographic "shift") was hinting at.

## Flag

```
deadface{ctr_nonce_reuse_breaks_confidentiality}
```

---
[← Back to index](../../README.md)
