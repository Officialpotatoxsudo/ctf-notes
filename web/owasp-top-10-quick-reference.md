# OWASP Top 10 Quick Reference

Based on the OWASP Top 10 2021, which is still the stable edition used by most programs.
Check the OWASP site for the latest release before quoting a category in a report.

## A01 — Broken Access Control

Users can act outside their intended permissions.

Test for:

- IDOR: change an object ID and see if another user's data is returned
- forced browsing to admin routes
- missing function-level checks on API endpoints
- horizontal and vertical privilege escalation
- permissive CORS with credentials
- path traversal in file endpoints

Quick check:

```http
GET /api/users/1234/invoices/9001
Cookie: session=<low-priv user>
```

If the response belongs to another user, that is a finding.

## A02 — Cryptographic Failures

Sensitive data is exposed because cryptography is missing or weak.

Test for:

- HTTP instead of HTTPS, or mixed content
- weak TLS versions and cipher suites
- hard-coded keys and secrets in source or JS bundles
- unsalted or fast password hashes
- predictable tokens and IVs
- missing encryption at rest for sensitive fields

Quick check:

```bash
openssl s_client -connect target:443 -servername target </dev/null 2>/dev/null | grep -E 'Protocol|Cipher'
```

## A03 — Injection

Untrusted data is interpreted as code or a query.

Test for:

- SQL injection
- NoSQL injection
- OS command injection
- LDAP injection
- template injection (SSTI)
- XPath / expression language injection
- header injection

Quick check: send a quote and a comment marker, then compare responses and timing.

## A04 — Insecure Design

The flaw is in the workflow, not the parser.

Test for:

- missing rate limits on OTP, login, and password reset
- business-logic bypass (negative quantities, coupon stacking)
- trust in client-side price or role fields
- race conditions in one-time operations
- missing approval steps in multi-step workflows

Quick check: replay a one-time request concurrently and see if it succeeds twice.

## A05 — Security Misconfiguration

The default or deployed configuration is unsafe.

Test for:

- default credentials
- verbose errors and stack traces
- directory listing
- unnecessary features, ports, and services
- missing security headers
- permissive cloud storage
- debug endpoints and actuator paths

Quick check:

```bash
curl -sI https://target | grep -iE 'server|x-powered-by|content-security-policy|strict-transport'
```

## A06 — Vulnerable and Outdated Components

A dependency has a known vulnerability.

Test for:

- outdated frameworks and libraries
- known CVEs in the exact version
- unpatched containers and base images
- client-side libraries with public exploits

Quick check:

```bash
npm audit
pip-audit
docker scout cves image:tag
```

## A07 — Identification and Authentication Failures

Authentication can be bypassed or weakened.

Test for:

- credential stuffing and weak password policy
- missing MFA on sensitive actions
- session fixation
- predictable remember-me tokens
- password reset token leakage
- username enumeration
- JWT `alg: none`, weak secrets, and missing signature checks

Quick check: request a password reset for a valid and an invalid user; compare status, body,
and timing.

## A08 — Software and Data Integrity Failures

Code or data is trusted without verification.

Test for:

- unsigned updates and auto-update channels
- insecure deserialization
- dependency confusion
- CI/CD secrets exposed to pull requests
- unsigned artifacts and container images
- client-side trust in server-provided roles or prices

Quick check: inspect deserialization sinks (`pickle`, `ObjectInputStream`, `unserialize`) and
whether the input is user-controlled.

## A09 — Security Logging and Monitoring Failures

Attacks are not detected or recorded.

Test for:

- login failures not logged
- no alerting on privilege changes
- logs missing request IDs, source IPs, or user IDs
- log injection via newlines
- retention shorter than the incident response window

This is usually a review finding, not a single request. Pair it with a concrete attack that
leaves no trace.

## A10 — Server-Side Request Forgery (SSRF)

The server fetches a URL controlled by the user.

Test for:

- image/PDF/URL import features
- webhooks and callback URLs
- redirects that reach internal hosts
- cloud metadata endpoints
- DNS rebinding
- allowlist bypass with `@`, decimal IPs, or redirects

Quick check (authorized lab only):

```text
http://169.254.169.254/latest/meta-data/
http://127.0.0.1:8080/
http://[::1]:8080/
```

## Reporting tip

Map each finding to one category, then state the concrete impact. "A03 Injection" is not a
finding; "union-based SQL injection in `id` returns the `users` table" is.
