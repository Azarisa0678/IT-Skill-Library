# Cloud Security Reference

## Shared Responsibility Model

| Layer | AWS | Azure | GCP | Customer |
|-------|-----|-------|-----|----------|
| Physical infrastructure | ✅ | ✅ | ✅ | — |
| Hypervisor / compute | ✅ | ✅ | ✅ | — |
| Network controls | Shared | Shared | Shared | Shared |
| OS patching (IaaS) | — | — | — | ✅ |
| Application code | — | — | — | ✅ |
| Data classification | — | — | — | ✅ |
| IAM / access control | Shared | Shared | Shared | ✅ |
| Encryption (data) | Shared | Shared | Shared | ✅ |

**Key principle:** The cloud provider secures *of* the cloud; you secure *in* the cloud.

---

## AWS Security

### IAM Hardening
```bash
# Audit root account usage
aws iam get-account-summary | grep -i root

# Find users with console access but no MFA
aws iam generate-credential-report
aws iam get-credential-report --query 'Content' --output text | base64 -d | \
  awk -F',' 'NR>1 && $4=="true" && $8=="false" {print $1, "- Console access, NO MFA"}'

# Find overly permissive policies (AdministratorAccess attached to users)
aws iam list-policies --scope Local --query 'Policies[*].PolicyName'
aws iam list-entities-for-policy \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# Find access keys older than 90 days
aws iam generate-credential-report
aws iam get-credential-report --output text --query 'Content' | base64 -d | \
  awk -F',' 'NR>1 && $9!="N/A" {print $1, $9}'
```

### S3 Security Audit
```bash
# Find public buckets
aws s3api list-buckets --query 'Buckets[*].Name' --output text | tr '\t' '\n' | \
while read bucket; do
  acl=$(aws s3api get-bucket-acl --bucket "$bucket" 2>/dev/null)
  public=$(echo "$acl" | grep -c "AllUsers\|AuthenticatedUsers")
  [ "$public" -gt 0 ] && echo "PUBLIC: $bucket"
done

# Check bucket policies for public access
aws s3api get-bucket-policy --bucket BUCKET_NAME 2>/dev/null | \
  python3 -c "import sys,json; p=json.load(sys.stdin); \
  [print('PUBLIC STATEMENT:', s) for s in p['Statement'] if s.get('Principal')=='*']"

# Enable S3 Block Public Access at account level
aws s3control put-public-access-block \
  --account-id $(aws sts get-caller-identity --query Account --output text) \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

### CloudTrail & Logging
```bash
# Verify CloudTrail is enabled in all regions
aws cloudtrail describe-trails --include-shadow-trails \
  --query 'trailList[*].{Name:Name,Region:HomeRegion,MultiRegion:IsMultiRegionTrail,LogValidation:LogFileValidationEnabled}'

# Check if CloudTrail log validation is enabled
aws cloudtrail get-trail-status --name TRAIL_NAME \
  --query '{IsLogging:IsLogging,LatestDelivery:LatestDeliveryTime}'

# Enable GuardDuty in all regions
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  aws guardduty create-detector --enable --region "$region" 2>/dev/null && \
    echo "GuardDuty enabled: $region"
done
```

### Security Groups Audit
```bash
# Find security groups allowing 0.0.0.0/0 inbound
aws ec2 describe-security-groups \
  --filters Name=ip-permission.cidr,Values='0.0.0.0/0' \
  --query 'SecurityGroups[*].{ID:GroupId,Name:GroupName,Rules:IpPermissions}' \
  --output table

# Find unrestricted SSH/RDP
aws ec2 describe-security-groups \
  --filters "Name=ip-permission.from-port,Values=22,3389" \
             "Name=ip-permission.cidr,Values=0.0.0.0/0" \
  --query 'SecurityGroups[*].{ID:GroupId,Name:GroupName}'
```

### AWS Security Checklist
- [ ] Root account has MFA enabled and no access keys
- [ ] All IAM users have MFA enabled
- [ ] Password policy enforces minimum 14 characters + complexity
- [ ] Access keys rotated every 90 days
- [ ] No inline policies — use managed policies
- [ ] Least privilege applied to all roles
- [ ] CloudTrail enabled in all regions with log validation
- [ ] GuardDuty enabled in all regions
- [ ] AWS Config enabled and recording
- [ ] S3 Block Public Access enabled at account level
- [ ] Security Hub enabled and findings reviewed
- [ ] VPC Flow Logs enabled
- [ ] Default VPC deleted or not used for production
- [ ] EBS/RDS/S3 encryption enabled at rest
- [ ] Secrets in Secrets Manager, not environment variables or code

---

## Azure Security

### Entra ID (Azure AD) Audit
```powershell
Connect-MgGraph -Scopes "User.Read.All","Policy.Read.All","Directory.Read.All"

# Users without MFA
Get-MgReportAuthenticationMethodUserRegistrationDetail |
  Where-Object { $_.IsMfaRegistered -eq $false -and $_.IsEnabled -eq $true } |
  Select-Object UserPrincipalName, UserDisplayName

# Global Administrators
(Get-MgDirectoryRole | Where-Object DisplayName -eq 'Global Administrator') |
  ForEach-Object { Get-MgDirectoryRoleMember -DirectoryRoleId $_.Id } |
  Select-Object @{N='UPN';E={$_.AdditionalProperties.userPrincipalName}}

# Conditional Access Policies — find disabled or report-only
Get-MgIdentityConditionalAccessPolicy |
  Where-Object State -ne 'enabled' |
  Select-Object DisplayName, State
```

### Azure Security Audit (Az CLI)
```bash
# Check security defaults / conditional access
az ad policy list --query '[].displayName'

# Find storage accounts with public blob access
az storage account list \
  --query '[?allowBlobPublicAccess==`true`].{Name:name,RG:resourceGroup}' \
  --output table

# Find VMs without disk encryption
az vm list --query \
  '[?storageProfile.osDisk.encryptionSettings==null].{Name:name,RG:resourceGroup}' \
  --output table

# Check Key Vault soft delete
az keyvault list --query '[*].{Name:name,SoftDelete:properties.enableSoftDelete}' \
  --output table

# Enable Microsoft Defender for Cloud on subscription
az security pricing create --name VirtualMachines --tier Standard
az security pricing create --name SqlServers --tier Standard
az security pricing create --name StorageAccounts --tier Standard
```

### Azure Security Checklist
- [ ] MFA enforced via Conditional Access (not just security defaults)
- [ ] Privileged Identity Management (PIM) for admin roles
- [ ] No permanent Global Admins — use eligible assignments
- [ ] Conditional Access: block legacy authentication
- [ ] Conditional Access: require compliant device for sensitive apps
- [ ] Microsoft Defender for Cloud enabled (Standard tier)
- [ ] Azure Policy: enforce encryption, approved regions, allowed VM SKUs
- [ ] Storage accounts: disable public blob access, require HTTPS
- [ ] Key Vault: soft delete + purge protection enabled
- [ ] Log Analytics workspace connected; all diagnostic logs flowing
- [ ] Microsoft Sentinel (SIEM) configured with analytics rules
- [ ] JIT (Just-in-Time) VM access enabled
- [ ] Azure DDoS Protection Standard on production VNets
- [ ] Resource locks on production resource groups

---

## GCP Security

### IAM & Organization Audit
```bash
# Find project-level IAM bindings with primitive roles (Owner/Editor)
gcloud projects get-iam-policy PROJECT_ID \
  --format='table(bindings.role,bindings.members)' | \
  grep -E "roles/owner|roles/editor"

# Find service accounts with admin privileges
gcloud iam service-accounts list --format='value(email)' | while read sa; do
  gcloud projects get-iam-policy PROJECT_ID \
    --flatten='bindings[].members' \
    --filter="bindings.members:$sa" \
    --format='table(bindings.role)' 2>/dev/null | grep -v ROLE
done

# Check org policies
gcloud resource-manager org-policies list --organization=ORG_ID

# Find public GCS buckets
gsutil ls -p PROJECT_ID | while read bucket; do
  acl=$(gsutil iam get "$bucket" 2>/dev/null)
  echo "$acl" | grep -q "allUsers\|allAuthenticatedUsers" && echo "PUBLIC: $bucket"
done
```

### GCP Security Checklist
- [ ] Organization policy: restrict public IP on VMs
- [ ] Organization policy: disable service account key creation
- [ ] Organization policy: restrict resource locations to approved regions
- [ ] No user-managed service account keys (use Workload Identity instead)
- [ ] Service accounts follow least privilege (no primitive roles)
- [ ] Cloud Audit Logs: Admin Activity and Data Access logs enabled
- [ ] Security Command Center Premium enabled
- [ ] VPC Service Controls around sensitive APIs
- [ ] Binary Authorization for container deployments
- [ ] Customer-managed encryption keys (CMEK) for sensitive data
- [ ] Cloud Armor WAF on internet-facing load balancers

---

## Multi-Cloud Security Tools

| Tool | Purpose | Clouds |
|------|---------|--------|
| Prowler | Open-source CSPM, CIS benchmarks | AWS, Azure, GCP |
| ScoutSuite | Multi-cloud security auditing | AWS, Azure, GCP |
| Steampipe | SQL queries against cloud APIs | AWS, Azure, GCP |
| Trivy | Container + IaC scanning | All |
| Checkov | IaC security scanning (Terraform, CF) | All |
| tfsec | Terraform security scanning | All |
| Pacu | AWS exploitation framework (authorized) | AWS |
| CloudSploit | Cloud security scanning | AWS, Azure, GCP |

```bash
# Prowler — AWS CIS Benchmark
pip install prowler
prowler aws --compliance cis_1.5_aws

# ScoutSuite — multi-cloud audit
pip install scoutsuite
scout aws --report-dir ./scout-report

# Steampipe — SQL over cloud APIs
steampipe query "select account_id, region, title from aws_iam_user where mfa_enabled = false"
```
