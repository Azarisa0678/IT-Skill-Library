# GitLab CI & Azure DevOps — Referenzmodul

GitLab CI/CD Pipelines, Azure DevOps Pipelines (YAML), Agents, Environments, Variable Groups.

---

## GitLab CI — Vollständige DevSecOps-Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - test
  - security
  - build
  - deploy-staging
  - dast
  - deploy-prod

variables:
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  DOCKER_BUILDKIT: "1"

default:
  image: node:20-alpine
  interruptible: true
  retry:
    max: 2
    when: [runner_system_failure, stuck_or_timeout_failure]

# ── Validate ──────────────────────────────────────────────────────────────────
lint:
  stage: validate
  script:
    - npm ci --cache .npm --prefer-offline
    - npm run lint
  cache:
    key:
      files: [package-lock.json]
    paths: [.npm/]

# ── Test ──────────────────────────────────────────────────────────────────────
unit-test:
  stage: test
  script:
    - npm ci --cache .npm --prefer-offline
    - npm test -- --coverage --coverageReporters=cobertura
  cache:
    key:
      files: [package-lock.json]
    paths: [.npm/]
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
      junit: junit.xml
    expire_in: 1 week

# ── Security ──────────────────────────────────────────────────────────────────
sast:
  stage: security
  include:
    - template: Security/SAST.gitlab-ci.yml
  variables:
    SAST_EXCLUDED_PATHS: "spec,test,tests,tmp"

secret-detection:
  stage: security
  include:
    - template: Security/Secret-Detection.gitlab-ci.yml

dependency-scanning:
  stage: security
  include:
    - template: Security/Dependency-Scanning.gitlab-ci.yml

# ── Build ─────────────────────────────────────────────────────────────────────
build-image:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - |
      docker buildx create --use --driver docker-container
      docker buildx build \
        --platform linux/amd64,linux/arm64 \
        --cache-from type=registry,ref=$CI_REGISTRY_IMAGE:cache \
        --cache-to   type=registry,ref=$CI_REGISTRY_IMAGE:cache,mode=max \
        --build-arg  BUILDKIT_INLINE_CACHE=1 \
        --tag $IMAGE \
        --tag $CI_REGISTRY_IMAGE:latest \
        --push .
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

container-scanning:
  stage: build
  include:
    - template: Security/Container-Scanning.gitlab-ci.yml
  variables:
    CS_IMAGE: $IMAGE
  needs: [build-image]

# ── Deploy Staging ────────────────────────────────────────────────────────────
deploy-staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.firma.de
    on_stop: stop-staging
  script:
    - kubectl config use-context $KUBE_CONTEXT_STAGING
    - helm upgrade --install myapp ./helm
        --namespace staging
        --set image.tag=$CI_COMMIT_SHA
        --wait --atomic --timeout 10m
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

stop-staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  environment:
    name: staging
    action: stop
  script:
    - helm uninstall myapp --namespace staging
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual

# ── DAST ─────────────────────────────────────────────────────────────────────
dast:
  stage: dast
  include:
    - template: Security/DAST.gitlab-ci.yml
  variables:
    DAST_WEBSITE: https://staging.firma.de
    DAST_FULL_SCAN_ENABLED: "false"
  needs: [deploy-staging]

# ── Deploy Production ─────────────────────────────────────────────────────────
deploy-production:
  stage: deploy-prod
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://app.firma.de
  script:
    - kubectl config use-context $KUBE_CONTEXT_PROD
    - helm upgrade --install myapp ./helm
        --namespace production
        --values helm/values-prod.yaml
        --set image.tag=$CI_COMMIT_SHA
        --wait --atomic --timeout 15m
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
      when: manual
  needs: [dast, build-image]
```

### GitLab CI — Reusable Job Templates

```yaml
# .gitlab/ci/templates.yml — Wiederverwendbare Job-Definitionen
.docker-build-template: &docker-build
  image: docker:24
  services: [docker:24-dind]
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

.helm-deploy-template: &helm-deploy
  image: alpine/helm:3.14.0
  script:
    - helm upgrade --install $APP_NAME ./helm
        --namespace $NAMESPACE
        --set image.tag=$CI_COMMIT_SHA
        --values helm/values-${CI_ENVIRONMENT_NAME}.yaml
        --wait --atomic --timeout 10m

# Nutzung in .gitlab-ci.yml:
deploy-staging:
  <<: *helm-deploy
  environment: staging
  variables:
    APP_NAME: myapp
    NAMESPACE: staging
```

### GitLab CI — Merge Request Pipelines

```yaml
# Nur bei MR ausführen (schneller Feedback-Loop)
unit-test:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# Nur bei Tags deployen
deploy-prod:
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
      when: manual

# Scheduled Pipeline (z.B. nächtliche Security-Scans)
nightly-security-scan:
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
```

---

## Azure DevOps Pipelines (YAML)

### Vollständige Multi-Stage-Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include: [main, develop]
  paths:
    exclude: ['*.md', 'docs/**']

pr:
  branches:
    include: [main]

variables:
  - group: prod-variable-group      # Variable Group aus Azure DevOps Library
  - name: imageRepository
    value: myapp
  - name: containerRegistry
    value: acrfirmaprod.azurecr.io
  - name: dockerfilePath
    value: $(Build.SourcesDirectory)/Dockerfile
  - name: tag
    value: $(Build.SourceVersion)

pool:
  vmImage: ubuntu-latest

stages:
# ── Stage 1: CI ───────────────────────────────────────────────────────────────
- stage: CI
  displayName: Build & Test
  jobs:
  - job: Test
    displayName: Unit Tests
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '20.x'
    - script: |
        npm ci
        npm run lint
        npm test -- --coverage
      displayName: Test
    - task: PublishCodeCoverageResults@2
      inputs:
        summaryFileLocation: coverage/cobertura-coverage.xml
        failIfCoverageEmpty: true

  - job: Security
    displayName: Security Scan
    steps:
    - task: CredScan@3
      displayName: CredScan
    - task: WhiteSource@21
      displayName: SCA Scan
      inputs:
        cwd: $(Build.SourcesDirectory)

  - job: Build
    displayName: Docker Build & Push
    dependsOn: [Test, Security]
    steps:
    - task: AzureCLI@2
      displayName: ACR Login (OIDC)
      inputs:
        azureSubscription: sc-azure-prod
        scriptType: bash
        scriptLocation: inlineScript
        inlineScript: az acr login --name acrfirmaprod

    - task: Docker@2
      displayName: Build & Push
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: sc-acr-prod
        tags: |
          $(tag)
          latest

    - task: AquaSecurityTrivy@1
      displayName: Trivy Image Scan
      inputs:
        image: $(containerRegistry)/$(imageRepository):$(tag)
        exitCode: 1
        severity: CRITICAL,HIGH

# ── Stage 2: Staging ──────────────────────────────────────────────────────────
- stage: Staging
  displayName: Deploy Staging
  dependsOn: CI
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployStaging
    displayName: Deploy to Staging
    environment: staging
    strategy:
      runOnce:
        deploy:
          steps:
          - task: HelmDeploy@0
            displayName: Helm Deploy
            inputs:
              connectionType: Kubernetes Service Connection
              kubernetesServiceEndpoint: sc-k8s-staging
              namespace: staging
              command: upgrade
              chartType: FilePath
              chartPath: $(Build.SourcesDirectory)/helm
              releaseName: myapp
              overrideValues: image.tag=$(tag)
              arguments: --wait --atomic --timeout 10m

# ── Stage 3: Production ───────────────────────────────────────────────────────
- stage: Production
  displayName: Deploy Production
  dependsOn: Staging
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployProd
    displayName: Deploy to Production
    environment: production    # Azure DevOps Environment mit Approval Gate!
    strategy:
      runOnce:
        deploy:
          steps:
          - task: HelmDeploy@0
            displayName: Helm Deploy Production
            inputs:
              connectionType: Kubernetes Service Connection
              kubernetesServiceEndpoint: sc-k8s-prod
              namespace: production
              command: upgrade
              chartType: FilePath
              chartPath: $(Build.SourcesDirectory)/helm
              releaseName: myapp
              valueFile: helm/values-prod.yaml
              overrideValues: image.tag=$(tag)
              arguments: --wait --atomic --timeout 15m
```

### Azure DevOps — Variable Groups & Secrets

```yaml
# Library Variable Group in Pipeline nutzen
variables:
  - group: myapp-prod-secrets     # Key Vault-verknüpfte Variable Group

# Direkter Key Vault Task
steps:
- task: AzureKeyVault@2
  displayName: Load Secrets from Key Vault
  inputs:
    azureSubscription: sc-azure-prod
    KeyVaultName: kv-firma-prod
    SecretsFilter: 'db-password,api-key,smtp-password'
    RunAsPreJob: true

# Secret in nachfolgendem Step nutzen (automatisch maskiert in Logs)
- script: |
    echo "Connecting to database..."
    # db-password ist jetzt als Variable $(db-password) verfügbar
  env:
    DB_PASSWORD: $(db-password)  # Sicheres Mapping
```

### Azure DevOps — Approval Gates

```yaml
# Environment-Konfiguration in Azure DevOps UI:
# Pipelines → Environments → production → Approvals and checks
# → Add Approval → Approvers: [Platform-Team-Gruppe]
# → Timeout: 24h
# → Instructions: "Bitte Deployment-Checklist prüfen: ..."

# In Pipeline referenzieren
- deployment: DeployProd
  environment: production   # Wartet auf Approval, bevor Deployment startet
```

---

## Vergleich: GitHub Actions vs. GitLab CI vs. Azure DevOps

| Feature | GitHub Actions | GitLab CI | Azure DevOps |
|---|---|---|---|
| YAML-Format | `.github/workflows/*.yml` | `.gitlab-ci.yml` | `azure-pipelines.yml` |
| Reusable Workflows | `workflow_call` | Includes + Anchors | Templates + Task Groups |
| OIDC Keyless Auth | ✅ Nativ (Azure/AWS/GCP) | ✅ ab GitLab 15.7 | ✅ Azure Service Connection |
| Container Registry | GHCR (inkludiert) | GitLab Registry (inkl.) | ACR (extern) |
| Environments + Approval | ✅ Environment Rules | ✅ Protected Environments | ✅ Approval Gates |
| Matrix Builds | ✅ `strategy.matrix` | ✅ `parallel.matrix` | ✅ `strategy.matrix` |
| Self-Hosted Runner | ✅ `runs-on: self-hosted` | ✅ GitLab Runner | ✅ Agent Pool |
| Secrets Scanning | Gitleaks (externe Action) | ✅ Integriert (Template) | ✅ CredScan |
| Pricing | Kostenlos für OSS, Minuten-Modell | Kostenlos für OSS, CI-Minutes | Azure-Subscription |
| DE-Datenschutz | GitHub.com / GitHub Enterprise | GitLab.com / Self-Managed | Azure Cloud (EU-Region) |
