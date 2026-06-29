# SRE Practices — Referenzmodul

SLO/SLI/SLA, Error Budgets, Toil-Reduktion, Incident Management, Postmortems, Capacity Planning.

---

## SLO-Framework

### SLI/SLO-Definition (Vorlage)

```yaml
# slo-definitions.yaml
services:
  api-gateway:
    slos:
      availability:
        description: "Anteil erfolgreicher HTTP-Anfragen"
        sli:
          good_events: 'sum(rate(http_requests_total{status!~"5.."}[1m]))'
          total_events: 'sum(rate(http_requests_total[1m]))'
        target: 99.9       # Prozent
        window: 30d
        error_budget_minutes: 43.8   # (1 - 0.999) * 30 * 24 * 60

      latency:
        description: "P99-Latenz < 500ms"
        sli:
          good_events: 'sum(rate(http_request_duration_seconds_bucket{le="0.5"}[1m]))'
          total_events: 'sum(rate(http_request_duration_seconds_count[1m]))'
        target: 99.0
        window: 30d

  payment-service:
    slos:
      availability:
        target: 99.95    # Strenger wegen Zahlungsprozessen
        window: 30d
        error_budget_minutes: 21.9

      transaction_success:
        description: "Erfolgreiche Zahlungstransaktionen"
        sli:
          good_events: 'sum(rate(payment_transactions_total{status="success"}[1m]))'
          total_events: 'sum(rate(payment_transactions_total[1m]))'
        target: 99.99
        window: 30d
```

### Error Budget Policy

```markdown
## Error Budget Policy — [Unternehmen]

### Grundsätze
- Error Budget = 1 - SLO-Ziel, gemessen über 30-Tage-Fenster
- Budget-Verbrauch steuert Feature-Velocity vs. Reliability-Fokus

### Budget-Zustände und Reaktionen

**>50% Budget verbleibend (grün):**
- Normaler Entwicklungsbetrieb
- Feature-Deploys ohne besondere Einschränkungen
- Reliability-Verbesserungen nach Bedarf

**25-50% Budget verbleibend (gelb):**
- Deployment-Freeze für riskante Änderungen
- Dev-Team widmet 20% Zeit für Reliability-Arbeit
- Wöchentliches SLO-Review

**<25% Budget verbleibend (orange):**
- Kein Feature-Deploy bis Budget erholt
- Dev-Team widmet 50% Zeit für Reliability-Arbeit
- Tägliches SLO-Review mit Management

**0% / Budget überschritten (rot):**
- Vollständiger Feature-Freeze
- Alle Kapazitäten auf Reliability
- Postmortem für alle Incidents der letzten 30 Tage
- Management-Eskalation

### Ausnahmen
- Security-Fixes: immer deployen, auch bei Budget-Freeze
- Hotfixes für laufende Incidents: erlaubt
- Geplante Wartung: aus Maintenance-Window-Budget (separat)
```

---

## Burn-Rate Alerts (Multi-Window)

```yaml
# Prometheus Alert-Regeln für SLO Burn Rate
# Quelle: Google SRE Workbook, Kapitel 5
groups:
  - name: slo-burn-rate
    rules:
      # Kritisch: 2% Budget in 1h verbrannt (Faktor 14.4x)
      - alert: SLOBurnRate_Critical_1h
        expr: |
          (
            error_rate_1h > (14.4 * (1 - 0.999))
          ) and (
            error_rate_5m > (14.4 * (1 - 0.999))
          )
        labels:
          severity: page
          window: 1h
        annotations:
          summary: "Kritisch: SLO-Budget verbraucht in ~1h"

      # Hoch: 5% Budget in 6h verbrannt (Faktor 6x)
      - alert: SLOBurnRate_High_6h
        expr: |
          (
            error_rate_6h > (6 * (1 - 0.999))
          ) and (
            error_rate_30m > (6 * (1 - 0.999))
          )
        labels:
          severity: ticket
          window: 6h

      # Mittel: 10% Budget in 3 Tagen (Faktor 3x)
      - alert: SLOBurnRate_Medium_3d
        expr: error_rate_3d > (3 * (1 - 0.999))
        labels:
          severity: ticket
          window: 3d

# Hilfsmethoden (Recording Rules für Performance)
  - name: slo-recording
    rules:
      - record: error_rate_5m
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m]))
      - record: error_rate_30m
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[30m]))
          / sum(rate(http_requests_total[30m]))
      - record: error_rate_1h
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[1h]))
          / sum(rate(http_requests_total[1h]))
      - record: error_rate_6h
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[6h]))
          / sum(rate(http_requests_total[6h]))
      - record: error_rate_3d
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[3d]))
          / sum(rate(http_requests_total[3d]))
```

---

## Incident Management

### Schweregrade (Severity Levels)

| Severity | Definition | Reaktionszeit | Kommunikation |
|---|---|---|---|
| **SEV-1** | Produktionsausfall, alle Nutzer betroffen, Datenverlust | Sofort (< 5 Min) | Status-Page + Stakeholder |
| **SEV-2** | Erhebliche Beeinträchtigung, >10% Nutzer betroffen | < 15 Min | Status-Page |
| **SEV-3** | Teilweiser Ausfall, Workaround verfügbar | < 1 Std | Internes Ticket |
| **SEV-4** | Geringe Auswirkung, kosmetisches Problem | Nächster Werktag | Ticket |

### Incident-Commander-Checkliste

```markdown
## Incident-Checkliste (SEV-1 / SEV-2)

### T+0 — Erkennen
- [ ] Incident-Kanal erstellen: #incident-YYYY-MM-DD-kurzbeschreibung
- [ ] Incident Commander (IC) und Communications Lead (CL) benennen
- [ ] Status-Page updaten: "Wir untersuchen Probleme mit [Service]"
- [ ] Ersten Status-Post im Kanal: Was ist bekannt, wer ist dran?

### T+5 Min — Stabilisieren
- [ ] Severity einschätzen: SEV-1 oder SEV-2?
- [ ] Bei SEV-1: Management + On-Call-Eskalation sofort
- [ ] Runbook: Gibt es ein passendes?
- [ ] Symptome dokumentieren (Screenshots, Metriken-Links, Fehlermeldungen)

### T+15 Min — Eindämmen
- [ ] Rollback möglich? → Wenn ja, sofort evaluieren
- [ ] Feature-Flag deaktivieren?
- [ ] Traffic-Routing ändern (Canary zurückschalten, Region isolieren)?
- [ ] Schaden begrenzen vor vollständiger RCA

### Regelmäßig — Kommunikation
- [ ] Alle 30 Min: Status-Update im Kanal (auch "kein Fortschritt" ist ein Update!)
- [ ] Status-Page alle 30 Min aktualisieren
- [ ] Stakeholder aktiv informieren (kein "sie werden es schon sehen")

### Nach Behebung
- [ ] Status-Page: Resolved (mit kurzem Beschreibungstext)
- [ ] Postmortem-Termin innerhalb 48h ansetzen
- [ ] Incident-Dokument fertigstellen
- [ ] Thank-you an alle Beteiligten
```

---

## Postmortem-Vorlage (Blameless)

```markdown
# Postmortem: [Kurztitel]

**Datum:** [TT.MM.JJJJ]
**Severity:** SEV-[1/2/3]
**Dauer:** [Beginn] → [Ende] (X Stunden Y Minuten)
**Impact:** [Z% der Nutzer betroffen, ca. X Transaktionen verloren]
**Autor:** [Name]
**Reviewer:** [Namen]

---

## Zusammenfassung (Executive Summary)
[2-3 Sätze: Was ist passiert, wie lange, was waren die Auswirkungen?]

## Timeline
| Zeitstempel | Ereignis |
|-------------|---------|
| 14:23 | Alertmanager: HighErrorRate auf api-gateway |
| 14:25 | On-Call nimmt Incident an, Kanal erstellt |
| 14:31 | Root Cause identifiziert: fehlerhafte DB-Migration |
| 14:45 | Rollback eingeleitet |
| 14:52 | Service wieder stabil, Fehlerrate < 0.1% |
| 15:10 | Status-Page: Resolved |

## Root Cause
[Was hat das Problem verursacht? Technisch präzise, ohne Schuldzuweisungen.]

Beispiel: *Ein automatisierter DB-Migrations-Job fügte einen non-nullable Index ohne DEFAULT-Wert hinzu.
Dadurch schlugen alle INSERT-Operationen fehl, bis das Deployment zurückgerollt wurde.*

## Was gut funktioniert hat
- Alerting hat das Problem innerhalb 2 Minuten erkannt
- Runbook war aktuell und hat den Rollback beschleunigt
- Kommunikation im Incident-Kanal war klar und regelmäßig

## Was schlecht funktioniert hat
- DB-Migrationen waren nicht in der Staging-Pipeline getestet
- Kein automatischer Smoke-Test nach Deploy
- Status-Page Update kam 10 Min verzögert

## Maßnahmen (Action Items)

| Maßnahme | Verantwortlich | Frist | Status |
|----------|---------------|-------|--------|
| DB-Migrationen in Staging-Pipeline testen | @dev-lead | 2 Wochen | [ ] |
| Post-Deploy Smoke-Test im CI/CD einbauen | @platform | 1 Woche | [ ] |
| Status-Page Update in Incident-Checkliste aufnehmen | @sre-lead | Diese Woche | [ ] |
| Runbook für DB-Migrations-Rollback erstellen | @dba | 2 Wochen | [ ] |

## Lessons Learned
[Was nehmen wir mit? Systemisch, nicht personenbezogen.]
```

---

## Toil-Tracking und -Reduktion

```markdown
## Toil-Klassifizierung

Toil = manuell, repetitiv, automatisierbar, taktisch, ohne dauerhaften Wert

TOIL-QUELLEN (typisch):
- Manuelle Deployments (→ Automatisieren mit CI/CD)
- Regelmäßige Passwort-Rotationen (→ Secrets Manager + Auto-Rotation)
- Manuelle Skalierung (→ HPA / KEDA)
- Ticket-Bearbeitung wiederkehrender Art (→ Self-Service-Portal)
- Ad-hoc-Berichte (→ Grafana-Dashboard + Scheduled Reports)
- Manuelle Backup-Prüfung (→ Automatisches Monitoring)

MESSUNG:
- SRE-Team: max. 50% der Zeit für Toil (Google-Empfehlung)
- Toil-Tracking: eigene Kategorie im Ticket-System
- Quarterly Review: welcher Toil nimmt zu?

ZIEL: Jede Toil-Aufgabe innerhalb 2 Quartalen automatisieren oder eliminieren.
```

---

## Capacity Planning Vorlage

```python
#!/usr/bin/env python3
"""Einfaches Capacity-Planning-Skript basierend auf Prometheus-Metriken."""
import requests
from datetime import datetime, timedelta

PROMETHEUS_URL = "http://prometheus:9090"

def query_prometheus(query: str, days: int = 30) -> float:
    """Durchschnittswert der letzten N Tage abfragen."""
    end = datetime.now()
    start = end - timedelta(days=days)
    resp = requests.get(f"{PROMETHEUS_URL}/api/v1/query_range", params={
        "query": query, "start": start.timestamp(),
        "end": end.timestamp(), "step": "1h"
    })
    values = [float(v[1]) for v in resp.json()["data"]["result"][0]["values"]]
    return sum(values) / len(values) if values else 0

# CPU-Wachstumsrate berechnen
cpu_30d  = query_prometheus('avg(rate(container_cpu_usage_seconds_total[1h]))', 30)
cpu_7d   = query_prometheus('avg(rate(container_cpu_usage_seconds_total[1h]))', 7)
growth   = (cpu_7d - cpu_30d) / cpu_30d * 100 if cpu_30d > 0 else 0

print(f"CPU-Auslastung (30d Schnitt): {cpu_30d:.2%}")
print(f"CPU-Auslastung (7d Schnitt):  {cpu_7d:.2%}")
print(f"Wachstumstrend:               {growth:+.1f}%")
print(f"Prognostiziert in 90 Tagen:   {cpu_7d * (1 + growth/100 * 3):.2%}")
```
