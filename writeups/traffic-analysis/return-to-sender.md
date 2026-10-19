# Return to Sender

> **Category:** Traffic Analysis / PCAP · **Points:** 80 · **Track:** [Traffic Analysis](README.md)

## Scenario

A compromised host is shipping stolen data out to a server disguised as ordinary web traffic. A packet capture is given, and the task is to find and reassemble the smuggled data.

## Step 1: Spot the odd connection

Just like in [Bad Guys Be Bad](bad-guys-be-bad.md), the capture was first summarised by which IP addresses talked to each other and how much. Once again, most traffic was harmless noise to the `93.184.x.x` decoy range, while one IP, `45.66.77.88`, had a real, repeated exchange (5 packets each direction), clearly standing out from the background noise.

## Step 2: Reassemble and read the TCP stream

The full TCP conversation with that IP turned out to be five separate HTTP `POST` requests to what looked like a telemetry endpoint (`telemetry.deadface-cc.net`), each one numbered (`seq=1` through `seq=5`) and each carrying a small Base64-encoded chunk in its body, plus, in every request, an identical custom header: `X-Session-Id`, also Base64-encoded.

This is the same "split the secret into small innocuous-looking pieces" exfiltration pattern seen before, but this time disguised as normal application telemetry (something security tools often ignore because it is so common in legitimate apps).

## Step 3: Reassemble all the pieces

The five body chunks were concatenated in sequence order and Base64-decoded:

```python
import base64

parts = ['ZGVhZGZh', 'Y2V7aHR0',
         'cF9leGZp', 'bF9zbmVh', 'a3lf']
print(base64.b64decode(''.join(parts)))
```

```
b'deadface{http_exfil_sneaky_'
```

The string was clearly cut off mid-word. The missing ending turned out to be hiding in that repeated `X-Session-Id` header noticed in every request:

```python
print(base64.b64decode('YnVzaW5lc3N9'))
# -> b'business}'
```

Putting the two pieces together, the body chunks and the header, completes the flag:

```
deadface{http_exfil_sneaky_} + business} -> deadface{http_exfil_sneaky_business}
```

> **Beginner tip:** when reassembling split secrets from network traffic, always check every part of the request, headers, cookies and URL parameters, not just the body. Challenge authors like to hide the last piece somewhere a rushed analyst wouldn't think to look.

## Flag

```
deadface{http_exfil_sneaky_business}
```

---
[← Back to index](../../README.md)
