# Example Security Reports

## Example 1 — Web Application Penetration Test Report

---

# Security Assessment Report
**Target:** customer-portal.acme.com (Web Application)
**Assessment Type:** External Web Application Penetration Test
**Date:** 2024-03-15
**Assessor:** Security Team
**Classification:** CONFIDENTIAL — For Internal Use Only

---

## Executive Summary

An external penetration test was conducted against the ACME customer portal between March 10–14, 2024. The assessment identified **12 vulnerabilities**, including two critical issues that allow unauthenticated access to customer data and remote code execution on the underlying server.

The most significant findings stem from insufficient input validation and outdated third-party components. Immediate remediation of the critical and high findings is strongly recommended before the application is used to process live customer data.

Overall risk posture: **HIGH**

---

## Scope & Methodology

**In scope:**
- https://customer-portal.acme.com (all endpoints)
- API: https://api.acme.com/v1 and /v2

**Out of scope:**
- Infrastructure / network layer
- Third-party integrations (Stripe, Salesforce)

**Methodology:** OWASP Testing Guide v4.2, PTES
**Tools used:** Burp Suite Pro, OWASP ZAP, sqlmap (controlled), ffuf, Nuclei

---

## Risk Summary

| Severity | Count |
|----------|-------|
| Critical | 2 |
| High | 3 |
| Medium | 4 |
| Low | 2 |
| Informational | 1 |
| **Total** | **12** |

---

## Findings

### CRIT-001 — SQL Injection in Search Endpoint

- **Severity:** Critical
- **CVSS Score:** 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
- **CWE:** CWE-89 — Improper Neutralization of Special Elements in SQL Commands
- **Affected component:** `GET /api/v1/products/search?q=`

**Description:**
The `q` parameter in the product search endpoint is directly concatenated into a SQL query without parameterization, allowing an attacker to manipulate the query logic.

**Evidence:**
```
Request:  GET /api/v1/products/search?q=' OR '1'='1
Response: 200 OK — returned all 4,821 product records including draft/hidden items

Request:  GET /api/v1/products/search?q=' UNION SELECT username,password,NULL FROM users--
Response: 200 OK — returned hashed credentials for 1,203 user accounts
```

**Impact:**
An unauthenticated attacker can extract the entire database contents, including user credentials, PII, and payment references. Further exploitation could allow writing files to disk (if FILE privilege is granted to the DB user).

**Remediation:**
1. Replace string concatenation with parameterized queries / prepared statements immediately.
2. Apply principle of least privilege to the database account — remove FILE and DROP privileges.
3. Implement a WAF rule as a short-term compensating control.
4. Review all other database queries in the codebase for the same pattern.

**References:** OWASP SQL Injection Prevention Cheat Sheet

---

### CRIT-002 — Remote Code Execution via File Upload

- **Severity:** Critical
- **CVSS Score:** 9.0 (AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H)
- **CWE:** CWE-434 — Unrestricted Upload of File with Dangerous Type
- **Affected component:** `POST /api/v1/profile/avatar`

**Description:**
The avatar upload endpoint validates file type using the client-supplied `Content-Type` header only. An authenticated attacker can upload a PHP web shell by setting `Content-Type: image/jpeg` while submitting a `.php` file.

**Evidence:**
```
# Uploaded file: shell.php (Content-Type: image/jpeg)
# Accessed at: https://customer-portal.acme.com/uploads/avatars/shell.php?cmd=id
# Response: uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Impact:**
An authenticated attacker gains remote code execution as the web server user, enabling full server compromise, lateral movement, and data exfiltration.

**Remediation:**
1. Validate file type server-side using magic bytes, not MIME headers.
2. Store uploads outside the webroot or in object storage (S3/Blob).
3. Rename uploaded files to a random UUID — never preserve original filename.
4. Serve uploads through a dedicated media domain with no execution permissions.

---

### HIGH-001 — Broken Access Control (IDOR) on Invoice Endpoint

- **Severity:** High
- **CVSS Score:** 7.5
- **CWE:** CWE-639 — Authorization Bypass Through User-Controlled Key
- **Affected component:** `GET /api/v1/invoices/{id}`

**Description:**
Invoice IDs are sequential integers. An authenticated user can access invoices belonging to other customers by incrementing the ID parameter. No ownership check is performed server-side.

**Evidence:**
```
Authenticated as user_id=1042, accessed /api/v1/invoices/5001 through /api/v1/invoices/5100
→ Successfully retrieved 100 invoices belonging to other customers, including PII and amounts.
```

**Remediation:**
1. Implement server-side ownership validation: verify `invoice.customer_id == current_user.id` before returning data.
2. Consider switching to UUIDs for resource identifiers to remove enumeration.

---

## Remediation Roadmap

| Priority | Finding | Effort | Target Date |
|----------|---------|--------|-------------|
| Immediate | CRIT-001 SQL Injection | Medium | Within 48 hours |
| Immediate | CRIT-002 File Upload RCE | Low | Within 48 hours |
| This sprint | HIGH-001 IDOR on invoices | Low | Within 1 week |
| This sprint | HIGH-002 Missing rate limiting | Low | Within 1 week |
| Next sprint | MED-001 through MED-004 | Medium | Within 1 month |

---

## Appendix

**A. Testing Timeline**
- Day 1: Reconnaissance, endpoint mapping
- Day 2: Authentication and session testing
- Day 3: Business logic and access control
- Day 4: Input validation, injection testing
- Day 5: Report writing and validation

**B. Glossary**
- CVSS: Common Vulnerability Scoring System
- IDOR: Insecure Direct Object Reference
- RCE: Remote Code Execution

---
---

## Example 2 — Incident Response Report

---

# Incident Response Report
**Incident ID:** IR-2024-0042
**Incident Type:** Ransomware (LockBit 3.0)
**Affected Systems:** 14 Windows servers, 3 file shares
**Date Detected:** 2024-02-08 03:14 UTC
**Date Contained:** 2024-02-08 09:47 UTC
**Report Date:** 2024-02-15
**Classification:** CONFIDENTIAL

---

## Executive Summary

On February 8, 2024 at 03:14 UTC, the SOC detected mass file encryption activity across the ACME internal network. Investigation confirmed a LockBit 3.0 ransomware infection that originated from a phishing email received 6 days prior (February 2). The threat actor established persistence, harvested credentials, and moved laterally before deploying ransomware. 14 servers and 3 file shares were encrypted. No customer data was exfiltrated based on current evidence. Systems were restored from backup within 72 hours.

**Total business impact:** ~18 hours of downtime, estimated cost €85,000.

---

## Timeline

| Time (UTC) | Event |
|------------|-------|
| Feb 2, 09:31 | Phishing email received by finance employee, macro-enabled Word doc opened |
| Feb 2, 09:34 | Cobalt Strike beacon established from finance workstation |
| Feb 2–7 | Lateral movement, credential harvesting (Mimikatz), BloodHound AD enumeration |
| Feb 7, 22:00 | Attacker deploys LockBit 3.0 to 14 servers via PsExec |
| Feb 8, 03:14 | SOC alert: mass file rename activity detected |
| Feb 8, 03:22 | IR team engaged, network segmentation initiated |
| Feb 8, 04:45 | All affected systems isolated |
| Feb 8, 09:47 | Containment confirmed, eradication begins |
| Feb 9–11 | Systems restored from backup |
| Feb 11, 14:00 | Business operations fully restored |

---

## Root Cause Analysis

**Initial vector:** Phishing email with malicious Word document containing a VBA macro. The macro downloaded a Cobalt Strike stager from `hxxps://update-cdn[.]net/svc.exe`.

**Why it succeeded:**
1. Macro execution was not disabled by policy (GPO not enforced)
2. Email gateway did not sandbox macro-enabled Office documents
3. Endpoint protection did not detect the Cobalt Strike beacon (legitimate tool abuse)
4. Domain Admin credentials were reused across multiple systems, enabling rapid lateral movement

---

## Indicators of Compromise (IOCs)

| Type | Value | Context |
|------|-------|---------|
| Domain | update-cdn[.]net | C2 / stager download |
| IP | 185.220.101.47 | Cobalt Strike C2 |
| SHA256 | a3f1c2...d9e4 | LockBit 3.0 binary |
| Filename | svc.exe | Cobalt Strike stager |
| Registry key | HKCU\Software\Microsoft\svc | Persistence key |
| Ransom note | !!!-Restore-My-Files-!!!.txt | LockBit marker |

---

## Containment & Eradication Actions

1. Isolated all affected VLANs at the firewall level
2. Disabled compromised accounts (3 service accounts, 1 DA account)
3. Reset all Active Directory passwords (forced)
4. Removed persistence mechanisms (registry keys, scheduled tasks)
5. Rebuilt 14 servers from clean OS images
6. Restored data from last clean backup (Feb 7, 01:00 UTC — ~2 hours of data loss)
7. Blocked IOCs at firewall, email gateway, and DNS

---

## Recommendations

| Priority | Recommendation |
|----------|---------------|
| Critical | Disable VBA macros via GPO for all users; use Attack Surface Reduction rules |
| Critical | Enable email sandboxing for all Office attachments |
| High | Implement tiered AD model — separate DA credentials from daily use |
| High | Deploy EDR with behavioral detection on all endpoints |
| High | Enforce MFA for all privileged accounts and VPN access |
| Medium | Implement network segmentation between workstations and servers |
| Medium | Test backups quarterly with actual restore exercises |
| Low | Security awareness training refresh — phishing simulation |

---

## Lessons Learned

**What worked well:**
- SOC detection was fast (11 minutes from encryption start to alert)
- IR playbook was followed correctly
- Backups were intact and restorable

**What needs improvement:**
- No macro policy enforcement allowed initial compromise
- Lateral movement was undetected for 6 days
- Credential hygiene (DA account reuse) amplified impact

---
