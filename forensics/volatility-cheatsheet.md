# Volatility 3 Cheatsheet

Memory forensics with Volatility 3. Always record the image hash and the tool version.

## 1. Identify the image

```bash
vol -f memory.raw windows.info
vol -f memory.raw linux.pslist
vol -f memory.raw mac.pslist
```

If the OS is unknown, start with `banners`:

```bash
vol -f memory.raw banners
```

## 2. Windows: processes

```bash
vol -f memory.raw windows.pslist
vol -f memory.raw windows.pstree
vol -f memory.raw windows.cmdline
vol -f memory.raw windows.getsids
vol -f memory.raw windows.privs
vol -f memory.raw windows.envars
```

Look for:

- processes with no parent or a suspicious parent
- `powershell`, `cmd`, `wscript`, `mshta`, `rundll32` with encoded or unusual arguments
- processes running from `%TEMP%`, `%APPDATA%`, or public directories
- mismatched process names and paths

## 3. Windows: injection and malware

```bash
vol -f memory.raw windows.malfind
vol -f memory.raw windows.vadinfo --pid <pid>
vol -f memory.raw windows.dlllist --pid <pid>
vol -f memory.raw windows.handles --pid <pid>
```

`malfind` highlights executable memory with no backing file. Confirm before calling it
malicious.

## 4. Windows: network

```bash
vol -f memory.raw windows.netscan
vol -f memory.raw windows.netstat
```

Map remote IPs and ports to the process. Check for beaconing intervals and uncommon ports.

## 5. Windows: persistence

```bash
vol -f memory.raw windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"
vol -f memory.raw windows.svcscan
vol -f memory.raw windows.scheduled_tasks
vol -f memory.raw windows.registry.userassist
```

Also check:

- `Startup` folders
- WMI event consumers
- services with unusual binary paths
- scheduled tasks running as SYSTEM

## 6. Windows: credentials

```bash
vol -f memory.raw windows.hashdump
vol -f memory.raw windows.lsadump
vol -f memory.raw windows.cachedump
```

Only use these on images you are authorized to examine. Treat recovered secrets as sensitive.

## 7. Windows: files and registry

```bash
vol -f memory.raw windows.filescan
vol -f memory.raw windows.dumpfiles --virtaddr <addr>
vol -f memory.raw windows.hivelist
vol -f memory.raw windows.registry.hivelist
```

Dump a file by virtual address from `filescan`. Verify the file type with `file` and a hash.

## 8. Linux

```bash
vol -f memory.raw linux.pslist
vol -f memory.raw linux.pstree
vol -f memory.raw linux.bash
vol -f memory.raw linux.proc.Maps
vol -f memory.raw linux.malfind
vol -f memory.raw linux.lsmod
vol -f memory.raw linux.sockstat
```

`linux.bash` reconstructs shell history from memory, which is often the fastest timeline.

## 9. macOS

```bash
vol -f memory.raw mac.pslist
vol -f memory.raw mac.pstree
vol -f memory.raw mac.netstat
vol -f memory.raw mac.lsmod
vol -f memory.raw mac.malfind
```

## 10. Timeline workflow

1. Hash the image: `sha256sum memory.raw`.
2. Identify the OS and kernel.
3. Build a process tree and command-line list.
4. Pivot on suspicious PIDs: DLLs, handles, network, memory.
5. Check persistence locations.
6. Extract files and compute hashes.
7. Write a timeline with UTC timestamps.
8. Correlate with disk, network, and log evidence.

## 11. Output and reproducibility

```bash
vol -f memory.raw windows.pslist > pslist.txt
vol -f memory.raw windows.pstree > pstree.txt
vol -f memory.raw windows.netscan > netscan.txt
vol -f memory.raw windows.malfind > malfind.txt
```

Keep the raw output, the exact command, the Volatility version, and the image hash. A finding
without reproducible commands is hard to defend.

## 12. Common pitfalls

- Wrong profile or symbol table: start with `windows.info` / `banners`.
- Huge images: use `--pid` and `--virtaddr` filters.
- `malfind` false positives: JIT engines and packed software look suspicious.
- Timestamps: normalize to UTC and state the timezone.
- Memory is volatile: hash and copy the image before analysis.
