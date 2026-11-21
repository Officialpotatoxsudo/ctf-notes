# CTF Notes

Practical, copy-paste-friendly notes for CTFs, labs, and authorized security testing.

These are my personal study notes. They are intentionally short: enough to jog your memory
during a challenge, with links to deeper material where it helps.

> **Authorized testing only.** Every technique here is for systems you own or have explicit
> permission to test. Don't use these notes on systems you don't control.

## Index

### Linux
- [Privilege escalation checklist](linux/privilege-escalation-checklist.md)
- [SUID / SGID cheatsheet](linux/suid-sgid-cheatsheet.md)

### Web
- [SQL injection cheatsheet](web/sql-injection-cheatsheet.md)
- [XSS cheatsheet](web/xss-cheatsheet.md)
- [OWASP Top 10 quick reference](web/owasp-top-10-quick-reference.md)

### Network
- [Nmap cheatsheet](network/nmap-cheatsheet.md)
- [Wireshark display filters](network/wireshark-filters.md)

### Android
- [ADB cheatsheet](android/adb-cheatsheet.md)
- [Frida quickstart](android/frida-quickstart.md)

### Forensics
- [Volatility 3 cheatsheet](forensics/volatility-cheatsheet.md)

### Crypto
- [OpenSSL cheatsheet](crypto/openssl-cheatsheet.md)

### Tooling
- [Git recovery cheatsheet](git/git-recovery-cheatsheet.md)
- [Python venv / pip cheatsheet](python/venv-pip-cheatsheet.md)
- [Docker cheatsheet](docker/docker-cheatsheet.md)

## Conventions

- Commands assume a Linux/macOS shell unless stated otherwise.
- Replace `<target>`, `<ip>`, `<port>`, and `<file>` with your own values.
- Prefer `-oN`/`-oA` on scans so you keep evidence.
- When a note says "try", verify the result manually. Context matters more than payload lists.

## License

MIT. See [LICENSE](LICENSE).
