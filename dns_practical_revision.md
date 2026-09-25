# DNS Practical Revision Notes

A practical revision sheet for DNS enumeration, Wireshark analysis, and connecting DNS to TCP and HTTPS.

---

## 1. `dig` Basics

`dig` (Domain Information Groper) is a command-line tool used to query DNS and inspect DNS records.

### Basic lookup

```bash
dig example.com
```

### What it does

Performs a normal DNS lookup and usually shows the **A record** (IPv4) when available.

Example:

```text
;; ANSWER SECTION:
example.com.    300    IN    A    93.184.216.34
```

Read it as:

```text
example.com → 93.184.216.34
TTL         → 300 seconds
Record      → A
```

### Why use it?

- Check whether a domain resolves
- See returned IP addresses
- Inspect TTL
- Check which DNS server answered
- Get a quick view of the DNS response

---

# 2. A Record

```bash
dig A example.com
```

## What it asks

> What is the IPv4 address for `example.com`?

Example:

```text
example.com.    300    IN    A    93.184.216.34
```

### Key point

```text
A → IPv4
```

### Why use it?

Useful for finding the IPv4 address associated with a hostname.

---

# 3. AAAA Record

```bash
dig AAAA example.com
```

## What it asks

> What is the IPv6 address for `example.com`?

Example:

```text
example.com.    300    IN    AAAA    2606:2800:220:1:248:1893:25c8:1946
```

### Key point

```text
AAAA → IPv6
```

### Why use it?

A target may support both IPv4 and IPv6, so check both:

```bash
dig A example.com
dig AAAA example.com
```

---

# 4. MX Record

```bash
dig MX example.com
```

## What it asks

> Which mail servers receive email for this domain?

Example:

```text
example.com.    10    mail1.example.com.
example.com.    20    mail2.example.com.
```

### Priority

Lower number = higher priority.

```text
10 → tried first
20 → tried later
```

### Why use it?

Useful for understanding the domain's email infrastructure and for security reconnaissance.

---

# 5. TXT Record

```bash
dig TXT example.com
```

## What it asks

> What TXT information is published for this domain?

Example:

```text
"v=spf1 include:_spf.google.com ~all"
```

TXT records are commonly used for:

- SPF
- Domain verification
- Other service verification
- Email/security-related information

### Why use it?

TXT records can reveal information about email configuration and external services associated with a domain.

---

# 6. NS Record

```bash
dig NS example.com
```

## What it asks

> Which DNS servers are authoritative for this domain/zone?

Example:

```text
example.com.    NS    ns1.dns-provider.com.
example.com.    NS    ns2.dns-provider.com.
```

### Key point

```text
NS → identifies authoritative DNS servers
```

### Why use it?

Useful for understanding who handles the domain's DNS records.

---

# 7. `dig +trace`

```bash
dig +trace example.com
```

## What it does

Shows the DNS delegation process starting from the **root**.

Conceptually:

```text
Root
  ↓
.com TLD
  ↓
Authoritative server for example.com
  ↓
Final DNS record
```

### Why use it?

Useful for learning and troubleshooting:

- DNS hierarchy
- Root servers
- TLD servers
- Authoritative servers
- Delegation
- Where a DNS lookup is getting its information

### Important distinction

Normal lookup:

```text
dig example.com
```

Usually asks your configured recursive resolver and gives you the result.

Trace:

```text
dig +trace example.com
```

Shows the delegation path from the root downward.

---

# 8. DNS Hierarchy to Remember

For:

```text
www.example.com
```

the hierarchy is:

```text
.                ← Root
└── com          ← TLD
    └── example  ← Domain
        └── www  ← Hostname
```

### DNS server roles

```text
Root
↓
Provides referral to TLD servers

TLD
↓
Provides referral to authoritative DNS servers

Authoritative DNS server
↓
Provides the actual DNS record
```

Remember:

```text
Root          → "Who handles .com?"
TLD           → "Who handles example.com?"
Authoritative → "What is the record for www.example.com?"
```

---

# 9. Capture DNS in Wireshark

Start Wireshark and capture traffic on the interface carrying your network traffic.

Then generate DNS traffic from a terminal:

```bash
dig example.com
```

## Display filter

Use:

```text
dns
```

This shows DNS packets.

You can make the capture easier to recognize by generating one query at a time:

```bash
dig example.com
```

or:

```bash
dig A example.com
```

---

# 10. Identify DNS Query and Response

A DNS exchange normally contains:

```text
DNS Query
    ↓
DNS Response
```

### Query

The client asks:

```text
"What is the A record for example.com?"
```

In Wireshark, inspect the DNS section and look for:

- Transaction ID
- Queries
- Query name
- Query type

For example:

```text
Query name: example.com
Query type: A
```

### Response

The DNS server replies with the answer.

Look for:

- Transaction ID
- Answers
- Answer name
- Record type
- IP address
- TTL

Example:

```text
Answer:
example.com → 93.184.216.34
```

### Important connection

The **Transaction ID** helps match a DNS response to the DNS query that caused it.

---

# 11. Identify Source/Destination IP and Port

In the Wireshark packet list, inspect:

```text
Source
Destination
Protocol
Info
```

For a normal DNS query using UDP:

```text
Client IP: 192.168.1.10
Client port: 53124

DNS server IP: 192.168.1.1
DNS server port: 53
```

So:

```text
192.168.1.10:53124
        ↓
192.168.1.1:53
```

The response reverses the direction:

```text
192.168.1.1:53
        ↓
192.168.1.10:53124
```

## Key point

DNS commonly uses:

```text
UDP 53
```

but DNS can also use:

```text
TCP 53
```

---

# 12. Wireshark Filters to Practice

### Show all DNS traffic

```text
dns
```

### Show only DNS queries

```text
dns.flags.response == 0
```

### Show only DNS responses

```text
dns.flags.response == 1
```

### Filter traffic involving port 53

```text
udp.port == 53
```

or:

```text
tcp.port == 53
```

### Show packets from one IP

```text
ip.addr == 192.168.1.10
```

Use your actual IP address.

---

# 13. DNS → TCP → HTTPS

This is the most important connection.

When you open:

```text
https://example.com
```

the stages are:

```text
1. DNS
2. Transport connection
3. TLS
4. HTTPS / HTTP
```

---

## Step 1 — DNS

Your system needs the IP address.

```text
example.com
    ↓ DNS
93.184.216.34
```

For example:

```bash
dig A example.com
```

might return:

```text
93.184.216.34
```

DNS answers:

> Which IP address should I connect to?

---

## Step 2 — TCP

For traditional HTTPS, the browser connects to:

```text
93.184.216.34:443
```

TCP establishes a connection using the three-way handshake:

```text
Client                    Server
  |                         |
  |-------- SYN ----------->|
  |<------ SYN-ACK ---------|
  |-------- ACK ----------->|
  |                         |
```

### TCP job

TCP provides the transport connection.

So:

```text
DNS → finds IP
TCP → establishes connection to IP:443
```

---

# 14. TLS

After the TCP connection is established, HTTPS over TCP uses TLS.

Conceptually:

```text
DNS
↓
IP address
↓
TCP connection
↓
TLS handshake
↓
Encrypted HTTP
```

TLS provides:

- Encryption
- Integrity
- Server authentication through certificates

---

# 15. HTTPS / HTTP

Once TLS is established, the browser can send an HTTP request.

Conceptually:

```http
GET / HTTP/1.1
Host: example.com
```

Because this is HTTPS, the HTTP data is protected by TLS while in transit.

So the full chain is:

```text
https://example.com
        ↓
       DNS
        ↓
   IP address
        ↓
     TCP :443
        ↓
   TLS handshake
        ↓
    HTTP request
        ↓
   HTTP response
```

---

# 16. Important Modern Exception — HTTP/3

Not every HTTPS connection uses TCP.

HTTP/3 uses:

```text
QUIC
  ↓
UDP
```

usually on:

```text
UDP 443
```

So:

```text
HTTP/1.1 → TCP + TLS
HTTP/2   → TCP + TLS
HTTP/3   → QUIC over UDP
```

Therefore:

```text
DNS → finds IP
   ↓
Transport
   ├── TCP → HTTPS (HTTP/1.1, HTTP/2)
   └── QUIC/UDP → HTTP/3
```

---

# 17. DNS, TCP and HTTPS Are Different Jobs

| Technology | Main job |
|---|---|
| DNS | Domain name → IP address |
| TCP | Reliable transport connection |
| TLS | Encryption + integrity + authentication |
| HTTP | Web requests and responses |
| HTTPS | HTTP protected by TLS |

### Important

DNS does **not** carry your normal web page.

TCP does **not** resolve the domain name.

HTTPS does **not** replace DNS.

They work together.

---

# 18. Quick Practical Exercise

Run:

```bash
dig A example.com
```

Write down:

```text
Domain:
IPv4:
TTL:
DNS server:
```

Then run:

```bash
dig AAAA example.com
dig MX example.com
dig TXT example.com
dig NS example.com
dig +trace example.com
```

In Wireshark:

1. Start a capture.
2. Apply:

```text
dns
```

3. Run:

```bash
dig A example.com
```

4. Find the DNS query.
5. Find the matching DNS response.
6. Record:
   - Source IP
   - Source port
   - Destination IP
   - Destination port
   - Query name
   - Query type
   - Answer
   - TTL
   - DNS Transaction ID

---

# 19. One-Minute Revision

```text
DNS
→ Resolves a hostname to an IP address.

A
→ IPv4.

AAAA
→ IPv6.

MX
→ Mail servers.

TXT
→ Text / verification / email-security information.

NS
→ Authoritative DNS servers.

+trace
→ Shows Root → TLD → Authoritative delegation.

DNS over UDP
→ Usually port 53.

Traditional HTTPS
→ TCP 443 + TLS + HTTP.

HTTP/3
→ QUIC over UDP 443.
```

## Core flow

```text
www.example.com
       ↓
      DNS
       ↓
93.184.216.34
       ↓
TCP :443
       ↓
TLS
       ↓
HTTPS
       ↓
HTTP request/response
```
