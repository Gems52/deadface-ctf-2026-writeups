# Data Go Brrrrr

> **Category:** Traffic Analysis / PCAP · **Points:** 25 · **Track:** [Traffic Analysis](README.md)

## Scenario

A client's network is leaking data, narrowed down to a small slice of traffic. A packet capture (`.pcap`) file is given, and the task is to find the flag hidden in the exfiltrated traffic.

## Step 1: Parse the capture

A `.pcap` file is just a binary log of every network packet seen. Each packet has an Ethernet header, then an IP header, then (depending on the protocol) a TCP, UDP or ICMP header, followed by the actual payload. A small Python script was written to read the file byte by byte and pull out that structure, without needing Wireshark installed.

> **Beginner tip:** Wireshark is not strictly necessary to read a pcap; the file format is simple and documented, so a short Python script using the built-in `struct` module can parse it. This is handy when working inside a minimal sandbox/container.

## Step 2: Look at the raw payload data

Once the pcap was parsed, every non-empty packet payload was printed in order. The capture contained a series of small packets, each carrying a short chunk of text that looked like **Base64** (a text encoding made of letters, digits, `+`, `/` and `=` padding), for example chunks like `ZGVhZGZh`, `Y2V7aWNt`, `cF8zeGZp`, and so on, spread across several packets.

This is a classic **data exfiltration** pattern: instead of sending one big suspicious file, an attacker splits a secret into tiny pieces and smuggles them out one packet at a time (here, disguised inside ICMP traffic, the same protocol used by the harmless `ping` command), hoping it blends into normal background noise.

## Step 3: Reassemble and decode

The fix was simple: concatenate all the Base64 fragments in the order they appeared in the capture, then Base64-decode the whole thing in one go:

```python
import base64

parts = ['ZGVhZGZh', 'Y2V7aWNt', 'cF8zeGZp',
         'bF93aDNu', 'X3RoM3lf', 'bGVhc3Rf',
         'c3VzcDNj', 'dH0=']
joined = ''.join(parts)
print(base64.b64decode(joined))
```

```
b'deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}'
```

## Flag

```
deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}
```

---
[← Back to index](../../README.md)
