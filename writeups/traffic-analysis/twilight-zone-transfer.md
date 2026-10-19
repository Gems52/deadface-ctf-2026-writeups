# Twilight Zone Transfer

> **Category:** Traffic Analysis / PCAP · **Points:** 50 · **Track:** [Traffic Analysis](README.md)

## Scenario

A misconfigured DNS server allowed an unauthorized **AXFR zone transfer** (a bulk export of every DNS record for an entire domain). The attacker pulled the whole zone in one request, and the answer is hidden somewhere in those records.

> **What is an AXFR zone transfer?** Normally, DNS servers answer one question at a time ("what's the IP for www.example.com?"). AXFR is a special bulk request that says "give me every record you know about this entire domain at once". This is meant to be used only between a domain's own primary and secondary name servers; if a DNS server allows anyone to request it, that is a serious misconfiguration, because it leaks the server's entire internal map of hostnames.

## Step 1: Find and reassemble the AXFR transfer in the capture

Unlike normal DNS lookups (which use UDP), zone transfers use **TCP port 53**, because the response is often too large to fit in a single UDP packet. Filtering the capture for TCP traffic on port 53 found exactly one such stream, reassembled in TCP sequence order to get the full, raw DNS message bytes.

## Step 2: Parse the DNS message by hand

DNS messages have a well-documented binary format (header, then a list of "answer" records, each with a name, type, TTL, and the actual data). A small Python DNS parser was written to walk through every answer record in the transferred zone. It listed 14 records in total: the usual nameserver (`NS`) and address (`A`) records for hosts like `www`, `mail`, `vpn`, `git`, `admin`, and so on, plus two `TXT` records.

## Step 3: Spot the flag record

One of the two `TXT` records was a normal SPF record (used for email anti-spoofing). The other one stood out immediately:

```
_flag.deadface-cc.net   TXT   -> "deadface{dns_zone_transfer}"
```

The entire reassembled stream was also scanned for printable text to check there wasn't anything else hidden anywhere in the raw bytes of the transfer (IP addresses, record ordering, any stray text); nothing else of interest turned up, confirming this `TXT` record was the intended flag.

## Flag

```
deadface{dns_zone_transfer}
```

---
[← Back to index](../../README.md)
