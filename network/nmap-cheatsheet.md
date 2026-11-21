# Nmap Cheatsheet

Common scans for authorized engagements. Always confirm scope and rate limits first.

## 1. Host discovery

```bash
nmap -sn 10.0.0.0/24
nmap -sn -PE -PP -PS80,443 10.0.0.0/24
nmap -Pn 10.0.0.10
```

- `-sn` — ping scan only (no ports)
- `-Pn` — treat the host as up (skip discovery)
- `-PE` / `-PP` — ICMP echo / timestamp
- `-PS` — TCP SYN ping

## 2. Top ports

```bash
nmap -sS -sV -sC --top-ports 1000 -oA scans/top1000 <target>
nmap -sS -p- --min-rate 2000 -oA scans/allports <target>
```

- `-sS` — SYN scan (needs root)
- `-sT` — connect scan (no root)
- `-sV` — service/version detection
- `-sC` — default NSE scripts
- `-p-` — all 65,535 ports
- `-oA` — save normal, XML, and grepable output

## 3. UDP

```bash
nmap -sU --top-ports 100 -sV <target>
nmap -sU -p 53,67,68,69,123,161,162,500 <target>
```

UDP scans are slow. Start with the common services: DNS, DHCP, TFTP, NTP, SNMP, IKE.

## 4. Version and OS detection

```bash
nmap -sV --version-all -p 22,80,443 <target>
nmap -O --osscan-guess <target>
```

`-O` needs at least one open and one closed port. Results are hints, not facts.

## 5. NSE scripts

```bash
nmap --script vuln -oA scans/vuln <target>
nmap --script "http-title,http-headers,http-methods" -p 80,443 <target>
nmap --script smb-enum-shares,smb-os-discovery -p 445 <target>
nmap --script ssl-enum-ciphers -p 443 <target>
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=example.com -p 53 <target>
```

Useful script categories: `auth`, `broadcast`, `default`, `discovery`, `exploit`, `safe`,
`vuln`.

## 6. Timing and performance

```bash
nmap -T4 --min-rate 1000 --max-retries 2 <target>
nmap -T2 --scan-delay 1s <target>
```

- `-T0` / `-T1` — IDS evasion, very slow
- `-T3` — default
- `-T4` — fast, stable networks
- `-T5` — aggressive, can miss results

## 7. Firewall and IDS evasion

```bash
nmap -f <target>                 # fragment packets
nmap -D RND:10 <target>          # decoy scan
nmap --source-port 53 <target>   # spoof source port
nmap --data-length 50 <target>   # pad packets
nmap --spoof-mac 0 <target>      # random MAC
```

Only use evasion techniques when the rules of engagement allow it.

## 8. Output that survives the engagement

```bash
nmap -sS -sV -sC -p- -oA scans/full <target>
```

`-oA` writes `.nmap`, `.xml`, and `.gnmap`. Keep the XML: it imports cleanly into tooling.

Convert XML to HTML:

```bash
xsltproc scans/full.xml -o scans/full.html
```

## 9. Quick reference

| Goal | Command |
|---|---|
| Is the host up? | `nmap -sn <target>` |
| Fast top-1000 scan | `nmap -sS -sV -sC --top-ports 1000 <target>` |
| All TCP ports | `nmap -sS -p- --min-rate 2000 <target>` |
| UDP top 100 | `nmap -sU --top-ports 100 <target>` |
| Vulnerabilities | `nmap --script vuln <target>` |
| SMB shares | `nmap --script smb-enum-shares -p 445 <target>` |
| TLS ciphers | `nmap --script ssl-enum-ciphers -p 443 <target>` |
| Save everything | `nmap -sS -sV -sC -p- -oA scans/full <target>` |

## 10. After the scan

- verify open ports manually with `nc`, `curl`, or the service client
- check for filtered ports that may open with a different source port
- compare TCP and UDP results
- review service versions against known CVEs
- record the exact command and timestamp for the report
