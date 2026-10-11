# HTTP, HTTPS, SSL/TLS, Certificates, CAs & CDN

> How does your browser know a website is safe? Why does that little padlock appear? And how do websites load fast from anywhere in the world? This guide explains it step by step, in plain language.

---

## Table of Contents

1. [HTTP vs HTTPS](#1-http-vs-https)
2. [SSL vs TLS](#2-ssl-vs-tls)
3. [TLS Handshake](#3-tls-handshake)
4. [SSL/TLS Certificate](#4-ssltls-certificate)
5. [Certificate Authority (CA)](#5-certificate-authority-ca)
6. [CDN (Content Delivery Network)](#6-cdn-content-delivery-network)
7. [Real-Time Example](#7-real-time-example)
8. [How It All Fits Together](#8-how-it-all-fits-together)
9. [Quick Hands-On Commands](#9-quick-hands-on-commands)
10. [Cheat Sheet](#10-cheat-sheet)

---

## 1. HTTP vs HTTPS

### What is HTTP?

**HTTP (HyperText Transfer Protocol)** is the language a browser and a web server use to talk to each other. Your browser asks for a page, the server sends it back.

The problem: **HTTP sends everything as plain text.** Anyone between you and the server (public Wi-Fi owner, ISP, an attacker) can read or change it.

### What is HTTPS?

**HTTPS (HTTP Secure)** is HTTP **wrapped inside TLS encryption**. The conversation is the same, but it's scrambled so only you and the server can understand it.

> **Analogy:** HTTP is sending a **postcard** — the mail carrier can read it. HTTPS is sending a **locked box** — only the receiver has the key.

### Side-by-Side Comparison

| Feature | HTTP | HTTPS |
|---|---|---|
| Full name | HyperText Transfer Protocol | HyperText Transfer Protocol **Secure** |
| Default port | **80** | **443** |
| Encryption | None (plain text) | Encrypted with TLS |
| Certificate needed | No | Yes |
| Data integrity | Can be modified in transit | Tampering is detected |
| Server identity verified | No | Yes (via certificate) |
| Browser indicator | "Not secure" warning | Padlock |
| SEO / trust | Lower | Higher (Google favors HTTPS) |

### The Three Promises of HTTPS

1. **Confidentiality** — nobody else can read your data (encryption).
2. **Integrity** — nobody can secretly change your data (tamper detection).
3. **Authentication** — you're really talking to the real website, not an imposter.

### HTTPS Flow Diagram

This is the full journey of a single HTTPS page load, from typing the address to seeing the page.

```text
  User / Browser            DNS Server              Web Server
        |                       |                       |
        | 1. Type https://example.com                   |
        |                       |                       |
        | 2. "What is the IP of example.com?"           |
        |---------------------->|                       |
        |<----------------------|                       |
        |   "93.184.216.34"     |                       |
        |                                               |
        | 3. TCP handshake (SYN, SYN-ACK, ACK) on port 443
        |---------------------------------------------->|
        |<----------------------------------------------|
        |                                               |
        | 4. TLS handshake                              |
        |    - ClientHello (versions, ciphers)          |
        |    - ServerHello + Certificate                |
        |    - Browser verifies certificate             |
        |    - Session key agreed                       |
        |<=============================================>|
        |                                               |
        | 5. Encrypted HTTP request                     |
        |    GET /index.html                            |
        |==============================================>|
        |                                               |
        | 6. Encrypted HTTP response                    |
        |    200 OK + page content                      |
        |<==============================================|
        |                                               |
        | 7. Browser decrypts and renders the page      |
        |    (padlock shown in the address bar)         |
        |                                               |
        | 8. Connection closed or kept alive            |
        |<=============================================>|
```

| Step | What happens |
|---|---|
| 1 | You enter an `https://` URL |
| 2 | DNS translates the domain name into an IP address |
| 3 | A TCP connection is opened to port 443 |
| 4 | The TLS handshake verifies the server and creates a shared session key |
| 5 | The browser sends the HTTP request, encrypted with the session key |
| 6 | The server sends the HTTP response, also encrypted |
| 7 | The browser decrypts the data and displays the page |
| 8 | The connection is closed, or reused for further requests |

### What an attacker sees

```text
HTTP  →  POST /login  username=sinsha&password=MySecret123      (readable!)
HTTPS →  x8F#kL9$2mQ!zP...a7B^&*(...                            (gibberish)
```

---

## 2. SSL vs TLS

### What is SSL?

**SSL (Secure Sockets Layer)** is the original security protocol, created by Netscape in the mid-1990s, to protect data sent between a browser and a web server. It does three things:

- **Encrypts** the data so outsiders cannot read it
- **Authenticates** the server using a certificate
- **Protects integrity** so data cannot be changed unnoticed

SSL had serious vulnerabilities (for example the POODLE attack on SSL 3.0), so it is now deprecated and should not be used.

### What is TLS?

**TLS (Transport Layer Security)** is the modern, improved successor to SSL. It does the same job (encryption, authentication, integrity) but with stronger algorithms, a faster handshake and no known legacy weaknesses. TLS is what secures HTTPS today, and it is also used for email (SMTPS, IMAPS), VPNs, databases and APIs.

> Simple way to remember: **SSL is the old name, TLS is the current technology.** A "SSL certificate" you buy today is actually used with TLS.

### The Short Story

**SSL (Secure Sockets Layer)** was the first protocol for encrypting web traffic. It had security flaws and was replaced by **TLS (Transport Layer Security)**.

> When people say "SSL" today, they almost always mean **TLS**. The name "SSL" just stuck, like calling every tissue a "Kleenex".

### Version History

| Protocol | Year | Status |
|---|---|---|
| SSL 1.0 | — | Never released (flawed) |
| SSL 2.0 | 1995 | Broken, deprecated |
| SSL 3.0 | 1996 | Broken (POODLE attack), deprecated |
| TLS 1.0 | 1999 | Deprecated |
| TLS 1.1 | 2006 | Deprecated |
| **TLS 1.2** | 2008 | Secure, widely used |
| **TLS 1.3** | 2018 | Newest, fastest, most secure |

### SSL vs TLS at a Glance

| | SSL | TLS |
|---|---|---|
| Era | Old (1995–1996) | Modern (1999–present) |
| Security | Vulnerable | Strong |
| Handshake speed | Slower | Faster (especially TLS 1.3) |
| Should you use it? | **No** | **Yes** |

**Best practice:** Enable **TLS 1.2 and TLS 1.3 only**. Disable everything older.

---

## 3. TLS Handshake

Before any encrypted data flows, the browser and server must **agree on how to encrypt it**. This negotiation is the **TLS Handshake**. It takes a fraction of a second.

### Goals of the Handshake

- Agree on the **TLS version** and **cipher suite** (encryption methods)
- **Verify the server's identity** using its certificate
- Safely create a shared **session key** for fast encryption

### Simplified Flow (TLS 1.2 style)

```text
   Browser (Client)                                Web Server
         │                                              │
         │  1. ClientHello                              │
         │  "Hi! I support TLS 1.2/1.3, these ciphers…" │
         │─────────────────────────────────────────────▶│
         │                                              │
         │  2. ServerHello + Certificate                │
         │  "Let's use TLS 1.3. Here's my certificate." │
         │◀─────────────────────────────────────────────│
         │                                              │
         │  3. Browser verifies the certificate         │
         │     (trusted CA? valid dates? right domain?) │
         │                                              │
         │  4. Key Exchange                             │
         │  Both sides compute the same secret          │
         │  session key (without ever sending it!)      │
         │◀────────────────────────────────────────────▶│
         │                                              │
         │  5. Finished                                 │
         │◀────────────────────────────────────────────▶│
         │                                              │
         │  Encrypted HTTP traffic begins               │
         │◀════════════════════════════════════════════▶│
```

### Step-by-Step Explanation

| Step | What happens | Plain-English meaning |
|---|---|---|
| **1. ClientHello** | Browser sends supported TLS versions, cipher suites, and a random number | "Here's what I can do." |
| **2. ServerHello** | Server picks a version + cipher and sends its certificate and a random number | "Let's use this. Here's my ID." |
| **3. Certificate check** | Browser validates the certificate against trusted CAs | "Is this ID real and not expired?" |
| **4. Key exchange** | Both sides derive a shared **session key** | "Let's agree on a secret code." |
| **5. Finished** | Both confirm the handshake worked | "Ready! Let's talk securely." |

### Two Types of Encryption Used

| Type | Also called | Used for | Speed |
|---|---|---|---|
| **Asymmetric** | Public/private key | Handshake & identity verification | Slow |
| **Symmetric** | Shared session key | Encrypting the actual data | Fast |

> TLS uses the slow-but-clever asymmetric method **only to safely agree on a key**, then switches to fast symmetric encryption for everything else.

### TLS 1.2 vs TLS 1.3 Handshake

| | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Round trips | 2 | **1** (even 0 on resume) |
| Speed | Good | Faster |
| Weak ciphers | Some allowed | Removed |

---

## 4. SSL/TLS Certificate

### What Is It?

An **SSL/TLS certificate** is a small digital file that acts like a **website's passport**. It proves:

- *"This is really `example.com`"*
- *"Here is my public key for encryption"*
- *"A trusted authority vouches for me"*

### What's Inside a Certificate?

| Field | Example | Purpose |
|---|---|---|
| Subject / Common Name | `www.example.com` | The domain it belongs to |
| Subject Alternative Names (SAN) | `example.com`, `api.example.com` | Other domains it covers |
| Public key | `MIIBIjANBgkq...` | Used for encryption |
| Issuer | `Let's Encrypt` | The CA that signed it |
| Valid from / to | `2026-01-01 → 2026-04-01` | Validity period |
| Digital signature | `3a:f2:9c...` | CA's proof of authenticity |

### Types of Certificates

**By validation level**

| Type | What's verified | Issued in | Best for |
|---|---|---|---|
| **DV** – Domain Validated | You control the domain | Minutes | Blogs, personal sites, most apps |
| **OV** – Organization Validated | Domain + organization is real | Days | Business websites |
| **EV** – Extended Validation | Strict legal/business checks | Days–weeks | Banks, large enterprises |

**By coverage**

| Type | Covers |
|---|---|
| **Single-domain** | One domain (`www.example.com`) |
| **Wildcard** | All subdomains (`*.example.com`) |
| **Multi-domain (SAN)** | Several different domains in one cert |

### Where Do You Get One?

- **Free:** [Let's Encrypt](https://letsencrypt.org), **AWS Certificate Manager (ACM)** (free for use with AWS services like ALB, CloudFront)
- **Paid:** DigiCert, Sectigo, GlobalSign, etc.

### Certificate Lifecycle

```text
Generate key pair + CSR  →  Submit to CA  →  CA validates you
        →  CA issues certificate  →  Install on server  →  Renew before expiry
```

> Certificates **expire** (commonly 90 days to 1 year). Expired certificate = scary browser warning. Automate renewal (e.g., Certbot, ACM auto-renew)!

---

## 5. Certificate Authority (CA)

### What Is a CA?

A **Certificate Authority (CA)** is a trusted organization that **issues and digitally signs certificates**. It's like a **government passport office**: anyone can *claim* to be someone, but a passport is trusted because a trusted authority verified and stamped it.

### Why Do Browsers Trust a CA?

Your operating system and browser ship with a built-in list called the **Root Store** (trusted root CAs). If a certificate was signed by a CA in that list (directly or through a chain), the browser trusts it.

### The Chain of Trust

```text
┌──────────────────────────────┐
│   Root CA                    │  ← Pre-installed in your browser/OS (trusted)
│   (e.g., ISRG Root X1)       │
└──────────────┬───────────────┘
               │ signs
               ▼
┌──────────────────────────────┐
│   Intermediate CA            │  ← Issues certs day-to-day, protects the root
│   (e.g., Let's Encrypt R10)  │
└──────────────┬───────────────┘
               │ signs
               ▼
┌──────────────────────────────┐
│   Server Certificate         │  ← The certificate on www.example.com
│   (www.example.com)          │
└──────────────────────────────┘
```

The browser walks **up the chain**: *server cert → intermediate → root*. If every link is valid and the root is trusted  → padlock shown.

### How Does a CA Issue a Certificate?

1. You generate a **private key** (keep it secret!) and a **CSR** (Certificate Signing Request).
2. You send the CSR to the CA.
3. The CA **validates** you (e.g., proves you control the domain via DNS record or file).
4. The CA **signs** and returns your certificate.
5. You install it on your server.

### Popular CAs

Let's Encrypt, DigiCert, Sectigo, GlobalSign, GoDaddy, Amazon (ACM), Google Trust Services.

### Self-Signed Certificates

You can sign a certificate **yourself** — no CA involved. Encryption works, but browsers show a warning because no trusted authority vouches for it.

| | CA-signed | Self-signed |
|---|---|---|
| Browser trust | Trusted | Warning |
| Cost | Free–paid | Free |
| Use for | Production | Local dev / testing |

### Common Certificate Errors

| Browser error | Likely cause |
|---|---|
| `NET::ERR_CERT_DATE_INVALID` | Certificate expired |
| `NET::ERR_CERT_COMMON_NAME_INVALID` | Domain doesn't match the certificate |
| `NET::ERR_CERT_AUTHORITY_INVALID` | Self-signed or unknown CA |
| `Incomplete chain` | Intermediate certificate missing on server |

---

## 6. CDN (Content Delivery Network)

### What Is a CDN?

A **CDN** is a network of servers spread across the world that **stores copies of your website's content close to your users**.

Without a CDN, a user in Bahrain loading a site hosted in the USA must wait for data to travel halfway around the globe. With a CDN, they get it from a nearby server.

> **Analogy:** Instead of one giant warehouse in one country shipping every order worldwide, you build **local mini-warehouses** in many cities. Deliveries become much faster.

### How a CDN Works

```text
WITHOUT CDN
User (Bahrain) ───────────── long distance ─────────────▶ Origin Server (USA)

WITH CDN
User (Bahrain) ──▶ Nearest CDN Edge (Middle East) ──(only if needed)──▶ Origin Server
                        │
                        └─ Cached copy returned instantly
```

### Key Terms

| Term | Meaning |
|---|---|
| **Origin server** | Your real server where the original content lives |
| **Edge location / PoP** | CDN server close to users that serves cached content |
| **Cache hit** | Edge already has the content → served instantly |
| **Cache miss** | Edge doesn't have it → fetches from origin, then caches it |
| **TTL (Time To Live)** | How long content stays cached before refresh |
| **Cache invalidation** | Forcing the CDN to drop old content after you update it |

### Benefits

- **Faster load times** (lower latency)
- **Reduced load** on your origin server
- **DDoS protection** and traffic absorption
- **Global reach** and high availability
- **Lower bandwidth costs**
- **Free/managed HTTPS** at the edge

### What Do CDNs Cache?

Images, CSS, JavaScript, videos, fonts, downloads — and with configuration, even API responses and full pages.

### CDN + HTTPS: How They Connect

CDNs terminate TLS at the edge, so users get a **fast handshake with a nearby server**. You then choose how the CDN talks to your origin:

| Mode | User → CDN | CDN → Origin | Security |
|---|---|---|---|
| HTTP only | HTTP | HTTP | Poor |
| Flexible | HTTPS | HTTP | Partial — avoid |
| **Full / End-to-End** | HTTPS | HTTPS | **Recommended** |

### Popular CDNs

**Amazon CloudFront**, Cloudflare, Akamai, Fastly, Google Cloud CDN, Azure CDN.

### AWS Example: CloudFront

```text
User ──HTTPS──▶ CloudFront (Edge, ACM certificate) ──HTTPS──▶ S3 bucket / ALB / EC2
```

- Certificates for CloudFront come from **AWS Certificate Manager (ACM)** — must be requested in **us-east-1 (N. Virginia)**.
- Use **Origin Access Control (OAC)** to keep your S3 bucket private so only CloudFront can read it.

---

## 7. Real-Time Example

Let's follow what really happens when you open an online shopping site, `https://www.shop-example.com`, and log in from Bahrain.

### The Setup

- The site's files live on an **origin server** in the USA (AWS `us-east-1`).
- The site uses a **CDN (Amazon CloudFront)** with edge locations around the world.
- Its certificate was issued by a **CA (Amazon, via ACM)** and attached to the CDN.

### What Happens, Step by Step

| # | Event | What is going on |
|---|---|---|
| 1 | You type `https://www.shop-example.com` | The `https` scheme tells the browser to use port 443 and TLS |
| 2 | DNS lookup | DNS returns the IP of the **nearest CloudFront edge** (for example in the Middle East), not the USA server |
| 3 | TCP connection | Browser connects to the edge on port 443 |
| 4 | TLS ClientHello | Browser says: "I support TLS 1.3 and TLS 1.2, here are my ciphers" |
| 5 | ServerHello + certificate | Edge replies: "Let's use TLS 1.3. Here is my certificate for `www.shop-example.com`" |
| 6 | Certificate check | Browser confirms: signed by a trusted CA, not expired, domain name matches |
| 7 | Session key | Both sides derive the same secret key. The key itself is never sent over the network |
| 8 | Padlock appears | The connection is now encrypted |
| 9 | Page request | Browser sends `GET /` over the encrypted channel |
| 10 | CDN cache | **Cache hit:** the edge returns the page instantly. **Cache miss:** the edge fetches it from the origin over HTTPS, caches it, then returns it |
| 11 | You log in | Your username and password travel inside the encrypted tunnel |
| 12 | Response | The server answers with your account page, encrypted back to you |

### What Different People Can See

| Who | Over plain HTTP | Over HTTPS |
|---|---|---|
| Public Wi-Fi owner | Sees your username, password and pages | Sees only that you connected to `shop-example.com`, not the content |
| Your ISP | Sees everything | Sees the domain name only |
| An attacker running a fake site | Can impersonate the site easily | Cannot, because the certificate would not match or be trusted |
| An attacker modifying traffic | Can inject ads or malware | Changes are detected and the connection fails |

### What If Something Goes Wrong?

| Situation | What the browser does |
|---|---|
| The certificate expired yesterday | Shows a full-page warning (`ERR_CERT_DATE_INVALID`) |
| The attacker presents a self-signed certificate | Shows `ERR_CERT_AUTHORITY_INVALID` |
| The certificate is for `other-site.com` | Shows `ERR_CERT_COMMON_NAME_INVALID` |

### Seeing It Yourself

Open your browser, visit any HTTPS site, click the padlock in the address bar, then view the certificate. You will see the issuer (CA), the validity dates and the domains it covers. You can also run:

```bash
curl -vI https://www.example.com
```

and read the lines that show the TLS version, cipher suite, certificate subject, issuer and expiry date.

---

## 8. How It All Fits Together

When you open `https://www.example.com`, here's the whole story:

```text
1. You type https://www.example.com
2. DNS finds the nearest CDN edge server's IP address
3. TLS Handshake begins with the edge server
4. Edge sends its SSL/TLS certificate
5. Browser checks: was it signed by a trusted CA? Valid dates? Right domain?
6. Trusted → session key agreed → connection encrypted 
7. CDN returns cached content (or fetches it from origin first)
8. You see the page — fast and secure!
```

| Concept | Role in the story |
|---|---|
| **HTTPS** | The secure way to browse |
| **TLS** | The encryption protocol behind HTTPS |
| **Handshake** | The "let's agree on a secret" negotiation |
| **Certificate** | The website's ID card |
| **CA** | The trusted authority that signs ID cards |
| **CDN** | Delivers content fast from nearby servers |

---

## 9. Quick Hands-On Commands

Try these in your Linux/macOS terminal.

**Check a site's certificate details**
```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer -dates
```

**See the full certificate chain**
```bash
openssl s_client -connect example.com:443 -showcerts </dev/null
```

**Test which TLS version a server supports**
```bash
openssl s_client -connect example.com:443 -tls1_3 </dev/null
```

**Verbose HTTPS request (shows the handshake)**
```bash
curl -vI https://example.com
```

**Check certificate expiry quickly**
```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -enddate
```

**Generate a self-signed certificate (for local testing)**
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout server.key -out server.crt -subj "/CN=localhost"
```

**Check if a site is served through a CDN (look at headers)**
```bash
curl -sI https://example.com | grep -iE "server|via|x-cache|cf-ray|x-amz-cf"
```

> *Tip for your portfolio: add screenshots of the browser padlock → certificate details, and your terminal output here.*

---

## 10. Cheat Sheet

| Question | Answer |
|---|---|
| HTTP port? | 80 |
| HTTPS port? | 443 |
| Does HTTPS encrypt data? | Yes, using TLS |
| Is SSL still used? | No — replaced by TLS |
| Best TLS versions? | 1.2 and 1.3 |
| What does the handshake do? | Agrees on encryption + verifies the server |
| What's a certificate? | A digital ID proving a site's identity |
| Who issues certificates? | A Certificate Authority (CA) |
| Free certificate option? | Let's Encrypt, AWS ACM |
| What does a CDN do? | Serves cached content from servers near the user |
| Best CDN-to-origin mode? | End-to-end HTTPS |

### Key Takeaways

- **HTTPS = HTTP + TLS** → encrypted, authenticated, tamper-proof.
- **TLS replaced SSL**; use TLS 1.2/1.3 only.
- The **TLS handshake** safely creates a shared secret key.
- A **certificate** proves identity; a **CA** vouches for it.
- A **CDN** makes sites faster, safer, and more reliable worldwide.

---

## References & Further Reading

- [Mozilla MDN — Transport Layer Security](https://developer.mozilla.org/en-US/docs/Web/Security/Transport_Layer_Security)
- [Let's Encrypt — How It Works](https://letsencrypt.org/how-it-works/)
- [Cloudflare Learning Center — What is TLS?](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/)
- [AWS Certificate Manager Documentation](https://docs.aws.amazon.com/acm/)
- [Amazon CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)

---

*Written as part of my #AWSDevOpsRestartJourney learning series. Feedback and corrections are welcome!*
