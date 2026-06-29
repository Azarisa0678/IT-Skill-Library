# Zero Trust Architecture

## Core Principles (NIST SP 800-207)

Zero Trust is a security model based on the principle **"never trust, always verify"** — no user, device, or network segment is trusted by default, even inside the perimeter.

### The 7 Zero Trust Tenets (NIST)

| Tenet | Description |
|-------|-------------|
| 1. All data sources and services are resources | Every device, service, and data store is a resource regardless of location |
| 2. All communication is secured | Regardless of network location — encrypt everything |
| 3. Access per session | Access granted per-request, not persistent |
| 4. Access policy is dynamic | Based on identity, device state, behavior, and context |
| 5. Monitor all assets | Continuous monitoring of all owned and associated devices |
| 6. Authentication and authorization are dynamic | Continuously re-evaluated; not set-and-forget |
| 7. Collect telemetry | Use it to improve security posture iteratively |

---

## Zero Trust Pillars

### 1. Identity
- Strong MFA for all users (phishing-resistant: FIDO2/Passkeys preferred)
- Privileged Identity Management (PIM) — just-in-time access
- Continuous access evaluation (CAE) — revoke sessions on risk signals
- Service accounts: managed identities, workload identity federation

**Tools:** Microsoft Entra ID, Okta, Ping Identity, HashiCorp Vault (for workloads)

### 2. Devices
- Device health checked before granting access
- MDM/MAM enrollment required (Intune, Jamf)
- Compliance policies: OS patch level, AV status, disk encryption
- Certificate-based device identity

**Tools:** Microsoft Intune, Jamf, CrowdStrike, SentinelOne

### 3. Network
- Micro-segmentation: workloads only talk to what they need
- Replace VPN with ZTNA (Zero Trust Network Access)
- Software-defined perimeter (SDP)
- DNS filtering and egress control

**Tools:** Cloudflare Access, Zscaler ZPA, Palo Alto Prisma Access, Tailscale

### 4. Applications
- Per-application access policies
- Inline CASB for SaaS apps
- Application-layer inspection
- OAuth app governance (consent policies)

### 5. Data
- Data classification (Public → Internal → Confidential → Restricted)
- DLP policies tied to classification
- Encryption at rest and in transit — always
- Rights Management / Information Protection (AIP/MIP)

### 6. Visibility & Analytics (SOC)
- Unified log collection (SIEM)
- UEBA (User and Entity Behavior Analytics)
- XDR across identity, endpoint, network, cloud
- Automated response playbooks (SOAR)

---

## Zero Trust Maturity Model (CISA)

| Stage | Description |
|-------|-------------|
| **Traditional** | Static perimeter, implicit trust inside network |
| **Initial** | Some MFA, basic segmentation, manual processes |
| **Advanced** | Identity-centric, dynamic policies, partial automation |
| **Optimal** | Fully automated, continuous verification, AI-driven analytics |

---

## Implementation Roadmap

### Phase 1 — Identity First (Months 1–3)
```
[ ] Deploy MFA for all users (start with admins)
[ ] Implement Conditional Access policies
[ ] Enable PIM for privileged roles
[ ] Deploy phishing-resistant auth (FIDO2) for high-risk users
[ ] Inventory all service accounts and managed identities
```

### Phase 2 — Device Trust (Months 3–6)
```
[ ] Enroll all devices in MDM
[ ] Define and enforce compliance policies
[ ] Block non-compliant devices from accessing corporate resources
[ ] Deploy EDR on all endpoints
[ ] Enable certificate-based device authentication
```

### Phase 3 — Network (Months 6–9)
```
[ ] Deploy ZTNA to replace legacy VPN
[ ] Implement micro-segmentation for critical workloads
[ ] Enable DNS filtering and egress inspection
[ ] Remove implicit trust between network segments
[ ] Deploy CASB for SaaS governance
```

### Phase 4 — Data & Applications (Months 9–12)
```
[ ] Implement data classification framework
[ ] Deploy DLP policies for sensitive data
[ ] Apply per-app access policies
[ ] Govern OAuth app consent (block risky apps)
[ ] Enable Information Protection labels
```

### Phase 5 — Automation & Optimization (Ongoing)
```
[ ] Integrate SIEM + SOAR for automated response
[ ] Enable UEBA for insider threat detection
[ ] Implement XDR across all pillars
[ ] Regular Zero Trust maturity assessment
[ ] Red team exercises against Zero Trust controls
```

---

## Microsoft Zero Trust (Entra + Defender + Intune)

```powershell
# Conditional Access — require MFA + compliant device for all apps
# (via Microsoft Graph PowerShell)
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess"

$policy = @{
    displayName = "ZT-Require-MFA-CompliantDevice"
    state = "enabled"
    conditions = @{
        users = @{ includeUsers = @("All") }
        applications = @{ includeApplications = @("All") }
        clientAppTypes = @("all")
    }
    grantControls = @{
        operator = "AND"
        builtInControls = @("mfa", "compliantDevice")
    }
}
New-MgIdentityConditionalAccessPolicy -BodyParameter $policy

# Check PIM eligible assignments
Get-MgRoleManagementDirectoryRoleEligibilitySchedule -All |
    Select-Object PrincipalId, RoleDefinitionId, StartDateTime, EndDateTime |
    Format-Table -AutoSize

# Enable Continuous Access Evaluation
# (done at tenant level via Entra ID settings)
Update-MgPolicyAuthorizationPolicy -DefaultUserRolePermissions @{
    AllowedToCreateApps = $false
    AllowedToCreateSecurityGroups = $false
}
```

---

## Cloudflare Zero Trust (Access + Gateway)

```bash
# Deploy Cloudflare Tunnel (replaces VPN)
cloudflared tunnel create my-app-tunnel
cloudflared tunnel route dns my-app-tunnel app.company.com

# Configure Access Policy (via API)
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/access/apps" \
  -H "Authorization: Bearer {api_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Internal App",
    "domain": "app.company.com",
    "type": "self_hosted",
    "session_duration": "8h",
    "allowed_idps": ["google-workspace"],
    "policies": [{
      "name": "Allow employees",
      "decision": "allow",
      "include": [{"email_domain": {"domain": "company.com"}}]
    }]
  }'
```

---

## Zero Trust Checklist

- [ ] No implicit trust based on network location
- [ ] MFA enforced for all users — phishing-resistant for admins
- [ ] All devices verified before access (MDM compliance)
- [ ] Least-privilege access — no standing admin rights
- [ ] All traffic encrypted — no plaintext internal communication
- [ ] Micro-segmentation applied to critical workloads
- [ ] Continuous monitoring with behavioral analytics
- [ ] Automated response to identity/device risk signals
- [ ] Regular access reviews (quarterly minimum)
- [ ] Zero Trust maturity assessed annually
