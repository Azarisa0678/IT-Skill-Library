# Serverless & Kubernetes Security — Referenzmodul

Lambda/Azure Functions Security, Kubernetes Security (RBAC, PSA, Network Policies, Falco),
Container Security, Runtime Protection, Secrets Management Cloud-Native.

---

## Serverless Security

### AWS Lambda Security

```python
#!/usr/bin/env python3
"""AWS Lambda — Produktionsreife Sicherheitskonfiguration"""
import json
import boto3
import logging
import os
from functools import wraps

logger = logging.getLogger()
logger.setLevel(logging.INFO)

# ─── INPUT-VALIDIERUNG ────────────────────────────────────────────────────────
def validate_event(schema: dict):
    """Decorator für Event-Validierung"""
    def decorator(func):
        @wraps(func)
        def wrapper(event, context):
            for field, validator in schema.items():
                if field not in event:
                    raise ValueError(f"Pflichtfeld fehlt: {field}")
                if not validator(event[field]):
                    raise ValueError(f"Ungültiger Wert für: {field}")
            return func(event, context)
        return wrapper
    return decorator

# ─── SECRETS AUS SSM / SECRETS MANAGER (nie ENV-Variablen für Secrets!) ───────
def get_secret(secret_name: str) -> dict:
    """Secret aus AWS Secrets Manager laden (gecacht)"""
    if not hasattr(get_secret, '_cache'):
        get_secret._cache = {}
    if secret_name not in get_secret._cache:
        client = boto3.client('secretsmanager')
        response = client.get_secret_value(SecretId=secret_name)
        get_secret._cache[secret_name] = json.loads(response['SecretString'])
    return get_secret._cache[secret_name]

# ─── LAMBDA HANDLER (sicher) ──────────────────────────────────────────────────
@validate_event({
    'userId':   lambda v: isinstance(v, str) and v.isalnum() and len(v) <= 50,
    'action':   lambda v: v in ['read', 'write', 'delete'],
    'resource': lambda v: isinstance(v, str) and len(v) <= 200
})
def lambda_handler(event, context):
    """Sicherer Lambda Handler mit vollständigem Logging"""

    # Strukturiertes Logging für CloudWatch
    logger.info(json.dumps({
        "requestId":  context.aws_request_id,
        "userId":     event.get('userId'),
        "action":     event.get('action'),
        "resource":   event.get('resource'),
        "sourceIp":   event.get('requestContext', {}).get('identity', {}).get('sourceIp')
    }))

    # Credentials aus Secrets Manager (niemals hartcodieren!)
    db_creds = get_secret(os.environ['DB_SECRET_NAME'])

    try:
        # Business-Logik hier...
        result = process_request(event, db_creds)
        return {
            'statusCode': 200,
            'body': json.dumps(result),
            'headers': {
                'Content-Type': 'application/json',
                'X-Content-Type-Options': 'nosniff',
                'X-Frame-Options': 'DENY',
                'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
            }
        }
    except PermissionError as e:
        logger.warning(f"Zugriff verweigert: {e}")
        return {'statusCode': 403, 'body': json.dumps({'error': 'Forbidden'})}
    except Exception as e:
        logger.error(f"Unerwarteter Fehler: {e}", exc_info=True)
        # NIEMALS interne Details im Response zurückgeben!
        return {'statusCode': 500, 'body': json.dumps({'error': 'Internal Server Error'})}
```

### Lambda IAM — Least Privilege

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["secretsmanager:GetSecretValue"],
      "Resource": "arn:aws:secretsmanager:eu-central-1:123456789:secret:prod/myapp/*",
      "Condition": {
        "StringEquals": {"aws:RequestedRegion": "eu-central-1"}
      }
    },
    {
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem", "dynamodb:PutItem"],
      "Resource": "arn:aws:dynamodb:eu-central-1:123456789:table/MyTable",
      "Condition": {
        "ForAllValues:StringEquals": {
          "dynamodb:LeadingKeys": ["${aws:PrincipalTag/UserId}"]
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:eu-central-1:123456789:log-group:/aws/lambda/myfunction:*"
    }
  ]
}
```

### Terraform: Sichere Lambda-Konfiguration

```hcl
resource "aws_lambda_function" "api" {
  function_name = "api-prod"
  runtime       = "python3.12"
  handler       = "handler.lambda_handler"
  filename      = data.archive_file.lambda.output_path
  role          = aws_iam_role.lambda.arn
  timeout       = 30
  memory_size   = 512

  # Kein direkter Internet-Zugang (VPC-gebunden)
  vpc_config {
    subnet_ids         = var.private_subnet_ids
    security_group_ids = [aws_security_group.lambda.id]
  }

  # Umgebungsvariablen NUR für nicht-sensitive Konfiguration!
  environment {
    variables = {
      DB_SECRET_NAME = "prod/myapp/database"    # Nur Name, nicht Wert!
      LOG_LEVEL      = "INFO"
      REGION         = "eu-central-1"
    }
  }

  # X-Ray Tracing
  tracing_config { mode = "Active" }

  # Code-Signing (nur signierte Pakete)
  code_signing_config_arn = aws_lambda_code_signing_config.main.arn

  # Reserved Concurrency (verhindert DDoS-Kostenfalle)
  reserved_concurrent_executions = 100

  tags = local.common_tags
}

# Lambda darf NIE aus Internet direkt erreichbar sein
resource "aws_security_group" "lambda" {
  name   = "sg-lambda-api"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS für AWS-APIs und externe Services"
  }
  # KEIN Ingress (Lambda wird via API GW aufgerufen, nicht direkt)
}
```

---

## Kubernetes Security

### RBAC — Least Privilege

```yaml
# Entwickler-Rolle: nur lesen + Pod-Logs in eigenem Namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log", "services", "configmaps"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch"]
  # KEIN: secrets, nodes, persistentvolumes
  # KEIN: create, update, delete, patch

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developers
  namespace: production
subjects:
  - kind: Group
    name: firma:developers    # Entra ID / Okta Gruppe
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io

---
# Service Account für Anwendung (Workload Identity)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp
  namespace: production
  annotations:
    # AWS EKS: IRSA (IAM Roles for Service Accounts)
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/myapp-prod
    # Azure AKS: Workload Identity
    azure.workload.identity/client-id: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
automountServiceAccountToken: false  # Nur setzen wenn wirklich benötigt!
```

### Pod Security Admission (PSA)

```yaml
# Namespace-Label für Pod Security Standards
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    # enforce: Pod wird abgelehnt wenn Standard nicht erfüllt
    # audit:   Audit-Log-Eintrag, Pod wird zugelassen
    # warn:    Warnung an kubectl, Pod wird zugelassen
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit:   restricted
    pod-security.kubernetes.io/warn:    restricted

---
# Pod der "restricted" PSA-Standard erfüllt
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
          # add: ["NET_BIND_SERVICE"]  # Nur wenn Port < 1024 nötig
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "512Mi"
```

### Falco — Runtime Security

```yaml
# Falco Helm Install
# helm install falco falcosecurity/falco --namespace falco --create-namespace \
#   --set falcosidekick.enabled=true \
#   --set falcosidekick.config.slack.webhookurl=https://hooks.slack.com/...

# Eigene Falco-Regel: Shell in Container
- rule: Shell spawned in container
  desc: Verdächtig — Shell in laufendem Container gestartet
  condition: >
    spawned_process
    and container
    and container.image.repository != "debug-tools"
    and proc.name in (shell_binaries)
    and not proc.pname in (container_entrypoints)
  output: >
    Shell gestartet (user=%user.name container=%container.name
    image=%container.image.repository cmd=%proc.cmdline)
  priority: WARNING
  tags: [container, shell, mitre_execution]

- rule: Sensitive file read in container
  desc: Zugriff auf sensible Dateien
  condition: >
    open_read
    and container
    and (fd.name startswith /etc/shadow
         or fd.name startswith /etc/kubernetes/admin.conf
         or fd.name startswith /run/secrets/kubernetes.io)
    and not proc.name in (known_sa_processes)
  output: >
    Sensible Datei gelesen (file=%fd.name user=%user.name container=%container.name)
  priority: CRITICAL
```

### Kubernetes Network Policies (Zero Trust)

```yaml
# Default Deny — alles blockieren, dann explizit freigeben
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}    # Gilt für alle Pods
  policyTypes: [Ingress, Egress]
  # Kein ingress/egress = alles blockiert!

---
# Nur API → Datenbank (keine andere Verbindung zur DB)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes: [Ingress]
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: myapi
      ports:
        - protocol: TCP
          port: 5432

---
# Pods dürfen nur DNS + bestimmte externe APIs
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-egress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: myapi
  policyTypes: [Egress]
  egress:
    - ports:                     # DNS immer erlauben
        - protocol: UDP
          port: 53
    - to:
        - podSelector:
            matchLabels:
              app: postgres
      ports:
        - port: 5432
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 10.0.0.0/8      # Kein Zugriff auf interne Netze
              - 172.16.0.0/12
              - 192.168.0.0/16
      ports:
        - port: 443
```

### Secrets — External Secrets Operator (ESO)

```yaml
# ClusterSecretStore: Verbindung zu AWS Secrets Manager
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: aws-secretsmanager
spec:
  provider:
    aws:
      service: SecretsManager
      region: eu-central-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
            namespace: external-secrets

---
# ExternalSecret: Lädt Secret aus AWS und erstellt K8s Secret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: ClusterSecretStore
  target:
    name: myapp-secrets
    creationPolicy: Owner
    template:
      type: Opaque
  data:
    - secretKey: db-password
      remoteRef:
        key: prod/myapp/database
        property: password
    - secretKey: api-key
      remoteRef:
        key: prod/myapp/api-key
```

---

## Kubernetes Security Checkliste

```markdown
## Kubernetes Security Baseline — Checkliste

### Cluster
- [ ] Kubernetes-Version aktuell (max. 2 Minor-Versionen hinter aktuell)
- [ ] API-Server: kein anonymer Zugang (`--anonymous-auth=false`)
- [ ] RBAC aktiviert (`--authorization-mode=RBAC`)
- [ ] Audit-Logging aktiviert (alle API-Calls loggen)
- [ ] etcd: Verschlüsselung at rest aktiviert
- [ ] kubelet: `--protect-kernel-defaults=true`

### Workloads
- [ ] Pod Security Admission: `restricted` in Production-Namespaces
- [ ] Alle Container: `runAsNonRoot: true`
- [ ] Alle Container: `readOnlyRootFilesystem: true`
- [ ] Alle Container: `capabilities.drop: [ALL]`
- [ ] Alle Container: `allowPrivilegeEscalation: false`
- [ ] Resource Limits für alle Container gesetzt
- [ ] `automountServiceAccountToken: false` wo nicht benötigt

### Netzwerk
- [ ] Default Deny NetworkPolicy in allen Namespaces
- [ ] Ingress-Controller: WAF vorgelagert
- [ ] Pod-zu-Pod-Kommunikation: explizit definiert

### Secrets
- [ ] Kein Secret in ConfigMap oder ENV (nur in Secret-Objekten)
- [ ] External Secrets Operator (ESO) für externe Secrets
- [ ] Kubernetes Secrets at rest verschlüsselt (KMS-Integration)
- [ ] Secrets im Code: Gitleaks in CI/CD aktiv

### Runtime
- [ ] Falco oder vergleichbares Runtime-Security-Tool aktiv
- [ ] Container-Images: nur aus vertrauenswürdiger Registry
- [ ] Image Signing: Cosign Policy Controller aktiv
- [ ] Image-Scan: Trivy in CI/CD, keine CRITICAL-Lücken in Prod
- [ ] Keine privilegierten Container in Production (`privileged: false`)
```
