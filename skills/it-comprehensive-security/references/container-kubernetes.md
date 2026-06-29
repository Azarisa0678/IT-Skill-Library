# Container & Kubernetes Security

## Docker Hardening

### Secure Dockerfile Practices

```dockerfile
# Use specific, minimal base image — never :latest in production
FROM python:3.12-slim-bookworm

# Run as non-root user
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

# Set working directory
WORKDIR /app

# Copy only what's needed — use .dockerignore
COPY --chown=appuser:appgroup requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appgroup . .

# Drop to non-root before CMD
USER appuser

# Explicit port — do not use EXPOSE for secrets
EXPOSE 8080

# Use exec form (not shell form) to handle signals correctly
CMD ["python", "app.py"]
```

**.dockerignore** (always include):
```
.git
.env
*.key
*.pem
secrets/
__pycache__
*.pyc
.DS_Store
node_modules/
```

### Docker Run Hardening
```bash
docker run \
  --read-only \                          # Read-only filesystem
  --tmpfs /tmp \                         # Writable tmp in memory only
  --no-new-privileges \                  # Prevent privilege escalation
  --cap-drop ALL \                       # Drop all capabilities
  --cap-add NET_BIND_SERVICE \           # Add only what's needed
  --security-opt no-new-privileges:true \
  --security-opt seccomp=default.json \  # Seccomp profile
  --user 1000:1000 \                     # Run as non-root UID
  --network app-network \                # Named network, not default bridge
  --memory 512m \                        # Resource limits
  --cpus 1 \
  myimage:1.2.3
```

### Docker Security Scanning
```bash
# Trivy — image vulnerability scanning
trivy image myimage:1.2.3
trivy image --severity HIGH,CRITICAL myimage:1.2.3
trivy image --format sarif --output results.sarif myimage:1.2.3

# Grype — alternative scanner
grype myimage:1.2.3

# Docker Bench Security — host/daemon hardening check
docker run --rm --net host --pid host --userns host --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /etc:/etc:ro -v /usr/bin/containerd:/usr/bin/containerd:ro \
  -v /usr/bin/runc:/usr/bin/runc:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  docker/docker-bench-security

# Hadolint — Dockerfile linting
hadolint Dockerfile
```

### Docker Checklist
- [ ] Non-root user in all containers
- [ ] Specific image tags — no `:latest` in production
- [ ] Multi-stage builds to minimize image size and attack surface
- [ ] No secrets in Dockerfile or image layers
- [ ] Read-only filesystem where possible
- [ ] All capabilities dropped, only required ones added
- [ ] Resource limits set (memory, CPU)
- [ ] Images scanned for CVEs before deployment
- [ ] Docker daemon not exposed over TCP without TLS
- [ ] Docker socket not mounted in containers
- [ ] Content Trust enabled (`DOCKER_CONTENT_TRUST=1`)

---

## Kubernetes Security

### RBAC (Role-Based Access Control)

```yaml
# Principle of least privilege — namespace-scoped role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: app-reader
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]     # Read-only — no create/delete/patch

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-reader-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: my-app-sa
  namespace: production
roleRef:
  kind: Role
  name: app-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Audit RBAC — who can do what
kubectl auth can-i --list --namespace production
kubectl auth can-i create pods --as system:serviceaccount:production:my-app-sa

# Find overly permissive ClusterRoleBindings
kubectl get clusterrolebindings -o json | jq -r \
  '.items[] | select(.roleRef.name=="cluster-admin") | .metadata.name + ": " + (.subjects[]?.name // "N/A")'
```

### Pod Security Standards

```yaml
# Enforce Pod Security Standards at namespace level (K8s 1.25+)
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted     # Block non-compliant pods
    pod-security.kubernetes.io/audit: restricted       # Audit violations
    pod-security.kubernetes.io/warn: restricted        # Warn on violations

---
# Compliant Pod spec for "restricted" policy
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myimage:1.2.3
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    resources:
      limits:
        memory: "256Mi"
        cpu: "500m"
      requests:
        memory: "128Mi"
        cpu: "100m"
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
```

### Network Policies

```yaml
# Default deny all ingress and egress in namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# Allow only specific traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-traffic
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
```

### Secrets Management

```yaml
# Never store secrets in plaintext in YAML — use external secrets
# Option 1: External Secrets Operator (pulls from Vault/AWS SSM/Azure KV)
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: db-credentials
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: secret/production/database
      property: password
```

```bash
# Audit: find secrets that might contain sensitive data
kubectl get secrets --all-namespaces -o json | jq -r \
  '.items[] | .metadata.namespace + "/" + .metadata.name + " [" + .type + "]"'

# Check for secrets mounted as env vars (less secure than volume mounts)
kubectl get pods --all-namespaces -o json | jq -r \
  '.items[] | select(.spec.containers[].env[]?.valueFrom.secretKeyRef != null) |
  .metadata.namespace + "/" + .metadata.name'
```

### Kubernetes Security Scanning Tools

```bash
# kube-bench — CIS Kubernetes Benchmark
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
kubectl logs job/kube-bench

# Trivy — K8s cluster scanning
trivy k8s --report summary cluster
trivy k8s --severity HIGH,CRITICAL --report all cluster

# Falco — Runtime threat detection
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm install falco falcosecurity/falco \
  --set falco.grpc.enabled=true \
  --set falco.grpcOutput.enabled=true

# Checkov — IaC scanning for K8s manifests
checkov -d ./k8s-manifests --framework kubernetes

# Polaris — Best practices audit
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
helm install polaris fairwinds-stable/polaris --namespace polaris --create-namespace
```

### Kubernetes Security Checklist

**Cluster Configuration:**
- [ ] API server: anonymous auth disabled, RBAC enabled, audit logging on
- [ ] etcd encrypted at rest; TLS for etcd communication
- [ ] Kubelet: anonymous auth disabled, authorization mode not AlwaysAllow
- [ ] Network policies: default deny in all namespaces
- [ ] Pod Security Standards: restricted profile enforced in production

**Workloads:**
- [ ] All containers run as non-root
- [ ] All containers have `allowPrivilegeEscalation: false`
- [ ] All containers have `readOnlyRootFilesystem: true` where possible
- [ ] All capabilities dropped; only necessary ones added
- [ ] Resource limits set on all containers
- [ ] No `hostNetwork`, `hostPID`, or `hostIPC` unless explicitly required
- [ ] No `privileged: true` containers in production

**Images:**
- [ ] Images scanned for CVEs in CI/CD pipeline
- [ ] Only images from trusted registries (allowlist via admission controller)
- [ ] Image tags pinned (SHA digest recommended for production)
- [ ] Binary Authorization / image signing enforced

**Access & Secrets:**
- [ ] RBAC: least privilege for all service accounts
- [ ] No service account token auto-mount where not needed
- [ ] Secrets from external secrets manager (Vault, AWS SSM, Azure KV)
- [ ] No plaintext secrets in manifests or environment variables
- [ ] kubectl access: MFA + audit logging; no shared kubeconfig files

**Runtime:**
- [ ] Falco or equivalent runtime threat detection deployed
- [ ] Audit logs forwarded to SIEM
- [ ] Admission controllers: OPA/Gatekeeper or Kyverno for policy enforcement
- [ ] Node OS hardened; minimal packages; auto-patching enabled

---

## Supply Chain Security

```bash
# Sign container images with Cosign (Sigstore)
cosign generate-key-pair
cosign sign --key cosign.key myregistry/myimage:1.2.3

# Verify signature
cosign verify --key cosign.pub myregistry/myimage:1.2.3

# Generate SBOM (Software Bill of Materials)
syft myimage:1.2.3 -o spdx-json > sbom.spdx.json
trivy image --format spdx-json --output sbom.json myimage:1.2.3

# Scan SBOM for vulnerabilities
grype sbom:sbom.spdx.json
```
