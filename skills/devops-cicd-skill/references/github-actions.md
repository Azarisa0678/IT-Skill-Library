# GitHub Actions — Referenzmodul

Vollständige Workflows, OIDC, Matrix-Builds, Environments, Secrets, Caching, Reusable Workflows.

---

## OIDC — Keyless Auth zu Azure/AWS (kein gespeichertes Secret!)

```yaml
# Azure via OIDC (kein Service Principal Secret nötig)
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - run: az webapp deploy --name myapp --resource-group rg-prod

# AWS via OIDC
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
          aws-region: eu-central-1
```

---

## Matrix-Build (Multi-Version, Multi-Platform)

```yaml
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: ['18', '20', '22']
        exclude:
          - os: windows-latest
            node: '18'
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: npm
      - run: npm ci && npm test

  docker-multiarch:
    runs-on: ubuntu-latest
    steps:
      - uses: docker/setup-qemu-action@v3
      - uses: docker/setup-buildx-action@v3
      - uses: docker/build-push-action@v5
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

---

## Reusable Workflow (DRY-Prinzip)

```yaml
# .github/workflows/_deploy.yml (reusable)
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      image-tag:
        required: true
        type: string
    secrets:
      KUBE_CONFIG:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4
      - uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      - run: |
          helm upgrade --install myapp ./helm \
            --namespace ${{ inputs.environment }} \
            --set image.tag=${{ inputs.image-tag }} \
            --wait --atomic --timeout 10m

---
# Aufruf aus anderem Workflow
jobs:
  deploy-staging:
    uses: ./.github/workflows/_deploy.yml
    with:
      environment: staging
      image-tag: ${{ needs.build.outputs.image-tag }}
    secrets: inherit

  deploy-prod:
    needs: deploy-staging
    uses: ./.github/workflows/_deploy.yml
    with:
      environment: production
      image-tag: ${{ needs.build.outputs.image-tag }}
    secrets: inherit
```

---

## Vollständige DevSecOps-Pipeline

```yaml
name: DevSecOps Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE: ${{ github.repository }}

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # ── SAST + Secrets Scan ──────────────────────────────────────────────────
  sast:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - name: CodeQL Analysis
        uses: github/codeql-action/init@v3
        with:
          languages: javascript, python
      - uses: github/codeql-action/autobuild@v3
      - uses: github/codeql-action/analyze@v3

      - name: Gitleaks (Secrets)
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Semgrep
        uses: semgrep/semgrep-action@v1
        with:
          config: >-
            p/owasp-top-ten
            p/docker
            p/terraform

  # ── SCA (Abhängigkeiten) ─────────────────────────────────────────────────
  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: npm audit
        run: npm audit --audit-level=high
      - name: OWASP Dependency-Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: ${{ github.repository }}
          path: '.'
          format: SARIF
      - uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: reports/

  # ── Unit Tests ───────────────────────────────────────────────────────────
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: npm }
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage --coverageReporters=lcov
      - uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          fail_ci_if_error: true
          minimum_coverage: 80

  # ── Build + Image Scan ───────────────────────────────────────────────────
  build:
    needs: [sast, sca, test]
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      digest: ${{ steps.build.outputs.digest }}
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Image-Metadaten
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE }}
          tags: |
            type=sha,prefix=,format=long
            type=semver,pattern={{version}}
            type=ref,event=branch

      - name: Build + Push
        id: build
        uses: docker/build-push-action@v5
        with:
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: true
          sbom: true

      - name: Trivy Image-Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE }}:${{ github.sha }}
          severity: CRITICAL,HIGH
          exit-code: 1
          format: sarif
          output: trivy-results.sarif
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-results.sarif

      - name: Cosign — Image signieren
        uses: sigstore/cosign-installer@v3
      - run: |
          cosign sign --yes \
            ${{ env.REGISTRY }}/${{ env.IMAGE }}@${{ steps.build.outputs.digest }}

  # ── Deploy Staging ───────────────────────────────────────────────────────
  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    uses: ./.github/workflows/_deploy.yml
    with:
      environment: staging
      image-tag: ${{ github.sha }}
    secrets: inherit

  # ── DAST (Staging) ───────────────────────────────────────────────────────
  dast:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - uses: zaproxy/action-full-scan@v0.10.0
        with:
          target: https://staging.firma.de
          fail_action: true

  # ── Deploy Production ────────────────────────────────────────────────────
  deploy-prod:
    needs: [build, dast]
    if: github.event_name == 'release'
    uses: ./.github/workflows/_deploy.yml
    with:
      environment: production
      image-tag: ${{ github.sha }}
    secrets: inherit
```

---

## Caching-Strategien

```yaml
# npm Cache
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('package-lock.json') }}
    restore-keys: npm-

# Docker Layer Cache (GitHub Cache Backend)
- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max

# Terraform Provider Cache
- uses: actions/cache@v4
  with:
    path: ~/.terraform.d/plugin-cache
    key: terraform-${{ hashFiles('**/.terraform.lock.hcl') }}

# Go Module Cache
- uses: actions/cache@v4
  with:
    path: |
      ~/go/pkg/mod
      ~/.cache/go-build
    key: go-${{ hashFiles('**/go.sum') }}
```

---

## Self-Hosted Runner (Azure Container Instances)

```bash
# Runner als ACI — on-demand, kein permanenter Runner nötig
az container create \
  --resource-group rg-github-runners \
  --name github-runner-$(date +%s) \
  --image mycontainerregistry.azurecr.io/github-runner:latest \
  --environment-variables \
    GITHUB_TOKEN="$RUNNER_TOKEN" \
    GITHUB_OWNER="firma" \
    GITHUB_REPO="myrepo" \
  --cpu 2 --memory 4 \
  --restart-policy Never
```
