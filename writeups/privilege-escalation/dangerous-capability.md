# Dangerous Capability

> **Category:** Forensics / Priv Esc / Linux · **Points:** 300 · **Author:** @syyntax · **Track:** [Linux Privilege Escalation](README.md)

## Scenario

Another privilege-escalation step on the same `charon` box. A "crypto tooling" binary left lying around was granted a Linux capability it should never have had. The task is to abuse that capability to become the next user in the chain and read their flag.

> **Note from the challenge:** `charon` is disposable, every web-shell session is a fresh copy of the box, so each flag must be grabbed as it is found.

> **What is a Linux capability?** Traditionally, Linux split the world into "root" (which can do anything) and "everyone else". Capabilities break root's power down into small, individually grantable pieces, for example `cap_net_bind_service` lets a program bind to a low-numbered network port without being fully root. `cap_setuid` specifically lets a program change its own user ID to *any* user, effectively a scoped-down version of root that only applies to the ability to "become someone else".

## Step 1: Find the binary with the dangerous capability

```bash
getcap -r / 2>/dev/null
# /opt/crypto/cryptctl cap_setuid=ep
```

This single binary is explicitly allowed to call the `setuid()` system call to become a different user, something normal programs cannot do.

## Step 2: Try the binary, and hit a dead end with guessing

Running the binary just printed:

```
[cryptctl] assuming the crypto service identity.
```

Many reasonable guesses were tried to influence which identity it assumes: environment variables like `USER`, `LOGNAME`, `SUDO_USER`, command-line arguments like `--user root`, renaming the binary (some programs behave differently depending on the name they were invoked as, like BusyBox). None of it changed the output. At this point, guessing blindly stops being efficient.

## Step 3: Trace its actual system calls

Instead of continuing to guess, the next step was to watch exactly what the binary does at the syscall level. Since `strace` was not installed on the box, a very small syscall tracer was written in Python using the `ptrace()` system call directly via `ctypes` (the same underlying mechanism `strace` itself is built on):

```bash
python3 /tmp/minitrace.py /opt/crypto/cryptctl
```

The output showed, immediately and unambiguously, the exact system call the binary makes:

```
>>> setuid(rdi=0x3ee, rsi=0x0, rdx=0x0)
<<< setuid returned 0
```

> **Beginner tip:** `0x3ee` is hexadecimal. Converting to decimal: `0x3ee = 1006`. Checking `/etc/passwd`, UID 1006 belongs to the user `spookyboi`. So the binary is hard-coded to always switch to the `spookyboi` identity, no argument or environment variable was ever going to change that, exactly why the earlier guesses all failed the same way.

## Step 4: Use the capability and read the flag

Because the binary was about to switch the shell to the `spookyboi` identity, the next step was simply to run it and then read that user's flag file directly:

```bash
/opt/crypto/cryptctl
cat /home/spookyboi/flag.txt
```

```
deadface{c4p_s3tu1d_cuts_d33p}
```

## Flag

```
deadface{c4p_s3tu1d_cuts_d33p}
```

---
[← Back to index](../../README.md)
