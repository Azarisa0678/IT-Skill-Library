# Observability — Referenzmodul

Prometheus, Grafana, Loki, Tempo, OpenTelemetry, Alerting, Dashboards, SLO-Monitoring.

---

## Prometheus — Konfiguration und Alerting

```yaml
# prometheus.yml — Scrape-Konfiguration
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: prod-eu-west
    environment: production

rule_files:
  - /etc/prometheus/rules/*.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

scrape_configs:
  - job_name: kubernetes-pods
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: "true"
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod

  - job_name: kubernetes-nodes
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
```

### Alert-Regeln (Produktionsreif)

```yaml
# rules/application.yml
groups:
  - name: application
    interval: 30s
    rules:
      # ── Verfügbarkeit ─────────────────────────────────────────────────────
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
          /
          sum(rate(http_requests_total[5m])) by (service)
          > 0.01
        for: 2m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "Hohe Fehlerrate: {{ $labels.service }}"
          description: "{{ $value | humanizePercentage }} Fehlerrate (Schwellenwert: 1%)"
          runbook_url: "https://wiki.firma.de/runbooks/high-error-rate"

      # ── Latenz ────────────────────────────────────────────────────────────
      - alert: HighLatencyP99
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
          ) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "P99-Latenz hoch: {{ $labels.service }}"
          description: "P99 = {{ $value | humanizeDuration }} (Schwellenwert: 500ms)"

      # ── Pods ──────────────────────────────────────────────────────────────
      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[15m]) * 60 * 15 > 3
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Pod crasht: {{ $labels.pod }} in {{ $labels.namespace }}"

      - alert: PodOOMKilled
        expr: kube_pod_container_status_last_terminated_reason{reason="OOMKilled"} == 1
        for: 0m
        labels:
          severity: warning
        annotations:
          summary: "OOMKilled: {{ $labels.container }} in {{ $labels.pod }}"

      # ── Ressourcen ────────────────────────────────────────────────────────
      - alert: NodeDiskPressure
        expr: kube_node_status_condition{condition="DiskPressure",status="true"} == 1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Disk-Pressure auf Node: {{ $labels.node }}"

      - alert: PersistentVolumeFillingUp
        expr: |
          kubelet_volume_stats_available_bytes
          / kubelet_volume_stats_capacity_bytes < 0.15
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "PV füllt sich: {{ $labels.persistentvolumeclaim }}"
          description: "Nur noch {{ $value | humanizePercentage }} frei"

  - name: slo
    rules:
      # ── SLO Burn-Rate Alert (Multi-Window) ────────────────────────────────
      - alert: SLOBurnRateCritical
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[1h])) by (service)
            / sum(rate(http_requests_total[1h])) by (service)
          ) > (14.4 * 0.001)
          and
          (
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
            / sum(rate(http_requests_total[5m])) by (service)
          ) > (14.4 * 0.001)
        for: 2m
        labels:
          severity: critical
          page: "true"
        annotations:
          summary: "SLO Burn Rate kritisch: {{ $labels.service }}"
          description: "Burn-Rate 14.4x — Error-Budget wird in 1 Stunde verbraucht"
```

### Alertmanager-Konfiguration

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'mail.firma.de:587'
  smtp_from: 'alerts@firma.de'
  smtp_auth_username: 'alerts@firma.de'
  smtp_auth_password_file: '/etc/alertmanager/smtp-password'

route:
  group_by: [alertname, service, namespace]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: default
  routes:
    - matchers:
        - severity=critical
      receiver: pagerduty
      continue: true
    - matchers:
        - severity=critical
        - page=true
      receiver: oncall
    - matchers:
        - team=backend
      receiver: backend-team

receivers:
  - name: default
    email_configs:
      - to: 'it-ops@firma.de'
        require_tls: true
  - name: pagerduty
    pagerduty_configs:
      - routing_key: $PAGERDUTY_KEY
        description: '{{ .GroupLabels.alertname }}: {{ .Annotations.summary }}'
  - name: oncall
    webhook_configs:
      - url: 'https://oncall.firma.de/webhook/prometheus'
  - name: backend-team
    slack_configs:
      - api_url: $SLACK_WEBHOOK
        channel: '#alerts-backend'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ .Annotations.description }}'

inhibit_rules:
  - source_matchers: [severity=critical]
    target_matchers: [severity=warning]
    equal: [alertname, service]
```

---

## OpenTelemetry — Instrumentierung

```yaml
# otel-collector.yaml — Collector-Konfiguration
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-prod
spec:
  mode: DaemonSet
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
      prometheus:
        config:
          scrape_configs:
            - job_name: otel-collector
              static_configs:
                - targets: [localhost:8888]

    processors:
      batch:
        timeout: 10s
        send_batch_size: 1024
      memory_limiter:
        limit_mib: 512
        spike_limit_mib: 128
      resource:
        attributes:
          - action: insert
            key: cluster
            value: prod-eu-west

    exporters:
      otlp/tempo:
        endpoint: tempo:4317
        tls:
          insecure: true
      prometheusremotewrite:
        endpoint: http://prometheus:9090/api/v1/write
      loki:
        endpoint: http://loki:3100/loki/api/v1/push

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, batch, resource]
          exporters: [otlp/tempo]
        metrics:
          receivers: [otlp, prometheus]
          processors: [memory_limiter, batch]
          exporters: [prometheusremotewrite]
        logs:
          receivers: [otlp]
          processors: [memory_limiter, batch]
          exporters: [loki]
```

```python
# Python Anwendung — OTel Instrumentierung
from opentelemetry import trace, metrics
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor

# Tracer einrichten
provider = TracerProvider()
provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="http://otel-collector:4317"))
)
trace.set_tracer_provider(provider)

# Auto-Instrumentierung
FastAPIInstrumentor.instrument_app(app)
SQLAlchemyInstrumentor().instrument(engine=engine)

# Manuelles Span
tracer = trace.get_tracer(__name__)
with tracer.start_as_current_span("process-order") as span:
    span.set_attribute("order.id", order_id)
    span.set_attribute("order.amount", amount)
    # ...
```

---

## Grafana — Dashboards as Code

```yaml
# grafana-dashboard-configmap.yaml (via grafana-operator oder Provisioning)
apiVersion: v1
kind: ConfigMap
metadata:
  name: dashboard-application
  labels:
    grafana_dashboard: "1"
data:
  application.json: |
    {
      "title": "Application Overview",
      "uid": "app-overview-prod",
      "panels": [
        {
          "title": "Request Rate",
          "type": "graph",
          "targets": [{
            "expr": "sum(rate(http_requests_total[5m])) by (service)",
            "legendFormat": "{{service}}"
          }]
        },
        {
          "title": "Error Rate",
          "type": "stat",
          "targets": [{
            "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))",
            "legendFormat": "Error Rate"
          }],
          "thresholds": {
            "steps": [
              {"color": "green", "value": 0},
              {"color": "yellow", "value": 0.005},
              {"color": "red", "value": 0.01}
            ]
          }
        }
      ]
    }
```

---

## Loki — Log-Abfragen (LogQL)

```logql
# Alle Error-Logs der letzten Stunde für einen Service
{namespace="production", app="myapp"} |= "ERROR" | logfmt | level="error"

# Rate der Error-Logs (für Alerting)
sum(rate({namespace="production"} |= "ERROR" [5m])) by (app)

# Strukturierte Logs parsen (JSON)
{app="myapp"} | json | status_code >= 500
  | line_format "{{.timestamp}} {{.method}} {{.path}} → {{.status_code}}"

# Latenz aus Logs extrahieren
{app="myapp"} | logfmt | duration > 500ms
  | unwrap duration | avg_over_time [5m]
```
