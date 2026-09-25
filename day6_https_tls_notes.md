# DAY 6 — HTTPS + TLS

## Goal

Understand HTTPS/TLS practically enough to inspect encrypted web traffic in Wireshark and understand what certificates do.

---

# 1. HTTP vs HTTPS

HTTP sends web data without TLS protection.

Example HTTP traffic can expose:

```http
GET /login HTTP/1.1
Host: example.com

username=naruto&password=123
```

HTTPS is essentially:

```text
HTTP
  +
TLS
  =
HTTPS
```

TLS protects the application data while it travels across the network.

---

# 2. TLS Purpose

TLS mainly provides:

- Confidentiality — protects data from being read by network observers.
- Integrity — helps detect tampering with protected traffic.
- Authentication — certificates help the client verify the server identity.

Basic mental model:

```text
Browser
   |
   | TLS-protected communication
   |
   v
Website
```

---

# 3. TLS Handshake

The handshake happens before normal encrypted application data is exchanged.

In Wireshark, useful packets include:

```text
Client Hello
Server Hello
Certificate
...
Application Data
```

Basic flow:

```text
CLIENT
  |
  | ClientHello
  v
SERVER
  |
  | ServerHello
  | Certificate
  v
CLIENT
  |
  | TLS negotiation
  v
ENCRYPTED APPLICATION DATA
```

The handshake is used to establish the parameters and cryptographic keys needed for the secure connection.

---

# 4. Wireshark Practical

Capture traffic while visiting an HTTPS website.

Useful Wireshark display filter:

```text
tls
```

Then look for:

- Client Hello
- Server Hello
- Certificate
- Application Data

A Client Hello can contain information such as:

- TLS versions supported
- Cipher suites
- Client random
- Extensions
- Server Name Indication (SNI)

After the handshake, application traffic generally appears as:

```text
Application Data
```

rather than readable HTTP.

---

# 5. Certificates

A TLS certificate contains information used to identify a server and establish trust.

Important fields:

```text
Subject
Issuer
Validity
Public Key
```

Think:

```text
Subject
  ↓
Who is the certificate for?

Issuer
  ↓
Who issued/signed the certificate?
```

Example:

```text
Subject: example.com
Issuer: Example Intermediate CA
```

The browser checks the certificate and its trust relationships before accepting the connection.

---

# 6. Certificate Chain

Certificates commonly form a chain:

```text
Trusted Root CA
       ↓
Intermediate CA
       ↓
Website Certificate
       ↓
example.com
```

The browser can use the chain to establish whether the website certificate ultimately connects to a trusted root.

The server commonly sends its certificate and required intermediate certificates. The root certificate is generally already present in the client's trust store.

---

# 7. Encryption vs Authentication

These are different concepts.

### Encryption

Protects the contents of the communication.

Without TLS:

```text
username=naruto
password=123
```

With TLS:

```text
████████████████
████ ENCRYPTED █
████████████████
```

### Authentication

The certificate and certificate validation help the browser establish that it is communicating with the server identified by the certificate.

Remember:

```text
Encryption
→ protects the data

Certificate validation
→ helps authenticate the server
```

---

# 8. What Wireshark Can and Cannot See

Even when HTTPS is being used, a network observer can still generally see metadata such as:

- Source IP
- Destination IP
- Ports
- Packet sizes
- Timing
- TLS handshake information
- Some TLS metadata
- Certificate information when transmitted

But without the required decryption keys/context, the observer normally cannot simply read the encrypted HTTP request/response contents.

For example, the observer should not simply see:

```text
username=naruto
password=123
```

inside modern properly configured HTTPS application data.

---

# 9. OpenSSL Certificate Inspection

Useful command:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Look for:

```text
Certificate chain
Server certificate
subject=
issuer=
```

To display the certificates sent by the server:

```bash
openssl s_client -connect example.com:443 -servername example.com -showcerts
```

The `-servername` option sends the hostname using SNI, which is important when a server hosts multiple HTTPS sites.

---

# 10. TLS Versions

Modern TLS deployments commonly use:

```text
TLS 1.2
TLS 1.3
```

Older versions such as TLS 1.0 and TLS 1.1 are deprecated.

You can test a server's supported protocol versions with OpenSSL:

```bash
openssl s_client -connect example.com:443 -servername example.com -tls1_2
```

and:

```bash
openssl s_client -connect example.com:443 -servername example.com -tls1_3
```

A failure does not automatically mean a security problem. It can simply mean the client and server do not have a compatible configuration.

---

# 11. Core Mental Model

```text
                 HTTPS
                   |
                   v
                  TLS
            _______|_______
           /                         v                 v
    ENCRYPTION         AUTHENTICATION
          |                 |
          v                 v
   Protects data       Certificate
   in transit          validation
```

---

# 12. Day 6 Practical Checklist

- [x] Capture HTTPS traffic in Wireshark
- [x] Filter TLS traffic
- [x] Identify Client Hello
- [x] Identify Server Hello
- [x] Identify Certificate
- [x] Identify Application Data
- [x] Inspect certificate information
- [x] Use OpenSSL to inspect certificates
- [x] Understand certificate subject
- [x] Understand certificate issuer
- [x] Understand certificate chain
- [x] Understand encryption vs authentication
- [x] Understand what HTTPS hides from network observers
- [x] Understand basic TLS versions

---

# FINAL CHEAT SHEET

```text
HTTP
→ Web protocol

HTTPS
→ HTTP protected by TLS

TLS
→ Secure communication protocol

Client Hello
→ Client starts TLS negotiation

Server Hello
→ Server responds and selects TLS parameters

Certificate
→ Used to establish server identity/trust

Subject
→ Certificate identity

Issuer
→ Certificate issuer

Certificate chain
→ Website certificate → Intermediate CA → Trusted Root

Encryption
→ Protects communication contents

Wireshark
→ Can observe TLS traffic and metadata

Encrypted Application Data
→ HTTP contents are protected

OpenSSL
→ Command-line tool for inspecting TLS/certificates
```

## The one thing to remember

```text
HTTP:
Browser ─────── readable HTTP ───────> Server

HTTPS:
Browser ─────── TLS encrypted ───────> Server
                    ↑
              certificate/
              handshake
```
