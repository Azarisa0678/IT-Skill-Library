---
name: devops-cicd-skill
description: >
  DevOps und CI/CD Skill fuer DevOps-Engineers und Platform-Engineers.
  GitHub Actions, OIDC, Matrix, Reusable Workflows, DevSecOps-Pipeline,
  GitLab CI, Azure DevOps, Jenkins, Terraform, Bicep, Pulumi, IaC,
  Docker, Kubernetes, Helm, ArgoCD, Flux, GitOps, App-of-Apps, Kustomize,
  Blue-Green, Canary, Argo Rollouts, KEDA, HPA, PDB, Network Policy,
  Sealed Secrets, External Secrets, ESO, Prometheus, Grafana, Loki, OpenTelemetry,
  Alertmanager, SRE, SLO, SLI, Error Budget, Burn Rate, Postmortem, Incident,
  Supply Chain Security, SBOM, SLSA, Cosign, Sigstore, Syft, CycloneDX, SPDX,
  Dependency-Track, OpenSSF Scorecard, Trivy, Image Signing, Provenance,
  Secrets Management, OIDC Keyless Auth, Container, Docker Buildx, Multi-Arch.
  Outputs: Pipeline YAML, Terraform HCL, Helm Charts, Kubernetes Manifests, Runbooks.
---

# DevOps / CI/CD Skill

## Guiding Principles

**Everything as Code.** Pipelines, Infrastruktur, Config, Policies — alles in Git, versioniert, reviewed.
**Shift Left.** Security, Tests, Qualität so früh wie möglich — nicht erst vor dem Release.
**Fast Feedback.** Kritischer Pfad der Pipeline < 10 Minuten. Langsame Pipelines werden umgangen.
**Immutable Artifacts.** Einmal gebaut, überall deployt. Kein Bauen auf Produktion.
**GitOps.** Produktions-State deklarativ in Git — kein manuelles kubectl auf Prod.
**No Secret Hardcoding.** Niemals Credentials in Code/YAML. OIDC, Sealed Secrets, ESO.

---

## Referenzmodule

Lade das passende Modul je nach Thema:

- `references/github-actions.md` — OIDC, Matrix, Reusable Workflows, DevSecOps-Pipeline, Caching, Self-Hosted Runner
- `references/terraform-iac.md` — Projekt-Struktur, Module, State, CI/CD Pipeline, Checkov, Testing
- `references/kubernetes-helm.md` — Production-Helm-Chart, Blue-Green, Canary, HPA, KEDA, PDB, Network Policies, Kustomize
- `references/gitops.md` — ArgoCD App-of-Apps, Flux Image Automation, Sealed Secrets, ESO, Multi-Cluster
- `references/observability.md` — Prometheus Alerts, Alertmanager, OpenTelemetry, Grafana, Loki LogQL
- `references/sre-practices.md` — SLO-Definition, Error Budget Policy, Burn-Rate Alerts, Incident-Checkliste, Postmortem-Vorlage

Routing:
- GitHub Actions / CI-Pipeline / DevSecOps → `github-actions.md`
- Terraform / Bicep / IaC → `terraform-iac.md`
- Helm / Kubernetes / Release-Strategie / HPA / KEDA → `kubernetes-helm.md`
- ArgoCD / Flux / GitOps / Sealed Secrets / ESO → `gitops.md`
- Prometheus / Grafana / Alerting / OpenTelemetry / Loki → `observability.md`
- SLO / SLI / Error Budget / Incident / Postmortem → `sre-practices.md`
- Mehrere Themen → mehrere Module laden

---

## GitHub Actions — Schnellreferenz

```yaml
# OIDC Keyless Auth zu Azure (kein gespeichertes Secret!)
permissions:
  id-token: write
  contents: read
steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

# Reusable Workflow aufrufen
jobs:
  deploy:
    uses: ./.github/workflows/_deploy.yml
    with:
      environment: production
      image-tag: ${{ github.sha }}
    secrets: inherit

# Matrix-Build
strategy:
  fail-fast: false
  matrix:
    node: ['18', '20', '22']
    os: [ubuntu-latest, windows-latest]
```

→ Vollständige DevSecOps-Pipeline mit SAST/SCA/Trivy/Cosign/DAST: `references/github-actions.md`

---

## Terraform — Schnellreferenz

```hcl
# Produktionsreifes Azure Backend + OIDC
terraform {
  required_version = ">= 1.7.0"
  required_providers {
    azurerm = { source = "hashicorp/azurerm", version = "~> 3.100" }
  }
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformfirmaprod"
    container_name       = "tfstate"
    key                  = "prod/terraform.tfstate"
    use_oidc             = true
  }
}

# Lifecycle für Prod-Ressourcen
resource "azurerm_resource_group" "main" {
  name     = "rg-${var.project}-${var.environment}-we"
  location = var.location
  tags     = local.common_tags
  lifecycle { prevent_destroy = true }
}
```

```bash
# CI/CD Terraform-Pipeline Schritte
terraform init
terraform validate && terraform fmt -check
tflint --init && tflint --recursive
checkov -d . --framework terraform --soft-fail false
terraform plan -out=tfplan -detailed-exitcode
terraform apply -auto-approve tfplan   # nur main-Branch, nach Approval
```

→ AKS-Cluster-Modul, vollständige CI/CD-Pipeline mit Plan-PR-Kommentar, Testing: `references/terraform-iac.md`

---

## Kubernetes/Helm — Schnellreferenz

```bash
# Production-Deploy mit automatischem Rollback
helm upgrade --install myapp ./myapp \
  --namespace production --create-namespace \
  --values values-prod.yaml \
  --set image.tag=$IMAGE_TAG \
  --wait --atomic --timeout 10m

# Diff vor Upgrade (helm-diff Plugin)
helm diff upgrade myapp ./myapp -f values-prod.yaml --set image.tag=$IMAGE_TAG

# Rollback
helm rollback myapp 0 -n production   # 0 = vorherige Version
```

```yaml
# Essentials jedes Production-Deployments:
# ✅ securityContext: runAsNonRoot, readOnlyRootFilesystem, capabilities.drop: ALL
# ✅ resources.requests + limits gesetzt
# ✅ liveness + readiness + startupProbe
# ✅ PodDisruptionBudget: minAvailable: 1
# ✅ topologySpreadConstraints (Multi-Node-Verteilung)
# ✅ NetworkPolicy: Default Deny, nur explizite Erlaubnisse
# ✅ HPA oder KEDA für Auto-Scaling
```

→ Vollständiges Helm-Chart, Blue-Green, Canary mit Argo Rollouts, KEDA: `references/kubernetes-helm.md`

---

## GitOps — Schnellreferenz

```yaml
# ArgoCD Application (Auto-Sync + SelfHeal)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: production
  source:
    repoURL: https://github.com/firma/k8s-configs
    targetRevision: main
    path: apps/myapp/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions: [CreateNamespace=true, ServerSideApply=true]
```

```bash
# Flux bootstrap (GitHub)
flux bootstrap github \
  --owner=firma --repository=k8s-configs \
  --branch=main --path=clusters/prod \
  --components-extra=image-reflector-controller,image-automation-controller
```

→ App-of-Apps, Flux Image Automation, Sealed Secrets, ESO Azure Key Vault, Multi-Cluster: `references/gitops.md`

---

## Observability — Schnellreferenz

```yaml
# Kritischer SLO-Alert: 2% Error-Budget in 1h verbraucht
- alert: SLOBurnRate_Critical
  expr: |
    error_rate_1h > (14.4 * (1 - 0.999))
    and error_rate_5m > (14.4 * (1 - 0.999))
  labels:
    severity: page
  annotations:
    summary: "SLO-Budget verbraucht in ~1h"

# Grundlegende Metriken-Tripel
# Rate:   sum(rate(http_requests_total[5m])) by (service)
# Errors: sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
# P99:    histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))
```

→ Vollständige Prometheus-Config, Alertmanager-Routing, OTel Collector, Loki LogQL: `references/observability.md`

---

## SRE — Schnellreferenz

```markdown
## SLO-Definition Vorlage
Service: [Name]
SLI:     Anteil erfolgreicher HTTP-Anfragen (status != 5xx)
SLO:     99.9% über 30 Tage
Error Budget: 43.8 Minuten/Monat

## Severity-Skala
SEV-1: Produktionsausfall, alle Nutzer → Sofort (< 5 Min), Status-Page
SEV-2: >10% Nutzer betroffen → < 15 Min
SEV-3: Workaround vorhanden → < 1 Std
SEV-4: Kosmetisch → Nächster Werktag
```

→ Error Budget Policy, Burn-Rate Alerts, Incident-Checkliste, Postmortem-Vorlage: `references/sre-practices.md`

---

## Output-Formate je Aufgabentyp

| Aufgabe | Format |
|---------|--------|
| GitHub Actions Workflow | Vollständiges YAML mit allen Jobs (test/security/build/deploy) |
| Terraform Modul | HCL mit variables.tf, outputs.tf, README-Hinweis |
| Helm Chart | Vollständige template/-Dateien mit _helpers.tpl |
| ArgoCD/Flux Manifest | Fertige YAML-Manifeste, deployment-ready |
| Prometheus Alert | alert + expr + for + labels + annotations mit runbook_url |
| Postmortem | Vollständige Vorlage mit Timeline, RCA, Action Items |
| Runbook | Schritt-für-Schritt mit Entscheidungsbäumen und Rollback-Schritten |
| Architektur | ASCII-Diagramm + Erklärung der Komponenten und Datenflüsse |

---

## GitLab CI & Azure DevOps — Schnellreferenz

```yaml
# GitLab CI — Grundstruktur mit Templates
stages: [validate, test, security, build, deploy-staging, deploy-prod]

include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml

# Job nur bei MR ausführen
unit-test:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# Deploy nur bei Tag
deploy-prod:
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
      when: manual
```

```yaml
# Azure DevOps — Multi-Stage mit Approval Gate
stages:
  - stage: CI
    jobs: [Test, Security, Build]
  - stage: Staging
    dependsOn: CI
    jobs:
      - deployment: DeployStaging
        environment: staging        # Protected Environment
  - stage: Production
    dependsOn: Staging
    jobs:
      - deployment: DeployProd
        environment: production     # Wartet auf manuelle Freigabe!
```

→ Vollständige Pipelines, Variable Groups, Key Vault Integration, Vergleichstabelle: `references/gitlab-azdo.md`

---

## Docker — Best Practices

```dockerfile
# Multi-Stage Build — produktionsreif
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS runtime
# Non-root User
RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -s /bin/sh -D appuser
WORKDIR /app
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --chown=appuser:appgroup src/ ./src/
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/healthz || exit 1
ENTRYPOINT ["node", "src/index.js"]
```

```bash
# Multi-Arch Build (AMD64 + ARM64)
docker buildx create --use --driver docker-container
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --cache-from type=registry,ref=ghcr.io/firma/myapp:cache \
  --cache-to   type=registry,ref=ghcr.io/firma/myapp:cache,mode=max \
  --tag ghcr.io/firma/myapp:$SHA \
  --push .

# Image-Scan mit Trivy
trivy image --severity CRITICAL,HIGH --exit-code 1 ghcr.io/firma/myapp:$SHA

# Image signieren mit Cosign (OIDC, kein Key nötig)
cosign sign --yes ghcr.io/firma/myapp@$DIGEST
```

---

## Supply Chain Security

```yaml
# SBOM generieren (Syft) + in Registry pushen
- name: SBOM generieren
  run: |
    syft ghcr.io/${{ github.repository }}:${{ github.sha }} \
      -o cyclonedx-json > sbom.json
    cosign attach sbom \
      --sbom sbom.json \
      ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}

# SLSA Provenance (automatisch via docker/build-push-action)
- uses: docker/build-push-action@v5
  with:
    provenance: true   # SLSA Level 2 Provenance
    sbom: true         # SBOM generieren und anhängen

# Cosign verify vor Deploy (in CD-Pipeline)
- name: Image-Signatur verifizieren
  run: |
    cosign verify \
      --certificate-identity-regexp="https://github.com/firma/.*" \
      --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
      ghcr.io/${{ github.repository }}@${{ inputs.digest }}
```

---

## Eval-Checkliste (Qualitätssicherung)

Beim Ausgeben von DevOps-Inhalten prüfen:

**CI/CD-Pipelines:**
- [ ] OIDC statt statische Secrets (kein `password:` im YAML)
- [ ] `concurrency` mit `cancel-in-progress: true` in GitHub Actions
- [ ] `npm ci` statt `npm install`
- [ ] Docker-Cache eingebaut (`type=gha` oder Registry-Cache)
- [ ] Security-Scan (SAST + SCA + Container) in Pipeline
- [ ] Deploy nur auf main/Tag, nicht auf Feature-Branches

**Terraform:**
- [ ] Remote Backend konfiguriert
- [ ] `required_version` + `required_providers` mit Versions-Pin
- [ ] `sensitive = true` für Passwort-Outputs
- [ ] `lifecycle { prevent_destroy = true }` für Prod-Ressourcen
- [ ] `local.common_tags` Tagging-Strategie

**Kubernetes/Helm:**
- [ ] `securityContext.runAsNonRoot: true`
- [ ] `readOnlyRootFilesystem: true`
- [ ] `resources.requests` und `limits` gesetzt
- [ ] `livenessProbe` + `readinessProbe` + `startupProbe`
- [ ] `PodDisruptionBudget` vorhanden
- [ ] `NetworkPolicy` vorhanden

**SRE/Observability:**
- [ ] SLI als PromQL-Ausdruck formuliert
- [ ] Error Budget numerisch berechnet
- [ ] Burn-Rate-Alert mit Multi-Window
- [ ] Postmortem: blameless, mit Action Items + Fristen

→ Vollständige Eval-Testfälle mit Kriterien: `references/evals.md`

---

## Supply Chain Security — Schnellreferenz

```bash
# SBOM generieren (Syft)
syft ghcr.io/firma/myapp:1.5.0 -o cyclonedx-json > sbom.json

# SBOM an Image attachen + attestieren (Cosign OIDC)
cosign attach sbom --sbom sbom.json --type cyclonedx \
    ghcr.io/firma/myapp@$DIGEST
cosign attest --yes --predicate sbom.json --type cyclonedx \
    ghcr.io/firma/myapp@$DIGEST

# Image signieren (keyless, GitHub Actions)
cosign sign --yes ghcr.io/firma/myapp@$DIGEST

# Signatur verifizieren (vor Deploy)
cosign verify \
    --certificate-identity-regexp="https://github.com/firma/.*" \
    --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
    ghcr.io/firma/myapp:1.5.0

# Vulnerability-Scan des SBOM
grype sbom:./sbom.json --fail-on critical
```

```yaml
# GitHub Actions: SLSA Provenance + SBOM + Signing in einem Job
- uses: docker/build-push-action@v5
  with:
    provenance: mode=max    # SLSA L2
    sbom: true              # Automatisches SBOM
    push: true
- uses: actions/attest-build-provenance@v1
  with:
    subject-name: ghcr.io/${{ github.repository }}
    subject-digest: ${{ steps.build.outputs.digest }}
    push-to-registry: true
```

→ SBOM-Formate (CycloneDX vs. SPDX), SLSA-Level, Dependency-Track, Scorecard, Kubernetes Policy Controller, vollständige Compliance-Checkliste: `references/supply-chain-security.md`
