# Eval-Testfälle: DevOps / CI-CD Skill

## Kategorie 1: Trigger-Tests

### T001 — GitHub Actions Pipeline
**Input:** "Erstelle eine GitHub Actions Pipeline für eine Node.js-Anwendung mit Tests, Security-Scan und Docker-Build"
**Kriterien:**
- ✅ on: push/pull_request korrekt
- ✅ concurrency mit cancel-in-progress
- ✅ npm ci (nicht npm install)
- ✅ Trivy oder Snyk für Security-Scan
- ✅ docker/build-push-action mit Cache (type=gha)
- ✅ OIDC statt statischem Secret für Registry-Login
- ✅ Environment für Production-Deploy

### T002 — Terraform Azure
**Input:** "Erstelle eine Terraform-Konfiguration für eine Azure Web App mit Datenbank"
**Kriterien:**
- ✅ Backend "azurerm" für Remote State
- ✅ required_version und required_providers
- ✅ locals für common_tags
- ✅ lifecycle { prevent_destroy = true } für Produktion
- ✅ Outputs definiert
- ✅ Variables mit validation
- ✅ Kein Passwort hartcodiert (sensitive = true)

### T003 — Helm Chart
**Input:** "Erstelle ein Helm Chart für eine Web-Anwendung mit 3 Replicas und Health Checks"
**Kriterien:**
- ✅ Deployment mit RollingUpdate-Strategie
- ✅ SecurityContext: runAsNonRoot, readOnlyRootFilesystem
- ✅ Resource Limits und Requests
- ✅ livenessProbe und readinessProbe
- ✅ Service und Ingress vorhanden
- ✅ values.yaml mit sinnvollen Defaults

### T004 — GitOps mit ArgoCD
**Input:** "Wie implementiere ich GitOps für unsere Kubernetes-Cluster mit ArgoCD?"
**Kriterien:**
- ✅ Application-Manifest vollständig
- ✅ syncPolicy.automated mit prune und selfHeal
- ✅ App-of-Apps Pattern erklärt
- ✅ Git als Single Source of Truth betont
- ✅ Kein manuelles kubectl apply auf Prod

### T005 — Terraform CI/CD
**Input:** "Wie integriere ich Terraform in eine GitHub Actions Pipeline mit Security-Scan?"
**Kriterien:**
- ✅ terraform init, validate, plan, apply als Stages
- ✅ OIDC für Cloud-Authentifizierung (kein Secret)
- ✅ Checkov oder tfsec für IaC Security
- ✅ Plan im PR kommentieren
- ✅ Apply nur auf main nach PR-Merge

### T006 — SLO definieren
**Input:** "Definiere SLOs für unsere API mit 99.9% Verfügbarkeit"
**Kriterien:**
- ✅ SLI-Formel in PromQL/KQL
- ✅ SLO-Ziel und Zeitfenster
- ✅ Error Budget berechnet (43.8 Min/Monat)
- ✅ Error Budget Policy beschrieben
- ✅ Unterschied SLI/SLO/SLA erklärt

### T007 — Observability Stack
**Input:** "Richte einen Observability-Stack mit Prometheus und Grafana ein"
**Kriterien:**
- ✅ Docker Compose oder Kubernetes-Deployment
- ✅ Prometheus-Konfiguration (scrape_configs)
- ✅ Alertmanager integriert
- ✅ Mindestens 3 sinnvolle Alert-Regeln
- ✅ Loki für Logs erwähnt
- ✅ Node Exporter als Basis-Metrik

### T008 — Postmortem
**Input:** "Unser API-Gateway war 2 Stunden ausgefallen. Erstelle ein Postmortem"
**Kriterien:**
- ✅ Vollständige Timeline (Entdeckung → Behebung)
- ✅ Root Cause Analysis (5 Whys)
- ✅ Impact-Messung (Nutzer, Error Budget)
- ✅ Konkrete Maßnahmen mit Verantwortlichem und Frist
- ✅ Blameless-Ansatz (kein Schuldiger)
- ✅ "Lessons Learned" Sektion

## Kategorie 2: Qualitäts-Tests

### Q001 — Security by Default
**Jede Pipeline muss enthalten:**
- ✅ Secret-Scanning (Gitleaks)
- ✅ Dependency-Scan (npm audit / Snyk)
- ✅ Container-Scan wenn Docker verwendet
- ✅ OIDC statt statische Secrets

### Q002 — IaC Best Practices
**Jede Terraform-Konfiguration:**
- ✅ Remote State Backend
- ✅ Keine Passwörter im Code
- ✅ Tagging-Strategie
- ✅ prevent_destroy für kritische Ressourcen

### Q003 — GitOps-Prinzip
**Bei K8s-Deployments:**
- ✅ Kein manuelles kubectl apply empfohlen
- ✅ Git als einzige Source of Truth
- ✅ ArgoCD oder Flux als Tool

## Kategorie 3: Negativ-Tests

### N001 — Linux-Administration
**Input:** "Wie konfiguriere ich SELinux?"
**Erwartetes Verhalten:**
- ❌ DevOps Skill nicht primär zuständig
- ✅ Linux Sysadmin Skill empfehlen

### N002 — Security Pentest
**Input:** "Führe einen Penetrationstest durch"
**Erwartetes Verhalten:**
- ❌ DevOps Skill nicht zuständig
- ✅ IT Security Skill empfehlen

## Bewertungsschema
**Mindest-Score:** 4/5 auf alle T-Tests
