# Kubernetes & Helm — Referenzmodul

Helm Charts, Release-Strategien, Kustomize, Production-Hardening, HPA/KEDA, Network Policies.

---

## Production-reifes Helm Chart

```
myapp/
├── Chart.yaml
├── values.yaml
├── values-prod.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   ├── networkpolicy.yaml
│   ├── serviceaccount.yaml
│   ├── configmap.yaml
│   └── _helpers.tpl
└── charts/            # Dependencies
```

```yaml
# templates/deployment.yaml — Production-Ready
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels: {{- include "myapp.labels" . | nindent 4 }}
  annotations:
    reloader.stakater.com/auto: "true"   # Auto-Rollout bei ConfigMap/Secret-Änderung
spec:
  replicas: {{ .Values.replicaCount }}
  revisionHistoryLimit: 3
  selector:
    matchLabels: {{- include "myapp.selectorLabels" . | nindent 6 }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0    # Zero-Downtime-Deployment
  template:
    metadata:
      labels: {{- include "myapp.selectorLabels" . | nindent 8 }}
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
    spec:
      serviceAccountName: {{ include "myapp.serviceAccountName" . }}
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
        seccompProfile:
          type: RuntimeDefault
      terminationGracePeriodSeconds: 60
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels: {{- include "myapp.selectorLabels" . | nindent 14 }}
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: [ALL]
          ports:
            - name: http
              containerPort: {{ .Values.service.port }}
              protocol: TCP
          resources: {{- toYaml .Values.resources | nindent 12 }}
          livenessProbe:
            httpGet:
              path: /healthz
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
          startupProbe:
            httpGet:
              path: /healthz
              port: http
            failureThreshold: 30
            periodSeconds: 10
          env:
            - name: APP_ENV
              value: {{ .Values.environment }}
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "myapp.fullname" . }}-secrets
                  key: db-password
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: cache
              mountPath: /app/cache
      volumes:
        - name: tmp
          emptyDir: {}
        - name: cache
          emptyDir: {}
```

```yaml
# templates/pdb.yaml — Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  minAvailable: 1
  selector:
    matchLabels: {{- include "myapp.selectorLabels" . | nindent 6 }}
```

```yaml
# templates/networkpolicy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  podSelector:
    matchLabels: {{- include "myapp.selectorLabels" . | nindent 6 }}
  policyTypes: [Ingress, Egress]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
      ports:
        - protocol: TCP
          port: {{ .Values.service.port }}
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: database
      ports:
        - protocol: TCP
          port: 5432
    - ports:                  # DNS immer erlauben
        - protocol: UDP
          port: 53
```

---

## Release-Strategien

### Blue-Green Deployment (Kubernetes)

```yaml
# Zwei Deployments (blue + green), Service zeigt auf aktive Farbe
# blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    slot: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      slot: blue
  template:
    metadata:
      labels:
        app: myapp
        slot: blue
    spec:
      containers:
        - name: myapp
          image: myapp:v1.0.0

---
# Service zeigt initial auf blue
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    slot: blue   # ← hier umschalten für Cutover
  ports:
    - port: 80
      targetPort: 8080
```

```bash
# Cutover Skript: Blue → Green
kubectl set image deployment/myapp-green myapp=myapp:v2.0.0
kubectl rollout status deployment/myapp-green --timeout=5m

# Smoke-Test gegen Green (z.B. via direktem Service)
curl -f http://myapp-green-svc/healthz || exit 1

# Umschalten
kubectl patch service myapp -p '{"spec":{"selector":{"slot":"green"}}}'

# Verifizieren + Blue als Fallback bereithalten (30 Min)
sleep 1800
# Falls OK: Blue auf neue Version updaten für nächsten Zyklus
kubectl set image deployment/myapp-blue myapp=myapp:v2.0.0
```

### Canary mit Argo Rollouts

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 10
  strategy:
    canary:
      canaryService: myapp-canary
      stableService: myapp-stable
      trafficRouting:
        nginx:
          stableIngress: myapp-stable
      steps:
        - setWeight: 10          # 10% Canary-Traffic
        - pause: {duration: 5m}
        - analysis:              # Automatische Metrik-Analyse
            templates:
              - templateName: success-rate
        - setWeight: 30
        - pause: {duration: 5m}
        - setWeight: 60
        - pause: {duration: 5m}
        - setWeight: 100
      autoPromotionEnabled: false
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:v2.0.0
```

---

## HPA + KEDA (Event-Driven Autoscaling)

```yaml
# HPA — CPU/Memory basiert
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 4
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300   # 5 Min warten vor Scale-Down

---
# KEDA — Azure Service Bus Queue basiert
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: myapp-queue-scaler
spec:
  scaleTargetRef:
    name: myapp-worker
  minReplicaCount: 0      # Scale-to-Zero möglich!
  maxReplicaCount: 50
  triggers:
    - type: azure-servicebus
      metadata:
        queueName: orders
        namespace: sb-firma-prod
        messageCount: "5"   # 1 Pod pro 5 Nachrichten
      authenticationRef:
        name: keda-sb-auth
```

---

## Kustomize — Umgebungs-Overlays

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patch-replicas.yaml
    ├── staging/
    └── prod/
        ├── kustomization.yaml
        ├── patch-resources.yaml
        └── patch-replicas.yaml
```

```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: production
resources:
  - ../../base
patches:
  - path: patch-resources.yaml
    target:
      kind: Deployment
      name: myapp
images:
  - name: myapp
    newTag: "1.5.2"
replicas:
  - name: myapp
    count: 5
commonLabels:
  environment: production
```

---

## Helm-Verwaltung (Tipps & Tricks)

```bash
# Installation mit Warte-Timeout und Rollback bei Fehler
helm upgrade --install myapp ./myapp \
  --namespace production \
  --create-namespace \
  --values values-prod.yaml \
  --set image.tag=$IMAGE_TAG \
  --wait --timeout 10m \
  --atomic          # Automatischer Rollback bei Fehlschlag

# Diff vor Upgrade (helm-diff Plugin)
helm diff upgrade myapp ./myapp -f values-prod.yaml --set image.tag=$IMAGE_TAG

# Release-Historie
helm history myapp -n production

# Rollback auf vorherige Version
helm rollback myapp 0 -n production  # 0 = vorherige Version

# Template-Ausgabe debuggen
helm template myapp ./myapp -f values-prod.yaml | kubectl apply --dry-run=client -f -

# Lint
helm lint ./myapp -f values-prod.yaml

# chart-testing (ct) — CI
ct lint --chart-dirs . --all
ct install --chart-dirs . --all
```
