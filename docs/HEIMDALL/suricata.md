# 🐉 SURICATA — Der Drache {: #suricata-main }

> *"High Performance Network IDS, IPS and Network Security Monitoring Engine. Der Drache wacht über jedes Paket."*

---

## 📋 Systeminformationen

| Attribut | Wert |
|----------|------|
| **Status** | 🟢 Aktiv |
| **Version** | 7.x |
| **Erstellt** | 2009 (OISF) |
| **Zuletzt geändert** | 2026-07 |
| **Lizenz** | Open Source (GPLv2) |

---

## 🎯 Ziel / Zweck

Suricata ist eine Open-Source Network Intrusion Detection System (NIDS), Intrusion Prevention System (NIPS) und Network Security Monitoring (NSM) Engine. Sie analysiert Netzwerkverkehr in Echtzeit anhand von Signaturen, erkennt Angriffe, protokolliert verdächtige Aktivitäten und kann im Inline-Modus aktiv blockieren.

---

## 🛠️ Komponenten

[Suricata Agent](suricata-agent.md){: .world-link } · [Rule Engine](suricata-rules.md){: .world-link } · [EVE JSON](suricata-eve.md){: .world-link } · [File Extraction](suricata-files.md){: .world-link }

---

## 🏗️ Architektur

```
┌─────────────────────────────────────────┐
│           Suricata Engine               │
├─────────────┬─────────────┬─────────────┤
│   Packet    │   Protocol  │   File      │
│   Decoder   │   Detection │   Extraction│
├─────────────┼─────────────┼─────────────┤
│   Rule      │   EVE JSON  │   Lua       │
│   Engine    │   Output    │   Scripting │
└─────────────┴─────────────┴─────────────┘
              │
              ▼
        ┌─────────────┐
        │  Network    │
        │  Interface  │
        └─────────────┘
```

---

## 🔍 Hauptfunktionen

### Network Intrusion Detection (NIDS)

Überwacht Netzwerkverkehr passiv und alarmiert bei verdächtigen Mustern:

- Port Scans (Nmap, Masscan)
- Exploit-Versuche
- Malware-Kommunikation (C2)
- Brute-Force-Angriffe
- Datenexfiltration

### Intrusion Prevention (NIPS)

Im Inline-Modus kann Suricata aktiv eingreifen:

- Verbindungen blockieren (DROP)
- Pakete zurückweisen (REJECT)
- Verbindungen resetten (RST)

> **Wichtig:** IPS-Modus erfordert, dass Suricata direkt im Datenpfad sitzt.

### Network Security Monitoring (NSM)

Protokolliert den gesamten Netzwerkverkehr zur späteren Analyse:

- Wer hat wann mit wem kommuniziert?
- Welche Domains wurden abgefragt?
- Welche TLS-Zertifikate wurden ausgetauscht?
- Wie groß waren die Datenübertragungen?

### TLS/SSL Inspection

Analyse verschlüsselter Verbindungen:

- **JA3-Fingerprinting** — Erkennung von TLS-Clients
- **JA3S-Fingerprinting** — Erkennung von TLS-Servern
- Erkennung von verdächtigen Zertifikaten
- TLS-Version und Cipher-Suite-Analyse

### File Reconstruction

Rekonstruiert übertragene Dateien aus dem Netzwerk:

- HTTP-Downloads
- E-Mail-Anhänge (SMTP)
- SMB-Dateiübertragungen
- Automatische Hash-Berechnung (MD5, SHA1, SHA256)

---

## 📊 Betriebsmodi

| Modus | Beschreibung |
|-------|--------------|
| **IDS Mode** | Passiv über SPAN-Port/TAP — nur überwachen, nicht blockieren |
| **IPS Mode** | Inline im Datenpfad — aktiv blockieren, DROP/REJECT |

---

## 📊 Rule-Ökosystem

| Rule-Set | Beschreibung | Kosten |
|----------|--------------|--------|
| **Emerging Threats Open** | Größtes kostenloses Rule-Set, täglich aktualisiert | Kostenlos |
| **Emerging Threats Pro** | Erweiterte Regeln, schnellere Updates | Lizenz |
| **Snort Community Rules** | Offizielle Snort-Regeln (Suricata-kompatibel) | Kostenlos |
| **Talos VRT Rules** | Cisco Talos Rules | Registrierung |
| **Eigene Regeln** | Benutzerdefinierte Signaturen | Kostenlos |

---

## ✅ Vorteile

| Vorteil | Beschreibung |
|---------|--------------|
| **Open Source** | Keine Lizenzkosten, volle Transparenz |
| **Multi-Threaded** | Nutzt alle CPU-Kerne, hohe Performance |
| **Snort-kompatibel** | Bestehende Snort-Regeln übernehmbar |
| **EVE JSON** | Einheitliches, maschinenlesbares Log-Format |
| **Protokoll-Erkennung** | Automatische Erkennung von 20+ Protokollen |
| **File Extraction** | Dateien aus dem Netzwerk rekonstruieren |
| **JA3-Fingerprinting** | Erkennung von TLS-Clients/Servern |
| **Lua Scripting** | Dynamische, komplexe Detektionslogik |
| **Aktive Community** | Regelmäßige Updates, große Community |

---

## 📊 Vergleich mit anderen IDS/IPS

| Kriterium | Suricata | Snort | Zeek | OSSEC |
|-----------|----------|-------|------|-------|
| **Typ** | NIDS/NIPS | NIDS/NIPS | NSM/Analyzer | HIDS |
| **Open Source** | ✅ Ja | ✅ Ja | ✅ Ja | ✅ Ja |
| **Multi-Threaded** | ✅ Ja | ❌ Nein | ✅ Ja | N/A |
| **EVE JSON** | ✅ Ja | ❌ Nein | ✅ Ja | ❌ Nein |
| **File Extraction** | ✅ Ja | ⚠️ Begrenzt | ✅ Ja | ❌ Nein |
| **JA3 Support** | ✅ Ja | ⚠️ Plugin | ✅ Ja | ❌ Nein |
| **IPS Mode** | ✅ Ja | ✅ Ja | ❌ Nein | ❌ Nein |
| **Ideal für** | Enterprise IDS/IPS | Klassisches IDS | Deep Network Analysis | Host-Monitoring |

---

## 🎯 Typische Use Cases

### 🏠 Homelab / Lernen

- Aufbau eines IDS-Labs
- Verständnis für Netzwerkangriffe
- Üben von PCAP-Analyse
- Vorbereitung auf Security-Zertifizierungen

### 🏢 Unternehmensnetzwerk

- Überwachung des Internet-Gateways
- Erkennung von Malware-Kommunikation
- Erkennung von Port Scans und Exploit-Versuchen
- Compliance-Nachweis

### 🏭 ICS/SCADA-Umgebungen

- Überwachung industrieller Protokolle (Modbus, DNP3)
- Erkennung von Angriffen auf kritische Infrastruktur
- Netzwerksegmentierung überwachen

### ☁️ Cloud Security

- Überwachung von Cloud-Netzwerken (AWS VPC, Azure VNet)
- Container-Netzwerküberwachung
- East-West-Traffic-Analyse

### 🔬 Forensik & Incident Response

- Rekonstruktion von Angriffen anhand von Netzwerklogs
- File Extraction für Malware-Analyse
- PCAP-Korrelation mit Suricata-Alerts

---

## 📝 Notizen

!!! note "Wichtig"
    - IDS-Mode zuerst testen, bevor IPS aktiviert wird
    - Regelmäßige Rule-Updates durchführen
    - EVE JSON-Logs an SIEM weiterleiten
    - Backup der Konfiguration empfohlen

---

> *"Der Drache sieht jedes Paket. Kein Datenstrom entgeht seinem Feuer."*
