# OWASP Top 10 Quick Reference

## OWASP Web Application Top 10 (2021)

### A01 — Broken Access Control
- **CWE:** 284, 285, 639
- **What it is:** Users can act outside their intended permissions (IDOR, privilege escalation, CORS misconfiguration)
- **Test for:** Access other users' resources by modifying IDs; access admin endpoints without auth
- **Mitigate:** Deny by default; enforce server-side; log access control failures; rate-limit API

### A02 — Cryptographic Failures
- **CWE:** 261, 296, 310, 319, 321, 326, 327
- **What it is:** Weak/missing encryption for sensitive data at rest or in transit
- **Test for:** HTTP instead of HTTPS; deprecated TLS versions; weak ciphers; secrets in code
- **Mitigate:** TLS 1.2+ everywhere; AES-256/ChaCha20 at rest; bcrypt/Argon2 for passwords; never hardcode secrets

### A03 — Injection
- **CWE:** 89 (SQLi), 77 (Command), 79 (XSS), 917 (EL)
- **What it is:** Untrusted data sent to an interpreter (SQL, OS, LDAP, NoSQL)
- **Test for:** `' OR 1=1--`, `; id`, template injection payloads
- **Mitigate:** Parameterized queries; ORM; input validation; least privilege DB accounts; WAF

### A04 — Insecure Design
- **CWE:** 209, 256, 501, 522
- **What it is:** Missing or ineffective security controls by design (not just implementation bugs)
- **Mitigate:** Threat modeling early; security user stories; abuse cases; secure design patterns

### A05 — Security Misconfiguration
- **CWE:** 16
- **What it is:** Default creds, unnecessary features enabled, verbose errors, missing hardening
- **Test for:** Default passwords; exposed admin interfaces; directory listing; verbose stack traces
- **Mitigate:** Hardening checklists; disable unused features; automated config scanning (Checkov, Scout Suite)

### A06 — Vulnerable & Outdated Components
- **CWE:** 1104
- **What it is:** Using libraries/frameworks with known vulnerabilities
- **Test for:** `npm audit`, `pip-audit`, Snyk, OWASP Dependency-Check
- **Mitigate:** SCA in CI/CD; Dependabot; maintain SBOM; subscribe to CVE feeds

### A07 — Identification & Authentication Failures
- **CWE:** 287, 295, 297, 384
- **What it is:** Weak passwords, missing MFA, broken session management, credential stuffing
- **Test for:** Brute force; session fixation; JWT alg:none; predictable tokens
- **Mitigate:** MFA; strong password policy; secure session cookies (HttpOnly, Secure, SameSite); short session timeouts

### A08 — Software & Data Integrity Failures
- **CWE:** 494, 502, 565, 784, 915
- **What it is:** Code/data assumed trusted without verification (insecure deserialization, CI/CD compromise)
- **Mitigate:** Verify signatures/checksums; secure CI/CD pipelines; avoid deserialization of untrusted data

### A09 — Security Logging & Monitoring Failures
- **CWE:** 117, 223, 532, 778
- **What it is:** Insufficient logging, no alerting, logs not protected
- **Mitigate:** Log auth events, access failures, input validation failures; centralize (SIEM); alert on anomalies; protect log integrity

### A10 — Server-Side Request Forgery (SSRF)
- **CWE:** 918
- **What it is:** App fetches remote resource using user-supplied URL, enabling internal network access
- **Test for:** Supply internal IPs (169.254.169.254 for cloud metadata); file:// URIs
- **Mitigate:** Allowlist permitted destinations; block RFC-1918 ranges; disable redirects; validate URL schemes

---

## OWASP API Security Top 10 (2023)

| # | Name | Key issue | Mitigation |
|---|------|-----------|------------|
| API1 | Broken Object Level Authorization | IDOR on object IDs | Authorization check per request per user |
| API2 | Broken Authentication | Weak tokens, no rate limit | Strong auth; MFA; rate limiting |
| API3 | Broken Object Property Level Auth | Exposing sensitive fields | Return only necessary fields; filter on server |
| API4 | Unrestricted Resource Consumption | No rate/size limits | Rate limiting; payload size limits; quotas |
| API5 | Broken Function Level Authorization | Accessing admin-only functions | Explicit function-level access control |
| API6 | Unrestricted Access to Sensitive Business Flows | Automated abuse (scalping, spam) | Bot detection; CAPTCHA; behavioral limits |
| API7 | Server-Side Request Forgery | Fetching internal resources | URL allowlisting; block internal ranges |
| API8 | Security Misconfiguration | Exposed debug endpoints, CORS | Hardening; disable unnecessary features |
| API9 | Improper Inventory Management | Shadow/zombie APIs | API gateway; full API inventory |
| API10 | Unsafe Consumption of APIs | Trusting third-party API data blindly | Validate external data; least privilege |

---

## Common Security Headers

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{random}'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), camera=(), microphone=()
```

Test with: [securityheaders.com](https://securityheaders.com) or `curl -I https://target`

---

## JWT Common Vulnerabilities

| Attack | Description | Prevention |
|--------|-------------|------------|
| alg:none | Remove signature verification | Always validate `alg`; reject `none` |
| Algorithm confusion | Switch RS256→HS256 using public key | Pin expected algorithm server-side |
| Weak secret | Brute-force HS256 secret | Use 256-bit+ random secret |
| No expiry | Token valid forever | Always set `exp`; short-lived tokens |
| Sensitive data in payload | Data readable without key | Never put secrets in JWT payload |
