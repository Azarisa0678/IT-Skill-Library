# Phishing & Social Engineering

## Threat Overview

Social engineering exploits human psychology rather than technical vulnerabilities. It is the most common initial access vector in real-world attacks (used in 70–90% of breaches).

### Attack Categories

| Type | Description | Common Target |
|------|-------------|---------------|
| Spear Phishing | Targeted email with personalized content | Executives, Finance, IT |
| Whaling | Spear phishing targeting C-level | CEO, CFO, CISO |
| Vishing | Voice/phone-based deception | Helpdesk, HR |
| Smishing | SMS-based phishing | Mobile users |
| BEC | Business Email Compromise — impersonates trusted party | Finance, AP teams |
| Pretexting | Fabricated scenario to extract info | Any employee |
| Quid Pro Quo | Offering something in exchange for info | IT support targets |
| Watering Hole | Compromise sites the target visits | Industry-specific targets |

---

## Phishing Simulation Planning

### Pre-Simulation Checklist
- [ ] Written authorization from senior management / legal
- [ ] Scope defined: which departments, email domains, time window
- [ ] HR and legal informed (to avoid false disciplinary actions)
- [ ] IT / SOC informed (to avoid false incident response)
- [ ] Baseline metric defined (current click rate if known)
- [ ] Follow-up training prepared before simulation runs
- [ ] Tool configured: GoPhish, Cofense, KnowBe4, or similar

### Simulation Difficulty Levels

| Level | Characteristics | Example |
|-------|----------------|---------|
| Easy | Generic sender, obvious red flags | "Your account will be suspended" from random domain |
| Medium | Plausible sender, reasonable pretext | IT helpdesk password reset |
| Hard | Targeted, personalized, branded | CEO requesting urgent wire transfer |

### Recommended Metrics to Track

| Metric | Description | Target |
|--------|-------------|--------|
| Open rate | % of users who opened the email | Baseline only |
| Click rate | % who clicked the link | < 5% after training |
| Credential submission rate | % who entered credentials | 0% |
| Report rate | % who reported the email | > 70% |
| Time to report | How quickly phishing is reported | < 1 hour |

---

## Phishing Email Indicators — Awareness Training Content

Teach users to look for:

**Sender / Domain:**
- Domain lookalikes: `acme-support.com`, `acme.com.login-portal.net`
- Display name spoofing: "IT Support" with a Gmail address
- Slight typos: `micros0ft.com`, `paypa1.com`

**Urgency & Pressure:**
- "Your account will be locked in 24 hours"
- "Immediate action required"
- "Confidential — do not forward"

**Links:**
- URL doesn't match display text — hover before clicking
- Shortened URLs (bit.ly, tinyurl) hiding the destination
- HTTP instead of HTTPS

**Attachments:**
- Macro-enabled Office files (.docm, .xlsm)
- Password-protected ZIPs (bypass email scanners)
- ISO / IMG files (bypass Mark-of-the-Web)

**Content:**
- Generic greeting ("Dear Customer")
- Grammar/spelling errors (though AI has reduced this signal)
- Requests for credentials, wire transfers, gift cards

---

## Phishing Reporting Procedure Template

*Customize this for your organization:*

```
PHISHING REPORT PROCEDURE
--------------------------
If you receive a suspicious email:

1. DO NOT click any links or open attachments
2. DO NOT reply to the sender
3. Report using one of these methods:
   - Click the "Report Phishing" button in Outlook
   - Forward to: phishing@yourcompany.com
   - Call the IT Helpdesk: +49 xxx xxx xxx

4. If you DID click a link or enter credentials:
   - Call the IT Helpdesk IMMEDIATELY: +49 xxx xxx xxx
   - Do not wait — early reporting limits damage

The IT Security team will investigate all reports within 4 business hours.
You will never be penalized for reporting a legitimate email in error.
```

---

## BEC (Business Email Compromise) Prevention

BEC causes more financial damage than any other cybercrime category (FBI IC3).

### Common BEC Scenarios

| Scenario | Description |
|----------|-------------|
| CEO Fraud | Attacker impersonates CEO, requests urgent wire transfer to finance |
| Invoice Fraud | Attacker impersonates supplier, requests change of bank account details |
| Payroll Diversion | Attacker impersonates employee, requests payroll redirect |
| Attorney Impersonation | Fake lawyer requests confidential transfer for "pending deal" |

### Technical Controls
- [ ] Enable DMARC (`p=reject`), DKIM, and SPF on all sending domains
- [ ] External email banners: tag emails from outside the organization
- [ ] Block lookalike domains: register common typosquats of your domain
- [ ] MFA on all email accounts (prevents account takeover)
- [ ] Email rules: alert on auto-forwarding rules created by users

### Process Controls
- [ ] Dual-authorization for all wire transfers above threshold (e.g. €5,000)
- [ ] Verify bank account changes via phone callback to known number (not from the email)
- [ ] Never process urgent financial requests via email alone
- [ ] Out-of-band verification for C-level payment requests

---

## Security Awareness Training Program

### Recommended Curriculum (Annual)

| Module | Audience | Frequency | Duration |
|--------|----------|-----------|----------|
| Phishing basics + simulation | All staff | Quarterly sim, annual training | 20 min |
| Password hygiene + MFA | All staff | Annual | 15 min |
| Data handling & classification | All staff | Annual | 20 min |
| BEC & wire fraud | Finance, AP, HR | Annual | 20 min |
| Secure remote work | Remote workers | Annual | 15 min |
| Social engineering (vishing) | Helpdesk, Receptionists | Annual | 20 min |
| Secure development | Developers | Annual | 60 min |
| Incident reporting | All staff | Annual | 10 min |

### Training Effectiveness Metrics

Track year-over-year:
- Phishing simulation click rate trend
- Phishing report rate trend
- Number of real incidents attributed to social engineering
- Training completion rate (target: 95%+)

---

## Vishing (Voice Phishing) Playbook

### Common Pretexts
- IT helpdesk: "We detected suspicious activity on your account, I need to verify your identity"
- Vendor: "Your payment is overdue, please confirm your bank details"
- Auditor: "I'm conducting a compliance review, what systems do you use?"
- Executive assistant: "The CEO needs this handled urgently while he's travelling"

### Verification Procedure (for Helpdesk)
Train helpdesk staff to:
1. Never reset passwords or grant access based on caller identity alone
2. Always verify via second channel (manager approval, email from corporate account)
3. Use a callback procedure: hang up, look up the official number, call back
4. Log all identity verification steps in the ticket

### Red Flags During a Call
- Caller creates urgency or pressure
- Caller discourages following standard procedure ("we don't have time for that")
- Caller has unusual knowledge of internal systems (may have done OSINT)
- Request involves credentials, access grants, or financial actions

---

## Incident Response — Phishing Victim

If a user reports clicking a phishing link or submitting credentials:

```
IMMEDIATE (within 1 hour):
[ ] Isolate the affected workstation from the network
[ ] Reset the user's password and revoke all active sessions/tokens
[ ] Check for OAuth app grants / email forwarding rules added
[ ] Review email sent from the account in the last 24 hours
[ ] Search email gateway logs for same phishing email to other recipients

SHORT-TERM (within 24 hours):
[ ] Analyse the phishing site (urlscan.io, VirusTotal)
[ ] Check if credentials were used (sign-in logs, impossible travel)
[ ] Block phishing domain/IP at email gateway and firewall
[ ] Notify other potential recipients with a user advisory
[ ] If BEC suspected: alert Finance to hold any pending transfers

FOLLOW-UP:
[ ] Root cause documented
[ ] User debrief (non-punitive)
[ ] IOCs added to threat intel feeds
[ ] Lessons learned fed back into training program
```
