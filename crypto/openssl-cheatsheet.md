# OpenSSL Cheatsheet

Everyday OpenSSL commands for CTFs, certificate checks, and file inspection.

## 1. Hashes

```bash
openssl dgst -sha256 file
openssl dgst -sha1 file
openssl dgst -md5 file
openssl dgst -sha512 file
```

Compare with the system tools:

```bash
sha256sum file
shasum -a 256 file
```

## 2. Base64 and hex

```bash
openssl base64 -in file -out file.b64
openssl base64 -d -in file.b64 -out file
openssl enc -base64 -A -in file
echo -n "text" | openssl enc -base64
echo "dGV4dA==" | openssl enc -base64 -d
```

Hex:

```bash
xxd file
xxd -r -p hex.txt file
```

## 3. Random data

```bash
openssl rand -hex 16
openssl rand -base64 32
openssl rand -out key.bin 32
```

## 4. Symmetric encryption

AES-256-CBC with a password (PBKDF2):

```bash
openssl enc -aes-256-cbc -pbkdf2 -salt -in plain.txt -out cipher.bin
openssl enc -d -aes-256-cbc -pbkdf2 -in cipher.bin -out plain.txt
```

AES-256-CBC with a raw key and IV:

```bash
openssl enc -aes-256-cbc -K <64-hex-key> -iv <32-hex-iv> -in plain.txt -out cipher.bin
openssl enc -d -aes-256-cbc -K <64-hex-key> -iv <32-hex-iv> -in cipher.bin -out plain.txt
```

GCM is not supported by the `enc` command in the same way; use a library for AEAD.

## 5. RSA keys

```bash
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
openssl rsa -in private.pem -text -noout
```

Inspect a key:

```bash
openssl rsa -in private.pem -check -noout
openssl pkey -in private.pem -text -noout
```

Encrypt and decrypt small data:

```bash
openssl pkeyutl -encrypt -pubin -inkey public.pem -in secret.bin -out secret.enc
openssl pkeyutl -decrypt -inkey private.pem -in secret.enc -out secret.bin
```

## 6. Certificates and CSRs

Generate a key and CSR:

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr \
  -subj "/CN=example.com"
```

Self-signed certificate:

```bash
openssl req -x509 -newkey rsa:2048 -nodes -keyout server.key -out server.crt \
  -days 365 -subj "/CN=example.com"
```

Inspect:

```bash
openssl x509 -in server.crt -text -noout
openssl req -in server.csr -text -noout
openssl verify -CAfile ca.crt server.crt
```

Convert formats:

```bash
openssl x509 -in server.crt -outform der -out server.der
openssl x509 -in server.der -inform der -out server.pem
openssl pkcs12 -export -out bundle.p12 -inkey server.key -in server.crt
openssl pkcs12 -in bundle.p12 -nodes -out bundle.pem
```

## 7. TLS checks

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null
openssl s_client -connect example.com:443 -servername example.com -tls1_2 </dev/null
openssl s_client -connect example.com:443 -servername example.com -showcerts </dev/null
```

Extract the certificate:

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null \
  | openssl x509 -text -noout
```

Check certificate dates:

```bash
openssl x509 -in server.crt -noout -dates -subject -issuer
```

## 8. Diffie-Hellman

```bash
openssl dhparam -out dhparam.pem 2048
openssl dhparam -in dhparam.pem -text -noout
```

## 9. Password-protected files

```bash
openssl rsa -in key.pem -passin pass:<password> -text -noout
openssl pkcs12 -in bundle.p12 -passin pass:<password> -nodes -out bundle.pem
```

## 10. File inspection

```bash
file file.bin
openssl asn1parse -in file.der -i
openssl x509 -in file.der -inform der -text -noout
strings -a file.bin | head
```

## 11. CTF patterns

- Base64 inside a certificate field: `openssl x509 -in cert.pem -text -noout`
- RSA with a small `e`: inspect `openssl rsa -text -noout`
- Shared prime: compare `modulus` values with `openssl rsa -modulus`
- Encrypted flag: identify the cipher from the file header and key length
- PEM with multiple blocks: split on `-----BEGIN` / `-----END`

## 12. Gotchas

- `-K` and `-iv` expect hex, not ASCII.
- `enc` with `-pbkdf2` is not the same as a raw key.
- `s_client` can hang waiting for input; redirect `</dev/null`.
- SNI matters: always pass `-servername` for virtual hosts.
- Old TLS versions may be disabled by the system policy; use `-tls1_2` explicitly when testing.
