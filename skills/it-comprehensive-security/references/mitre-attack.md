# MITRE ATT&CK Quick Reference

## Tactics Overview (Enterprise)

| ID | Tactic | Description |
|----|--------|-------------|
| TA0001 | Initial Access | Gaining a foothold in the network |
| TA0002 | Execution | Running attacker-controlled code |
| TA0003 | Persistence | Maintaining access across restarts |
| TA0004 | Privilege Escalation | Gaining higher-level permissions |
| TA0005 | Defense Evasion | Avoiding detection |
| TA0006 | Credential Access | Stealing credentials |
| TA0007 | Discovery | Enumerating the environment |
| TA0008 | Lateral Movement | Moving through the network |
| TA0009 | Collection | Gathering data of interest |
| TA0010 | Exfiltration | Stealing data out of the network |
| TA0011 | Command & Control | Communicating with compromised systems |
| TA0040 | Impact | Disrupting availability or integrity |

---

## High-Value Techniques by Tactic

### TA0001 — Initial Access
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| Phishing attachment | T1566.001 | Macro-enabled Office docs | Email gateway, sandboxing |
| Phishing link | T1566.002 | Evilginx, GoPhish | Proxy logs, DNS |
| Valid accounts | T1078 | Stolen creds | Failed/unusual logons |
| Exploit public-facing app | T1190 | SQLmap, Metasploit | WAF, IDS alerts |
| Supply chain compromise | T1195 | SolarWinds-style | Software integrity checks |

### TA0002 — Execution
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| PowerShell | T1059.001 | Empire, Cobalt Strike | Script Block Logging, Event 4104 |
| Windows Command Shell | T1059.003 | cmd.exe | Process creation Event 4688 |
| Scheduled Task | T1053.005 | schtasks.exe | Event 4698 |
| WMI | T1047 | wmic.exe | WMI activity logging |
| MSBuild / LOLBins | T1127 | msbuild.exe | Unusual parent-child processes |

### TA0003 — Persistence
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| Registry Run Keys | T1547.001 | reg.exe | Registry monitoring |
| Scheduled Task | T1053.005 | schtasks | Event 4698/4702 |
| New service | T1543.003 | sc.exe | Event 7045 |
| Web shell | T1505.003 | China Chopper, ASPX shells | Unusual web server child processes |
| Account creation | T1136 | net user | Event 4720 |

### TA0004 — Privilege Escalation
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| Token impersonation | T1134 | Incognito, Mimikatz | Unusual token usage |
| Bypass UAC | T1548.002 | UACME | Event 4688 with auto-elevated processes |
| Exploit vulnerable service | T1068 | Local privilege exploits | Patch management, EDR |
| Sudo / SUID abuse | T1548.003 | GTFOBins | Auditd, SUID file monitoring |

### TA0005 — Defense Evasion
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| Obfuscated files | T1027 | Invoke-Obfuscation | Script Block Logging |
| Disable security tools | T1562.001 | taskkill, reg | Tamper protection alerts |
| Masquerading | T1036 | Rename malware to svchost | Hash + path validation |
| Process injection | T1055 | Cobalt Strike, Metasploit | EDR behavioral detection |
| AMSI bypass | T1562.001 | Memory patching | AMSI telemetry, Event 1116 |

### TA0006 — Credential Access
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| LSASS dump | T1003.001 | Mimikatz, ProcDump | Event 4656/4663 on lsass; Defender |
| Kerberoasting | T1558.003 | Rubeus, GetUserSPNs | Event 4769 with RC4 encryption |
| AS-REP Roasting | T1558.004 | Rubeus | Event 4768 with no pre-auth |
| Password spraying | T1110.003 | Spray, CrackMapExec | Multiple 4625 across accounts |
| NTLM hash capture | T1557.001 | Responder | Network anomalies, LLMNR/NBNS traffic |

### TA0007 — Discovery
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| Network scanning | T1046 | nmap, masscan | IDS, unusual ICMP/SYN traffic |
| AD enumeration | T1087.002 | BloodHound, ldapdomaindump | LDAP query volume |
| File/share discovery | T1135 | net view, PowerView | SMB access logs |
| System info discovery | T1082 | systeminfo, WMI | Process creation logs |

### TA0008 — Lateral Movement
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| Pass-the-Hash | T1550.002 | Mimikatz, CrackMapExec | Event 4624 type 3 with NTLM |
| Pass-the-Ticket | T1550.003 | Rubeus | Event 4768/4769 anomalies |
| PsExec / SMB | T1021.002 | Impacket, Sysinternals | Event 7045, SMB named pipes |
| WinRM / PS Remoting | T1021.006 | Evil-WinRM | Event 4624 type 3, WinRM logs |
| RDP | T1021.001 | xfreerdp, mstsc | Event 4624 type 10 |

### TA0010 — Exfiltration
| Technique | ID | Detection |
|-----------|-----|-----------|
| Exfil over C2 channel | T1041 | Unusual outbound traffic volume |
| Exfil over web service | T1567 | Traffic to Pastebin, Dropbox, GitHub |
| DNS exfiltration | T1048.003 | High volume DNS TXT queries |
| Compressed/encrypted archive | T1560 | rar/7z/zip creation before transfer |

### TA0040 — Impact
| Technique | ID | Common Tools | Detection |
|-----------|-----|-------------|-----------|
| Ransomware / data encryption | T1486 | LockBit, Conti | Mass file rename, shadow copy deletion |
| Delete shadow copies | T1490 | vssadmin delete shadows | Event 524, vssadmin process |
| Defacement | T1491 | Web shell | Web server file integrity |
| DoS | T1498 | Various | Network traffic anomalies |

---

## ATT&CK Mapping to Skill Sections

| Skill Section | Relevant ATT&CK Tactics |
|---------------|------------------------|
| Penetration Testing | TA0001, TA0002, TA0003, TA0004, TA0006, TA0008 |
| Incident Response | All tactics (detection & response) |
| Hardening / Defensive | TA0003, TA0004, TA0005, TA0006 (prevention) |
| PowerShell Scripting | T1059.001, T1562.001, T1003.001, T1053.005 |
| DevSecOps | T1195 (supply chain), T1552 (credentials in code) |
| Phishing / Social Engineering | TA0001 (T1566) |

---

## Detection Coverage Checklist

Use this to assess your detection gaps:

- [ ] Initial Access: Email gateway + sandbox for attachments
- [ ] Execution: Script Block Logging, Process creation (Event 4688)
- [ ] Persistence: Registry monitoring, Event 4698/7045
- [ ] Privilege Escalation: UAC events, local exploit detection (EDR)
- [ ] Defense Evasion: AMSI telemetry, tamper protection
- [ ] Credential Access: LSASS protection, Kerberos event monitoring
- [ ] Discovery: LDAP query alerting, network scan detection
- [ ] Lateral Movement: Event 4624 type analysis, SMB monitoring
- [ ] Exfiltration: DLP, unusual outbound volume alerting
- [ ] Impact: File integrity monitoring, shadow copy alerting

---

## Useful ATT&CK Resources

- Navigator: https://mitre-attack.github.io/attack-navigator/
- Full technique list: https://attack.mitre.org/techniques/enterprise/
- D3FEND (defensive countermeasures): https://d3fend.mitre.org/
- ATT&CK for ICS: https://attack.mitre.org/matrices/ics/
