# X Marks the Spot

> **Category:** Crypto · **Points:** 30 · **Track:** [Cryptography](README.md)

## Scenario

A network sensor picked up an outbound packet from a host registered to "Spectre Technology". The payload is hex-encoded, and it is hiding something `spookyboi` wants kept secret.

## Step 1: Undo the hex encoding

"Hex-encoded" just means every byte of the real data is written out as two hexadecimal characters (e.g. the byte `0x41` is written as the text `41`). This is trivial to reverse in Python with `bytes.fromhex()`, but the result was not readable text, just seemingly random bytes, which indicated the underlying data was also encrypted somehow.

## Step 2: Guess the cipher, single-byte XOR

The challenge only says "decrypt it" with no further hints, and the data is fairly short. A very common lightweight "encryption" in CTF challenges (and, historically, in real malware) is **single-byte XOR**: every byte of the plaintext is XOR-ed with the *same* one byte, repeated. This has exactly 256 possible keys (0–255), making it trivial to brute-force by trying every possible key and checking if any of the 256 results look like readable English containing something like "deadface{".

```python
h = '02362b2964302c2164372c25202b3337'  # (full hex string)
data = bytes.fromhex(h)
for k in range(256):
    out = bytes(b ^ k for b in data)
    if b'deadface' in out.lower():
        print(k, out)
```

## Step 3: Found it

Key `0x68` produced fully readable text:

```
From the shadows of Spectre Technology: deadface{xor_1_byt3_4t_4_t1m3}
```

> **Beginner tip:** brute-forcing single-byte XOR is one of the very first techniques worth trying whenever short, hex-encoded or Base64-encoded gibberish shows up with no other obvious structure; it costs almost nothing to try (only 256 possibilities) and is correct surprisingly often in beginner/intermediate crypto challenges.

## Flag

```
deadface{xor_1_byt3_4t_4_t1m3}
```

---
[← Back to index](../../README.md)
