[README.md](https://github.com/user-attachments/files/29453873/README.md)
# IT Skill Library

# 🗂️ IT Skill Library

> **10 spezialisierte Claude-Skills für IT, Security und Compliance** — entwickelt für den DACH-Raum mit Fokus auf deutsche Regulierung (NIS2, DSGVO, BSI IT-Grundschutz, ISO 27001, DORA, IEC 62443).

[![Skills](https://img.shields.io/badge/Skills-10-blue)](#skills-übersicht)
[![Module](https://img.shields.io/badge/Referenzmodule-86-green)](#)
[![Sprache](https://img.shields.io/badge/Outputs-Deutsch%20%2F%20Englisch-orange)](#)
[![Lizenz](https://img.shields.io/badge/Lizenz-MIT-lightgrey)](LICENSE)

---

## 🚀 Quickstart

### 1. Skill herunterladen

Alle Skills sind als `.skill`-Pakete in den [Releases](../../releases) verfügbar.

### 2. In Claude installieren

```
Einstellungen → Skills → Skill hinzufügen → .skill-Datei hochladen
```

### 3. Skill-Navigator öffnen

👉 **[Interaktiver Skill-Navigator](https://your-username.github.io/it-skill-library/skill-navigator.html)** — findet den richtigen Skill für deine Situation.

---

## 📦 Skills Übersicht

| Skill | Module | Schwerpunkt | Outputs |
|-------|--------|-------------|---------|
| [🛡️ IT Comprehensive Security](skills/it-comprehensive-security/) | 24 | Pentesting, IR, SIEM, Healthcare, Finance, Energy, Pharma, OT | Reports DE/EN, Playbooks, Scripts |
| [🪟 M365 Admin](skills/m365-admin-skill/) | 13 | Entra ID, Intune, Exchange, SharePoint, PAM, Defender | PowerShell, Anleitungen DE |
| [⚙️ DevOps & CI/CD](skills/devops-cicd-skill/) | 9 | GitHub Actions, Terraform, GitOps, SRE, Supply Chain | Pipeline YAML, HCL, Runbooks |
| [🐧 Linux Sysadmin](skills/linux-sysadmin-skill/) | 8 | Bash, systemd, Härtung, Ansible, Monitoring | Bash-Scripts, Ansible-Playbooks |
| [💾 Backup & DR](skills/backup-dr-skill/) | 4 | Veeam, Cloud-Backup, DR-Planung, BCP | Konzepte DE, Scripts, Protokolle |
| [🌐 Network Engineering](skills/network-engineering-skill/) | 6 | Cisco, BGP/OSPF, Palo Alto, FortiGate, Design | CLI-Snippets, ASCII-Diagramme |
| [📋 IT Compliance](skills/it-compliance-skill/) | 9 | DSGVO, BSI IT-Grundschutz, ISO 27001, NIS2, KRITIS | Deutsche Vorlagen, Checklisten |
| [⚡ PowerShell](skills/powershell-skill/) | 5 | AD, Graph API, JEA, Code-Signing, Automatisierung | PS1-Scripts, Module, HTML-Reports |
| [☁️ Cloud Architecture](skills/cloud-architecture-skill/) | 5 | Well-Architected, FinOps, K8s Security, Serverless | Terraform HCL, Matrizen |
| [🏭 OT/ICS Security](skills/ot-ics-skill/) | 3 | OT-Protokolle, IEC 62443, Nozomi/Claroty, KRITIS | Sicherheitskonzepte, IR-Playbooks |

---

## 🎯 Anwendungsfälle

### Nach Situation

| Ich brauche... | Empfohlene Skills |
|----------------|-------------------|
| NIS2 Gap-Assessment | IT Compliance + IT Comprehensive Security |
| Zero Trust Rollout | IT Comprehensive Security + M365 Admin + Network Engineering |
| Ransomware-Schutz | Backup & DR + IT Comprehensive Security + Network Engineering |
| DevSecOps Pipeline | DevOps & CI/CD + Cloud Architecture + IT Comprehensive Security |
| KRITIS Energie/Wasser | OT/ICS Security + IT Comprehensive Security + IT Compliance |
| ISO 27001 Audit | IT Compliance + IT Comprehensive Security + PowerShell |
| Krankenhaus IT-Sicherheit | IT Comprehensive Security + IT Compliance + Backup & DR |
| Linux-Server-Härtung | Linux Sysadmin + Backup & DR |
| M365 Tenant-Administration | M365 Admin + PowerShell |
| Cloud Landing Zone | Cloud Architecture + DevOps & CI/CD |

---

## 🗺️ Regulatorische Abdeckung (DACH)

| Standard / Regulation | Skills |
|-----------------------|--------|
| **NIS2 / NIS2UmsuCG** | IT Compliance, IT Comprehensive Security, OT/ICS |
| **DSGVO / BDSG** | IT Compliance, M365 Admin |
| **BSI IT-Grundschutz** | IT Compliance, IT Comprehensive Security |
| **ISO 27001:2022** | IT Compliance, IT Comprehensive Security |
| **KRITIS-Dachgesetz** | IT Compliance, IT Comprehensive Security, OT/ICS |
| **DORA** | IT Comprehensive Security (Finance-Modul) |
| **BAIT / VAIT / MaRisk** | IT Comprehensive Security (Finance-Modul) |
| **IEC 62443** | OT/ICS Security, IT Comprehensive Security |
| **EU GMP Annex 11 / GAMP 5** | IT Comprehensive Security (Pharma-Modul) |
| **TISAX / VDA ISA** | IT Comprehensive Security (Manufacturing-Modul) |
| **PCI DSS v4.0** | IT Comprehensive Security (Finance-Modul) |
| **EU MDR / IVDR** | IT Comprehensive Security (Healthcare-Modul) |

---

## 📁 Repository-Struktur

```
it-skill-library/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── skill-navigator.html          # Interaktiver Navigator (GitHub Pages)
├── skills/
│   ├── it-comprehensive-security/
│   │   ├── SKILL.md              # Skill-Konfiguration
│   │   └── references/           # Referenzmodule (Markdown)
│   │       ├── cloud-security.md
│   │       ├── healthcare-security.md
│   │       └── ...
│   ├── m365-admin-skill/
│   ├── devops-cicd-skill/
│   ├── linux-sysadmin-skill/
│   ├── backup-dr-skill/
│   ├── network-engineering-skill/
│   ├── it-compliance-skill/
│   ├── powershell-skill/
│   ├── cloud-architecture-skill/
│   └── ot-ics-skill/
└── .github/
    ├── ISSUE_TEMPLATE/
    │   ├── skill-request.md
    │   └── bug-report.md
    └── workflows/
        └── validate-skills.yml   # CI: Skill-Validierung
```

---

## 🤝 Beitragen

Feedback, Korrekturen und neue Module sind willkommen! Bitte lies zuerst [CONTRIBUTING.md](CONTRIBUTING.md).

**Schnell beitragen:**
- 🐛 [Bug melden](../../issues/new?template=bug-report.md)
- 💡 [Neuen Skill vorschlagen](../../issues/new?template=skill-request.md)
- 📝 [Referenzmodul verbessern](../../pulls)

---

## 📄 Lizenz

MIT License — siehe [LICENSE](LICENSE)

---

## 🏷️ Tags

`claude-skills` `it-security` `dach` `nis2` `dsgvo` `bsi-grundschutz` `iso27001` `kritis`
`m365` `azure` `devops` `linux` `backup` `network` `ot-security` `iec62443` `compliance`
