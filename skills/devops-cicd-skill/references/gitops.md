# GitOps — Referenzmodul

ArgoCD, Flux v2, App-of-Apps, Multi-Cluster, Image Automation, Secrets mit Sealed Secrets / ESO.

---

## ArgoCD — Vollständige Konfiguration

### App-of-Apps Pattern (Management-Cluster)

```yaml
# apps/root-app.yaml — steuert alle anderen Apps
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/firma/k8s-configs
    targetRevision: main
    path: clusters/prod/apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

```yaml
# clusters/prod/apps/myapp.yaml
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
    syncOptions:
      - CreateNamespace=true
      - ServerSideApply=true
      - PrunePropagationPolicy=foreground
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas    # HPA steuert Replicas
```

### ArgoCD AppProject (RBAC)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production Applications
  sourceRepos:
    - https://github.com/firma/k8s-configs
    - https://charts.firma.de/*
  destinations:
    - namespace: production
      server: https://kubernetes.default.svc
    - namespace: monitoring
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
  namespaceResourceBlacklist:
    - group: ''
      kind: ResourceQuota
  roles:
    - name: developer
      description: Read-only für Developer
      policies:
        - p, proj:production:developer, applications, get, production/*, allow
        - p, proj:production:developer, applications, sync, production/*, deny
      groups:
        - firma:developers
    - name: deployer
      policies:
        - p, proj:production:deployer, applications, *, production/*, allow
      groups:
        - firma:platform-team
```

---

## Flux v2 — Vollständige Installation und Konfiguration

```bash
# Flux CLI installieren
curl -s https://fluxcd.io/install.sh | sudo bash

# Bootstrap (GitHub)
flux bootstrap github \
  --owner=firma \
  --repository=k8s-configs \
  --branch=main \
  --path=clusters/prod \
  --personal=false \
  --token-auth=false \   # OIDC nutzen
  --components-extra=image-reflector-controller,image-automation-controller
```

```yaml
# clusters/prod/flux-system/gotk-sync.yaml (auto-generiert)
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m
  ref:
    branch: main
  url: ssh://git@github.com/firma/k8s-configs
  secretRef:
    name: flux-system

---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 10m
  path: ./clusters/prod
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
  wait: true
  timeout: 5m
```

### Flux Image Automation (automatisches Image-Tag-Update)

```yaml
# Image-Repository überwachen
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: myapp
  namespace: flux-system
spec:
  image: ghcr.io/firma/myapp
  interval: 5m
  secretRef:
    name: ghcr-auth

---
# Image-Policy: latest semver in main
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: myapp
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: myapp
  policy:
    semver:
      range: '>=1.0.0'

---
# Image-Update: automatisch PR/Commit erstellen
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 30m
  sourceRef:
    kind: GitRepository
    name: flux-system
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: fluxbot@firma.de
        name: FluxBot
      messageTemplate: |
        chore: update {{range .Updated.Images}}{{println .}}{{end}}
    push:
      branch: main
  update:
    path: ./apps
    strategy: Setters
```

```yaml
# Im Deployment: Marker für Image Automation
containers:
  - name: myapp
    image: ghcr.io/firma/myapp:1.2.3   # {"$imagepolicy": "flux-system:myapp"}
```

---

## Secrets Management in GitOps

### Sealed Secrets (Bitnami)

```bash
# kubeseal installieren
brew install kubeseal

# Secret versiegeln (nur der Cluster kann entschlüsseln)
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=geheim \
  --dry-run=client -o yaml | \
  kubeseal --controller-namespace sealed-secrets \
    --controller-name sealed-secrets \
    --format yaml > sealed-db-credentials.yaml

# Ergebnis in Git committen (sicher!)
git add sealed-db-credentials.yaml && git commit -m "feat: add db credentials"
```

### External Secrets Operator (ESO) — Azure Key Vault

```yaml
# SecretStore: Verbindung zu Azure Key Vault
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: azure-keyvault
spec:
  provider:
    azurekv:
      tenantId: "tenant-id"
      vaultUrl: "https://kv-firma-prod.vault.azure.net"
      authType: WorkloadIdentity
      serviceAccountRef:
        name: external-secrets-sa
        namespace: external-secrets

---
# ExternalSecret: Secret aus Key Vault laden
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: azure-keyvault
    kind: ClusterSecretStore
  target:
    name: myapp-secrets
    creationPolicy: Owner
  data:
    - secretKey: db-password       # Kubernetes Secret Key
      remoteRef:
        key: myapp-db-password     # Azure Key Vault Secret Name
    - secretKey: api-key
      remoteRef:
        key: myapp-api-key
```

---

## Multi-Cluster GitOps (ArgoCD)

```yaml
# Externer Cluster registrieren
argocd cluster add production-cluster --name prod-eu-west

# ApplicationSet — Deploy zu mehreren Clustern
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-multicluster
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - cluster: prod-eu-west
            url: https://prod-eu-west.k8s.firma.de
            environment: production
          - cluster: prod-eu-central
            url: https://prod-eu-central.k8s.firma.de
            environment: production
  template:
    metadata:
      name: 'myapp-{{cluster}}'
    spec:
      project: production
      source:
        repoURL: https://github.com/firma/k8s-configs
        targetRevision: main
        path: 'apps/myapp/overlays/{{environment}}'
      destination:
        server: '{{url}}'
        namespace: production
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

## GitOps Checkliste (Best Practices)

```
✅ Kein kubectl apply --force auf Produktions-Cluster direkt
✅ Alle Konfigurationsänderungen via Pull Request (4-Augen-Prinzip)
✅ Branch Protection auf main: keine direkten Pushes
✅ Secrets: NIEMALS plain-text in Git (Sealed Secrets oder ESO)
✅ Image-Tags: keine :latest in Prod (immer SHA oder Semver)
✅ Sync-Status: Alert wenn App OutOfSync > 10 Minuten
✅ Pruning: prune: true aktiviert (verwaiste Ressourcen löschen)
✅ selfHeal: true aktiviert (manuelle kubectl-Änderungen werden revertiert)
✅ Separate Repos: App-Code vs. K8s-Konfiguration trennen
✅ RBAC: Developer lesen nur, Platform-Team deployt
```
