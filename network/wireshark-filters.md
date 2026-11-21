# Wireshark Display Filters

Capture filters decide what is recorded. Display filters decide what you see. These are
display filters unless noted.

## 1. Addresses and ports

```text
ip.addr == 10.0.0.5
ip.src == 10.0.0.5
ip.dst == 10.0.0.5
ip.addr == 10.0.0.0/24
tcp.port == 443
udp.port == 53
tcp.srcport == 8080
!(arp or icmp or dns)
```

## 2. TCP flags and health

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0     # SYN
tcp.flags.reset == 1                          # RST
tcp.analysis.retransmission
tcp.analysis.zero_window
tcp.analysis.flags
tcp.stream eq 0
```

Follow a stream: right-click a packet → **Follow → TCP Stream**. The filter bar then shows
`tcp.stream eq N`.

## 3. HTTP

```text
http.request
http.response
http.request.method == "POST"
http.host contains "example.com"
http.request.uri contains "/admin"
http.response.code >= 400
http.user_agent contains "sqlmap"
http.cookie
http.authorization
http.content_type contains "json"
```

Find credentials in form posts:

```text
http.request.method == "POST" && http.file_data contains "password"
```

## 4. DNS

```text
dns
dns.qry.name contains "example.com"
dns.qry.type == 1          # A
dns.qry.type == 28         # AAAA
dns.qry.type == 16         # TXT
dns.flags.response == 0    # queries
dns.flags.response == 1    # responses
dns.flags.rcode != 0       # errors
```

Long or high-entropy subdomains are a common exfiltration signal:

```text
dns.qry.name matches "^[a-z0-9]{20,}\."
```

## 5. TLS

```text
tls
tls.handshake.type == 1                              # ClientHello
tls.handshake.extensions_server_name contains "example.com"
tls.alert_message
tls.record.version == 0x0303
```

Decrypt TLS with a key log file:

1. Set `SSLKEYLOGFILE=/tmp/keys.log` before launching the browser.
2. In Wireshark: **Preferences → Protocols → TLS → (Pre)-Master-Secret log filename**.
3. Filter for `http` again.

## 6. ARP, ICMP, DHCP

```text
arp
arp.opcode == 1          # request
arp.opcode == 2          # reply
arp.duplicate-address-detected
icmp
icmp.type == 8           # echo request
icmp.type == 0           # echo reply
dhcp
dhcp.option.dhcp == 3    # DHCP request
bootp.option.dhcp == 3   # older name
```

## 7. SMB, FTP, SSH

```text
smb2
smb2.cmd == 3            # tree connect
smb2.filename contains "secret"
ftp
ftp.request.command == "USER"
ftp.request.command == "PASS"
ssh
```

## 8. Files and objects

```text
frame contains "password"
frame contains "BEGIN RSA PRIVATE KEY"
tcp contains "flag{"
http.request.uri contains ".php"
```

Export HTTP objects: **File → Export Objects → HTTP**. Export SMB objects similarly.

## 9. Useful CLI equivalents

```bash
# HTTP hosts and URIs
tshark -r capture.pcap -Y 'http.request' -T fields -e http.host -e http.request.uri

# DNS queries
tshark -r capture.pcap -Y 'dns.flags.response == 0' -T fields -e dns.qry.name

# Unique TLS SNI values
tshark -r capture.pcap -Y 'tls.handshake.type == 1' -T fields -e tls.handshake.extensions_server_name | sort -u

# Conversations
tshark -r capture.pcap -q -z conv,tcp
```

## 10. Workflow

1. Start broad: `ip.addr == <target>`.
2. Narrow by protocol: `http`, `dns`, `tls`, `smb2`.
3. Follow the stream for context.
4. Extract objects and hashes.
5. Pivot on hostnames, user agents, and timing.
6. Save the filter and the packet numbers in your notes.
