<div align="center">

# 💀 DEADFACE CTF 2026: Writeups

**26 challenges solved · 9 categories · 4045 points**

![CTF](https://img.shields.io/badge/CTF-DEADFACE%202026-0b0b0f?style=for-the-badge&labelColor=0b0b0f&color=ec4899)
![Team](https://img.shields.io/badge/Team-RUY-0b0b0f?style=for-the-badge&labelColor=0b0b0f&color=a78bfa)
![Solved](https://img.shields.io/badge/Solved-26-0b0b0f?style=for-the-badge&labelColor=0b0b0f&color=22d3ee)
![Language](https://img.shields.io/badge/Writeups-English-0b0b0f?style=for-the-badge&labelColor=0b0b0f&color=fbbf24)

SQL · Linux Privilege Escalation · Pwn · Traffic Analysis · OSINT · Mobile · Crypto · Reverse Engineering · Forensics

</div>

---

## About

Writeups for the challenges solved by **GEMS52** of team **RUY** at **DEADFACE CTF 2026**, which ran from 2026-10-17 14:00 UTC to 2026-10-19 00:00 UTC, roughly 34 hours of competition.

The challenges span a hospital data breach, a clinical-trial fraud investigation, a multi-stage Linux privilege-escalation chain on a box named `charon`, a set of pwn binaries from the same "bulletproof hosting" author, DNS/ICMP exfiltration captures, two Android apps, and a handful of standalone crypto, reverse-engineering and forensics puzzles. Every writeup documents the path that actually led to the flag: the artifact, the key insight, and the decisive command or script. Dead ends are not reproduced.

## Scoreboard at a glance

Challenges are grouped by category (highest points first), matching the order they were solved and documented. Click a challenge name to open its writeup.

### 🗄️ Database / SQL — 1285 pts

*Cross-database joins and anomaly hunting in clinical-trial, hospital and audit data.* ([track overview](writeups/sql/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Two Places at Once](writeups/sql/two-places-at-once.md) | 450 | `deadface{kbhatt}` |
| 2 | [Too Good to be True](writeups/sql/too-good-to-be-true.md) | 310 | `deadface{VELT-0148}` |
| 3 | [Ghost in the Registry](writeups/sql/ghost-in-the-registry.md) | 240 | `deadface{AH-0090001}` |
| 4 | [After the Lock](writeups/sql/after-the-lock.md) | 150 | `deadface{VELT-0137}` |
| 5 | [Upon the Throne](writeups/sql/upon-the-throne.md) | 120 | `deadface{REVENANT-7}` |
| 6 | [Admitted Under Another Name](writeups/sql/admitted-under-another-name.md) | 15 | `deadface{AH-0044719}` |

### 🔓 Linux Privilege Escalation — 800 pts

*PATH hijacking and Linux-capability abuse on the shared box `charon`.* ([track overview](writeups/privilege-escalation/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Heavy is the Crown](writeups/privilege-escalation/heavy-is-the-crown.md) | 500 | `deadface{p4th_h1j4ck3d_t0_thr0n3}` |
| 2 | [Dangerous Capability](writeups/privilege-escalation/dangerous-capability.md) | 300 | `deadface{c4p_s3tu1d_cuts_d33p}` |

### 💣 Pwn (Binary Exploitation) — 610 pts

*Stack overflows and ROP chains, no canary, no PIE, NX bypassed.* ([track overview](writeups/pwn/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Cursed Library](writeups/pwn/cursed-library.md) | 235 | `deadface{T#3_L1br4ry_Kn0w5_A11}` |
| 2 | [Cursed Mirror](writeups/pwn/cursed-mirror.md) | 150 | `deadface{M1RR0R_R1M}` |
| 3 | [Skeleton Key](writeups/pwn/skeleton-key.md) | 125 | `deadface{R355UR3CT_D34D_C0D3}` |
| 4 | [The Crypt](writeups/pwn/the-crypt.md) | 100 | `deadface{d3crypt3d_1nt0_n0th1ng}` |

### 📡 Traffic Analysis (PCAP) — 505 pts

*DNS tunneling, HTTP smuggling and ICMP exfiltration hidden behind decoy traffic.* ([track overview](writeups/traffic-analysis/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Bad Guys Be Bad](writeups/traffic-analysis/bad-guys-be-bad.md) | 350 | `deadface{c2_traffic_be_bussing}` |
| 2 | [Return to Sender](writeups/traffic-analysis/return-to-sender.md) | 80 | `deadface{http_exfil_sneaky_business}` |
| 3 | [Twilight Zone Transfer](writeups/traffic-analysis/twilight-zone-transfer.md) | 50 | `deadface{dns_zone_transfer}` |
| 4 | [Data Go Brrrrr](writeups/traffic-analysis/data-go-brrrrr.md) | 25 | `deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}` |

### 🕵️ OSINT / Privilege Escalation — 350 pts

*A targeted wordlist built from a social-media post, chained into a privilege-escalation bug.* ([track overview](writeups/osint/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Old Habits](writeups/osint/old-habits.md) | 350 | `deadface{0s1nt_n3v3r_f0rg3ts}` |

### 📱 Mobile — 270 pts

*Two Android APKs reverse-engineered by hand, no decompiler.* ([track overview](writeups/mobile/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Locked... or Not?](writeups/mobile/locked-or-not.md) | 150 | `deadface{ez-mobile-to-start}` |
| 2 | [Diagnosis](writeups/mobile/diagnosis.md) | 100 | `deadface{flaG_ManIfesT3d}` |
| 3 | [Inside Man](writeups/mobile/inside-man.md) | 20 | `deadface{l3FT_b3Hind!}` |

### 🔐 Cryptography — 125 pts

*Nonce reuse, single-byte XOR brute force, and a Caesar cipher.* ([track overview](writeups/crypto/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Second Shift](writeups/crypto/second-shift.md) | 90 | `deadface{ctr_nonce_reuse_breaks_confidentiality}` |
| 2 | [X Marks the Spot](writeups/crypto/x-marks-the-spot.md) | 30 | `deadface{xor_1_byt3_4t_4_t1m3}` |
| 3 | [Hail Caesar](writeups/crypto/hail-caesar.md) | 5 | `deadface{Fetch}` |

### 🔧 Reverse Engineering — 75 pts

*A custom Base64 alphabet with an RC4 payload, and a Doom level that spells out its own flag.* ([track overview](writeups/reverse-engineering/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Cereal Killer 01](writeups/reverse-engineering/cereal-killer-01.md) | 50 | `deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}` |
| 2 | [DOOM (Bonus)](writeups/reverse-engineering/doom.md) | 25 | `deadface{Lets_Play_Doom}` |

### 🖼️ Forensics — 25 pts

*Spotting a forged painting through its PNG metadata.* ([track overview](writeups/forensics/README.md))

| # | Challenge | Pts | Flag |
|:-:|---|:-:|---|
| 1 | [Art Heist: The Forgery](writeups/forensics/art-heist-the-forgery.md) | 25 | `deadface{L4MIA_SIGNS_HER_WORK}` |

## Techniques covered

| Domain | Highlights |
|---|---|
| **SQL** | Cross-database joins, timestamp-based tamper detection, statistical outlier detection (3σ) |
| **Privilege escalation** | PATH hijacking against a SUID binary, `cap_setuid` capability abuse, a custom ptrace-based syscall tracer |
| **Pwn** | Stack buffer overflows, PIE/libc leaks, ROP chains, ret2libc, multi-stage overflow chaining |
| **Traffic analysis** | TCP stream reassembly, DNS tunneling (AXFR, Base32-in-subdomains), HTTP/ICMP exfiltration, a from-scratch pcap parser |
| **OSINT** | Social-media reconnaissance turned into a targeted wordlist, offline hash cracking (John the Ripper) |
| **Mobile** | A hand-written DEX parser, hardcoded secrets and backdoor accounts in a shipped APK, live API abuse |
| **Crypto** | Stream-cipher nonce reuse ("two-time pad"), single-byte XOR brute force, Caesar/ROT13 |
| **Reverse engineering** | Custom Base64 alphabets, RC4 payload decryption, binary file-format parsing (Doom WAD) |
| **Forensics** | PNG chunk and XMP metadata comparison across near-identical files |

## Repository layout

```
.
├── README.md
├── assets/                       # evidence images referenced by the writeups
└── writeups/
    ├── sql/                      # Database / SQL
    ├── privilege-escalation/     # Linux Privilege Escalation
    ├── pwn/                      # Binary Exploitation
    ├── traffic-analysis/         # PCAP / Network Forensics
    ├── osint/                    # OSINT / Privilege Escalation
    ├── mobile/                   # Android
    ├── crypto/                   # Cryptography
    ├── reverse-engineering/      # Reverse Engineering
    └── forensics/                # Forensics
```

## Credits

- **GEMS52**: author of the writeups, solver of the challenges listed here
- **Team RUY**
- Challenge authors credited on each writeup where known (@syyntax, @SpiffyLich, @G2Gh0st, @TheZealot)
- Platform: DEADFACE CTF 2026

## License

Released under the [MIT License](LICENSE). Challenge names, files and flags belong to their respective authors and organizers.
