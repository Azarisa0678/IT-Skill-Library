# Disaster Recovery & Business Continuity — Referenzmodul

DR-Strategien, BIA, RTO/RPO-Berechnung, DR-Testplanung, Notfallhandbuch-Vorlagen.

---

## Business Impact Analysis (BIA)

### BIA-Vorlage

```markdown
## Business Impact Analysis — [Unternehmen] — [Datum]

### Kritikalitätsskala
| Stufe | Name        | RTO-Ziel  | RPO-Ziel  | Beschreibung |
|-------|-------------|-----------|-----------|--------------|
| 1     | Kritisch    | ≤ 4 Std   | ≤ 1 Std   | Sofortiger Umsatz-/Patientenverlust, Betriebsunfähigkeit |
| 2     | Wichtig     | ≤ 8 Std   | ≤ 4 Std   | Erhebliche Beeinträchtigung, Workaround möglich |
| 3     | Standard    | ≤ 24 Std  | ≤ 24 Std  | Betrieb eingeschränkt aber möglich |
| 4     | Niedrig     | ≤ 72 Std  | ≤ 24 Std  | Kein sofortiger Betriebseinfluss |

### Systemkatalog (Vorlage)
| System | Prozess | Nutzer | Kritikalität | RTO | RPO | Abhängigkeiten | Notfallkontakt |
|--------|---------|--------|--------------|-----|-----|----------------|----------------|
| ERP (SAP) | Auftragsverarbeitung, Buchhaltung | 150 | 1 | 4h | 1h | AD, SQL, Storage | SAP-Admin: +49... |
| AD / DNS | Alle Windows-Systeme | 500 | 1 | 2h | 0h | - | IT-Leiter: +49... |
| E-Mail (Exchange) | Kommunikation | 500 | 2 | 8h | 4h | AD | Exchange-Admin: +49... |
| Dateiserver | Dateiablage | 300 | 2 | 8h | 4h | AD, Storage | IT-Admin: +49... |
| CRM | Kundenbetreuung | 50 | 2 | 8h | 4h | AD, SQL | CRM-Admin: +49... |
| WLAN | Mobiles Arbeiten | 200 | 3 | 24h | - | AD | Netz-Admin: +49... |
| Intranet | Informationsportal | 500 | 4 | 72h | 24h | Webserver | IT-Admin: +49... |
```

---

## DR-Strategien nach Kosten/RTO

```
STRATEGIE 1: COLD STANDBY (günstigste Option)
  RTO: 24-72 Stunden
  RPO: Letztes Backup-Intervall
  Beschreibung: Hardware/Cloud-Ressourcen erst bei Disaster bereitstellen
  Kosten: ★☆☆☆☆ (nur Backup-Storage)
  Geeignet für: Tier-3/4-Systeme, KMU ohne DR-Budget
  
  Umsetzung:
  - Veeam Backup auf Offsite-Repo / Cloud
  - Im Disaster: neue VMs in Azure/AWS anlegen, Backup restoren
  - Dokumentation: welche VMs in welcher Reihenfolge!

STRATEGIE 2: WARM STANDBY
  RTO: 4-8 Stunden
  RPO: 1-4 Stunden
  Beschreibung: DR-Systeme im Standby, werden bei Bedarf aktiviert
  Kosten: ★★★☆☆ (laufende Kosten für DR-Infrastruktur)
  Geeignet für: Tier-1/2-Systeme, mittlere Unternehmen
  
  Umsetzung:
  - Veeam Replication: stündliche VM-Replikation zu DR-Site
  - Azure Site Recovery (ASR): kontinuierliche Replikation
  - Failover-Test: monatlich (isoliertes Test-Failover)

STRATEGIE 3: HOT STANDBY / ACTIVE-PASSIVE
  RTO: 1-4 Stunden  
  RPO: Minuten bis 0
  Beschreibung: DR-Systeme laufen parallel, synchrone Replikation
  Kosten: ★★★★☆ (doppelte Infrastruktur)
  Geeignet für: Tier-1-kritische Systeme, Banken, Krankenhäuser
  
  Umsetzung:
  - SQL Server Always On Availability Groups
  - Storage Replikation (NetApp SnapMirror, EMC SRDF)
  - Azure ASR mit synchroner Replikation

STRATEGIE 4: ACTIVE-ACTIVE / MULTI-SITE
  RTO: Sekunden (automatisches Failover)
  RPO: 0
  Beschreibung: Beide Sites aktiv, Load Balancing
  Kosten: ★★★★★ (sehr hoch)
  Geeignet für: Banking, e-Commerce, kritische Infrastruktur
  
  Umsetzung:
  - Global Load Balancer (Azure Traffic Manager, AWS Route 53)
  - Datenbankcluster multi-region
  - Anwendung muss aktiv-aktiv-fähig sein (zustandslos bevorzugt)
```

---

## Azure Site Recovery (ASR) — Konfiguration

```powershell
# Azure Site Recovery: On-Premises VMware → Azure

# 1. Recovery Services Vault (für ASR)
$vault = New-AzRecoveryServicesVault `
    -Name "rsv-asr-prod" `
    -ResourceGroupName "rg-dr" `
    -Location "westeurope"

# 2. Replikationsrichtlinie (RPO 4 Stunden, Retention 72 Stunden)
$policy = New-AzRecoveryServicesAsrPolicy `
    -Name "asr-policy-prod" `
    -ReplicationProvider "HyperVReplicaAzure" `
    -ReplicationFrequencyInSeconds 14400 `  # 4 Stunden
    -RecoveryPoints 72 `
    -ApplicationConsistentSnapshotFrequencyInHours 4

# 3. Test-Failover (WICHTIG: immer testen!)
$vm = Get-AzRecoveryServicesAsrReplicationProtectedItem -Name "vm-erp-01"
$testFailoverJob = Start-AzRecoveryServicesAsrTestFailoverJob `
    -ReplicationProtectedItem $vm `
    -Direction PrimaryToRecovery `
    -AzureVMNetworkId "/subscriptions/.../virtualNetworks/vnet-dr-test"

# Testen → Applikation prüfen → Cleanup
Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $vm
```

---

## DR-Testplan

### Jahresplan DR-Tests

```markdown
## DR-Testplan [Jahr]

### Teststufen

| Stufe | Typ              | Häufigkeit | Dauer     | Risiko | Beteiligte |
|-------|-----------------|------------|-----------|--------|------------|
| 1     | Tabletop-Übung  | Quartalsweise | 2-4h   | Keins  | IT + Management |
| 2     | Komponenten-Test | Monatlich  | 1-2h    | Niedrig | IT-Team |
| 3     | Isolierter Failover-Test | Quartalsweise | 4-8h | Niedrig | IT + Fachbereiche |
| 4     | Vollständiger DR-Test | Jährlich | 1-2 Tage | Mittel | Alle |

### Quartalsplan

**Q1 — Januar/Februar:**
- [ ] Tabletop: Szenario "Ransomware verschlüsselt alle Dateiserver"
- [ ] Komponenten-Test: Restore einzelner VMs aller Tier-1-Systeme
- [ ] Doku-Review: Notfallhandbuch aktuell?

**Q2 — April/Mai:**
- [ ] Tabletop: Szenario "Rechenzentrum-Brand, RZ nicht zugänglich"
- [ ] Isolierter Failover: ERP auf DR-Site (ohne Produktionsunterbrechung)
- [ ] Backup-Validierung: Hash-Prüfung aller Backup-Sätze

**Q3 — Juli/August:**
- [ ] Tabletop: Szenario "ISP-Ausfall, keine Internetverbindung"
- [ ] Komponenten-Test: Active Directory Recovery (Tombstone-Restore)
- [ ] Netzwerk-DR: MPLS-Failover auf LTE-Backup testen

**Q4 — Oktober/November:**
- [ ] VOLLSTÄNDIGER DR-TEST (siehe Vorlage unten)
- [ ] Lessons-Learned-Workshop
- [ ] Jahres-Review: RTO/RPO-Ziele noch realistisch?
```

### Vollständiger DR-Test — Protokoll-Vorlage

```markdown
## DR-Test-Protokoll — Vollständiger Failover
Datum:           [TT.MM.JJJJ]
Testleiter:      [Name, Rolle]
Teilnehmer:      [Liste aller Beteiligten]
Testszenario:    [z.B. "Primäres RZ vollständig ausgefallen, kein Zugang"]
Testfenster:     [Beginn] — [Ende]
Produktionseinfluss: [Keiner / Eingeschränkt / Wartungsfenster]

---

### Phase 1: Vorbereitung
| Zeit | Schritt | Verantwortlich | Status |
|------|---------|----------------|--------|
| -30min | Backup-Status prüfen (alle Jobs erfolgreich?) | Backup-Admin | [ ] |
| -30min | DR-Umgebung auf Bereitschaft prüfen | DR-Admin | [ ] |
| -30min | Kommunikation an Nutzer (falls nötig) | IT-Leiter | [ ] |
| -15min | Krisenstab-Kurzmeeting: Go/No-Go | Testleiter | [ ] |

### Phase 2: Failover-Ausführung
| Zeit | System | Soll-Schritt | Ist-Ergebnis | RTO-Ziel | Tatsächlich | OK? |
|------|--------|--------------|--------------|----------|-------------|-----|
| 00:00 | AD (2x DC) | Restore aus Backup oder Failover | | 2h | | [ ] |
| 00:30 | DNS | Zeigt auf DR-Site | | 30min | | [ ] |
| 01:00 | SQL Server | Always-On Failover / Restore | | 2h | | [ ] |
| 01:30 | ERP (SAP) | Start auf DR-Server, Anmeldung OK | | 4h | | [ ] |
| 02:00 | Exchange | Mailfluss auf DR-Infrastruktur | | 4h | | [ ] |
| 02:30 | Dateiserver | Zugriff auf letzte Daten | | 4h | | [ ] |
| 03:00 | VPN / Remote Access | Externe Nutzer können arbeiten | | 4h | | [ ] |

### Phase 3: Validierung
| Prüfpunkt | Soll | Ist | OK? |
|-----------|------|-----|-----|
| ERP: Bestellung anlegen | Vollständig möglich | | [ ] |
| E-Mail: Senden/Empfangen | Vollständig möglich | | [ ] |
| Dateiserver: Lesen/Schreiben | Vollständig möglich | | [ ] |
| VPN: Remote-Zugang | Vollständig möglich | | [ ] |
| Max. Datenverlust (RPO) | ≤ [X] Stunden | [Tatsächlich] Stunden | [ ] |

### Phase 4: Rückfailover (falls Test-Failover)
| Zeit | Schritt | Status |
|------|---------|--------|
| | Rückfailover auf primäre Site | [ ] |
| | Datenkonsistenz prüfen | [ ] |
| | Produktionssysteme wieder aktiv | [ ] |

### Ergebnis
Gesamtdauer Failover:    [X Stunden Y Minuten]
RTO-Ziel erreicht:       [Ja / Nein]
RPO-Ziel erreicht:       [Ja / Nein]
Kritische Mängel:        [Beschreibung oder "Keine"]
Empfehlungen:            [Liste]
Nächster vollständiger DR-Test: [Datum]

Unterschrift Testleiter: _________________ Datum: _______
Unterschrift IT-Leiter:  _________________ Datum: _______
```

---

## Notfallhandbuch (Runbook) — Vorlage

```markdown
# IT-Notfallhandbuch — [Unternehmen]
Version: [X.Y] | Letzte Aktualisierung: [Datum] | Verantwortlich: [IT-Leiter]

## 1. Kontakte (immer aktuell halten!)

### Intern
| Rolle | Name | Mobil | Erreichbarkeit |
|-------|------|-------|----------------|
| IT-Leiter | [Name] | [Nummer] | 24/7 |
| Backup-Admin | [Name] | [Nummer] | 24/7 |
| Netzwerk-Admin | [Name] | [Nummer] | Werktage + Rufbereitschaft |
| Geschäftsführung | [Name] | [Nummer] | 24/7 bei Katastrophe |
| Datenschutzbeauftragter | [Name] | [Nummer] | Werktags |

### Extern
| Dienstleister | Kontakt | Telefon | Vertragsnummer | SLA |
|---------------|---------|---------|----------------|-----|
| ISP (Hauptleitung) | [Name] | [Nummer] | [Vertrag] | 4h |
| ISP (Backup-Leitung) | [Name] | [Nummer] | [Vertrag] | 8h |
| Veeam Support | support.veeam.com | +1-XXX | [Vertrag] | 24/7 |
| Microsoft Support | aka.ms/support | [Nummer] | [Vertrag] | 24/7 |
| Hardware-Lieferant | [Name] | [Nummer] | [Vertrag] | NBD |
| Externe IT-Notfall-Firma | [Firma] | [Nummer] | [Vertrag] | 4h |

## 2. Sofortmaßnahmen bei IT-Notfall

### Erste 15 Minuten:
1. Ausmaß feststellen: Einzelsystem / Netzwerksegment / Gesamtes RZ
2. IT-Leiter informieren (immer, auch nachts!)
3. Bei Cyberangriff-Verdacht: NICHTS löschen, NICHTS neustarten
4. Protokoll starten: Wer hat was wann gemacht (Zeitstempel!)
5. Entscheidung: DR-Plan aktivieren ja/nein?

### Eskalationsstufen:
Stufe 1 — Einzelsystem ausgefallen → IT-Admin eigenständig
Stufe 2 — Mehrere Systeme / Service-Ausfall → IT-Leiter informieren
Stufe 3 — Kritische Systeme / Produktionsausfall → Geschäftsführung + DR-Plan
Stufe 4 — RZ-Ausfall / Cyberangriff / Katastrophe → Krisenstab + BSI/Polizei

## 3. System-Restore-Prozeduren

### Active Directory (Domain Controller)
[Verweis auf: \\backup-srv\Runbooks\AD-Recovery.pdf]
Backup-Ort: [Pfad/URL]
Letzter Test: [Datum] | Tester: [Name]

### ERP-System
[Verweis auf: \\backup-srv\Runbooks\ERP-Recovery.pdf]
Backup-Ort: [Pfad/URL]
DB-Restore: [Anleitung oder Link]
Anwendungsstart: [Reihenfolge der Dienste]
Letzter Test: [Datum] | Tester: [Name]

### E-Mail (Exchange / Microsoft 365)
[Verweis auf: \\backup-srv\Runbooks\Exchange-Recovery.pdf]
Bei M365-Ausfall: [Notfallkommunikation über private Handys]

## 4. Checklisten im Notfall

→ [Verweis auf gedruckte Checklisten im Notfallkoffer]
Notfallkoffer Standort: [physischer Ort, z.B. "Serverraum Schrank 3, Fach 2"]
Inhalt: Ausgedruckte Runbooks, Zugangsdaten-Safe-Kombi, USB mit Offline-Tools

## 5. Kommunikationsplan

Intern: [Messenger-App außerhalb der eigenen Infrastruktur, z.B. Signal-Gruppe "IT-Notfall"]
Extern (Kunden): [Wer kommuniziert was? → immer IT-Leiter oder GF, nie Techniker direkt]
Presse: [Nur GF oder beauftragte Agentur]
Behörden: [BSI, Datenschutz → IT-Leiter + DSB]
```
