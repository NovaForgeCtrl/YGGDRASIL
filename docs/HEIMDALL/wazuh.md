# 🛡️ WAZUH SIEM — Der Wächter {: #wazuh-main }

> *"Open-Source Security Information and Event Management. Nichts entgeht dem wachsamen Auge."*

---

## 📋 Systeminformationen

| Attribut | Wert |
|----------|------|
| **Status** | 🟢 Aktiv |
| **Version** | 4.x |
| **Erstellt** | 2015 (OSSEC Fork) |
| **Zuletzt geändert** | 2026-07 |
| **Lizenz** | Open Source (GPLv2) |

---

## 🎯 Ziel / Zweck

Wazuh ist eine Open-Source Security-Plattform, die SIEM-Funktionalität, Host-based Intrusion Detection (HIDS), Log-Analyse, File Integrity Monitoring (FIM), Schwachstellen-Erkennung und Compliance-Überwachung in einem einzigen System vereint.

---

## 🛠️ Komponenten

[Wazuh Agent](wazuh-agent.md){: .world-link } · [Wazuh Manager](wazuh-manager.md){: .world-link } · [Wazuh Indexer](wazuh-indexer.md){: .world-link } · [Wazuh Dashboard](wazuh-dashboard.md){: .world-link }

---

## 🏗️ Architektur

```
┌─────────────────┐     ┌─────────────────┐
│  Wazuh Agent    │────▶│  Wazuh Manager  │
│  (Endpoint)     │     │  (SIEM Engine)  │
└─────────────────┘     └────────┬────────┘
                                 │
                    ┌────────────┴────────────┐
                    ▼                         ▼
           ┌─────────────────┐      ┌─────────────────┐
           │  Wazuh Indexer  │      │ Wazuh Dashboard │
           │  (Datenbank)    │      │  (Web-UI)       │
           └─────────────────┘      └─────────────────┘
```

---

## 🔍 Hauptfunktionen

### Log-Analyse

Wazuh sammelt und analysiert Logs aus verschiedensten Quellen:

- System-Logs (Linux: syslog, auth.log / Windows: Event Log)
- Anwendungs-Logs (Webserver, Datenbanken, Container)
- Netzwerk-Logs (Firewall, IDS wie Suricata)
- Cloud-Logs (AWS, Azure, GCP)

### File Integrity Monitoring (FIM)

Überwacht kritische Dateien und Verzeichnisse auf Veränderungen:

- `/etc/passwd`, `/etc/shadow` (Linux)
- `C:\Windows\System32` (Windows)
- Webserver-Verzeichnisse
- Konfigurationsdateien

Erkennt: Manipulationen, Backdoors, unautorisierte Änderungen

### Intrusion Detection (HIDS)

Erkennt verdächtige Aktivitäten auf Host-Ebene:

- Brute-Force-Angriffe (SSH, RDP)
- Privilege Escalation (sudo, su)
- Malware-Indikatoren
- Rootkits
- Ungewöhnliche Prozessaktivitäten

### Vulnerability Detection

Scannt Systeme auf bekannte Schwachstellen:

- CVE-Datenbank-Abgleich
- Betriebssystem- und Software-Versionen prüfen
- Priorisierung nach CVSS-Score

### Active Response

Automatisierte Gegenmaßnahmen bei erkannten Bedrohungen:

- IP-Blocking via Firewall (iptables, Windows Firewall)
- Benutzer sperren
- Prozesse beenden
- Benutzerdefinierte Skripte ausführen

### Compliance-Überwachung

Unterstützt gängige Security-Frameworks:

| Framework | Abdeckung |
|-----------|-----------|
| **NIS2** | Art. 21 Risikomanagement, Art. 23 Vorfallmeldung |
| **BSI IT-Grundschutz** | IND.2 Angriffserkennung, OPS.1.1.3 Logging |
| **ISO 27001** | Zugriffskontrolle, Audit, Änderungsmanagement |
| **PCI-DSS** | Authentifizierung, Protokollierung, Integrität |
| **HIPAA** | Zugriffskontrolle, Audit-Trails |
| **GDPR** | Datenschutz, Protokollierung |

---

## 📊 Architektur-Varianten

| Variante | Beschreibung |
|----------|--------------|
| **All-in-One** | Alle Komponenten auf einem Server — ideal für Homelabs |
| **Distributed** | Manager + Indexer + Dashboard getrennt — skalierbar |
| **Cloud** | Docker / Kubernetes, AWS/Azure/GCP Marketplace |

---

## ✅ Vorteile

| Vorteil | Beschreibung |
|---------|--------------|
| **Open Source** | Keine Lizenzkosten, volle Transparenz |
| **Aktive Community** | Große Community, regelmäßige Updates |
| **Modular** | Erweiterbar durch eigene Regeln, Decoder, Active Response |
| **Plattformübergreifend** | Linux, Windows, macOS, Cloud |
| **Skalierbar** | Von Homelab bis Enterprise |
| **Integration** | Suricata, VirusTotal, AbuseIPDB, etc. |
| **Compliance-ready** | Vorgefertigte Regeln für NIS2, PCI-DSS, ISO 27001 |

---

## 📊 Vergleich mit anderen SIEMs

| Kriterium | Wazuh | Splunk | Elastic SIEM | QRadar |
|-----------|-------|--------|--------------|--------|
| **Kosten** | Kostenlos | Sehr teuer | Teuer | Sehr teuer |
| **Lernkurve** | Mittel | Steil | Steil | Steil |
| **Open Source** | ✅ Ja | ❌ Nein | Teilweise | ❌ Nein |
| **FIM** | ✅ Integriert | ✅ App nötig | ✅ Integriert | ✅ Integriert |
| **Vulnerability Scan** | ✅ Integriert | ❌ Extra | ❌ Extra | ❌ Extra |

---

## 🎯 Typische Use Cases

### 🏠 Homelab / Lernen

- Aufbau eines SOC-Labs
- Üben von Threat Hunting
- MITRE ATT&CK Simulationen

### 🏢 Kleine & Mittlere Unternehmen

- Zentrale Log-Sammlung
- Server-Überwachung
- Compliance-Nachweis

### 🌐 Managed Security Service Provider (MSSP)

- Multi-Tenant-Betrieb
- Überwachung hunderter Kunden-Endpoints
- White-Label Dashboards

### ☁️ Cloud Security

- Überwachung von AWS, Azure, GCP
- Container-Security (Docker, Kubernetes)
- Cloud-Trail-Analyse

---

## 📝 Notizen

!!! note "Wichtig"
    - System läuft stabil
    - Regelmäßige Updates durchführen
    - Monitoring aktiv
    - Backup der Konfiguration empfohlen

---

> *"Der Wächter schläft nie. Jeder Log-Eintrag erzählt eine Geschichte — man muss nur lernen, sie zu lesen."*
