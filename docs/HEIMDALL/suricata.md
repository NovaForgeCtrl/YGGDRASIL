# 🐉 Suricata — Allgemeine Einführung

> *"High Performance Network IDS, IPS and Network Security Monitoring Engine"*

---

## Was ist Suricata?

**Suricata** ist eine Open-Source Network Intrusion Detection System (NIDS), Intrusion Prevention System (NIPS) und Network Security Monitoring (NSM) Engine. Sie wurde vom **Open Information Security Foundation (OISF)** entwickelt und ist ein direkter, leistungsstarker Konkurrent zu Snort — dem Klassiker unter den Open-Source IDS.

Suricata analysiert Netzwerkverkehr in Echtzeit anhand von Signaturen (Rules), erkennt Angriffe, protokolliert verdächtige Aktivitäten und kann im Inline-Modus sogar aktiv blockieren. Dank Multi-Threading nutzt es alle verfügbaren CPU-Kerne und ist damit deutlich schneller als ältere Single-Thread-Lösungen.

---

## Für wen ist Suricata gedacht?

| Zielgruppe | Einsatzzweck |
|-----------|--------------|
| **SOC-Teams** | Netzwerküberwachung, Erkennung von Angriffen im Datenverkehr |
| **Blue Teams** | Verteidigung, Netzwerk-Forensik, Threat Hunting |
| **Netzwerkadministratoren** | Überwachung des Netzwerkverkehrs, Erkennung von Anomalien |
| **Penetration Tester** | Validierung eigener Angriffe, Erkennung testen |
| **Homelab-Enthusiasten** | Lernen, Aufbau eines IDS-Labs, Verständnis für Netzwerkangriffe |
| **MSSP / SOC-as-a-Service** | Überwachung von Kundennetzwerken aus der Ferne |

---

## Kernkomponenten

Suricata besteht aus einer monolithischen Engine, die intern mehrere Module kombiniert:

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

### 1. Packet Decoder

Entschlüsselt Netzwerkpakete auf verschiedenen Ebenen:

- Ethernet, IP, TCP, UDP, ICMP
- VLAN, MPLS, GRE-Tunnel
- IPv6 nativ

### 2. Protocol Detection

Erkennt automatisch über 20 Protokolle:

- HTTP, HTTPS (TLS), DNS, SSH, FTP, SMTP, SMB
- Modbus, DNP3 (ICS/SCADA)
- QUIC, HTTP/2

### 3. Rule Engine

Vergleicht jede Verbindung mit tausenden Signaturen:

- Snort-kompatible Syntax
- Suricata-eigene Erweiterungen
- Thresholds, Flowbits, Content-Matching

### 4. EVE JSON Output

Das Herzstück der Logging-Funktion:

- Einheitliches JSON-Format für alle Events
- Strukturiert, maschinenlesbar, SIEM-freundlich
- Unterstützt alle Event-Typen (Alerts, HTTP, DNS, TLS, Flow, etc.)

### 5. File Extraction

Kann Dateien aus dem Netzwerkverkehr rekonstruieren:

- HTTP-Downloads, E-Mail-Anhänge
- TLS-verschlüsselte Dateien (mit Keys)
- Automatische Hash-Berechnung (MD5, SHA1, SHA256)

---

## Hauptfunktionen

### 🔍 Network Intrusion Detection (NIDS)

Überwacht Netzwerkverkehr passiv und alarmiert bei verdächtigen Mustern:

- Port Scans (Nmap, Masscan)
- Exploit-Versuche
- Malware-Kommunikation (C2)
- Brute-Force-Angriffe
- Datenexfiltration

### 🛡️ Intrusion Prevention (NIPS)

Im Inline-Modus kann Suricata aktiv eingreifen:

- Verbindungen blockieren (DROP)
- Pakete zurückweisen (REJECT)
- Verbindungen resetten (RST)

> **Wichtig:** IPS-Modus erfordert, dass Suricata direkt im Datenpfad sitzt (z. B. zwischen Firewall und Switch).

### 📊 Network Security Monitoring (NSM)

Protokolliert den gesamten Netzwerkverkehr zur späteren Analyse:

- Wer hat wann mit wem kommuniziert?
- Welche Domains wurden abgefragt?
- Welche TLS-Zertifikate wurden ausgetauscht?
- Wie groß waren die Datenübertragungen?

### 🔐 TLS/SSL Inspection

Analyse verschlüsselter Verbindungen ohne Entschlüsselung:

- **JA3-Fingerprinting** — Erkennung von TLS-Clients anhand des Handshake-Musters
- **JA3S-Fingerprinting** — Erkennung von TLS-Servern
- Erkennung von verdächtigen Zertifikaten, selbstsignierten Zertifikaten
- TLS-Version und Cipher-Suite-Analyse

### 🧠 Lua Scripting

Ermöglicht komplexe, dynamische Detektionslogik:

- Benutzerdefinierte Prüfungen
- Verbindung mit externen Datenbanken
- Erweiterte Entscheidungslogik über einfache Regeln hinaus

### 📁 File Reconstruction

Rekonstruiert übertragene Dateien aus dem Netzwerk:

- HTTP-Downloads
- E-Mail-Anhänge (SMTP)
- SMB-Dateiübertragungen
- Automatische Hash-Berechnung für Threat Intelligence-Abgleich

---

## Betriebsmodi

### IDS Mode (Passive)

```
Internet ──▶ Firewall ──▶ Switch ──▶ Internes Netz
                              │
                              ▼ (SPAN-Port / TAP)
                           Suricata (nur überwachen)
```

- Überwacht den Verkehr über einen SPAN-Port oder Netzwerk-TAP
- Alarmiert, blockiert aber nicht
- Kein Einfluss auf die Netzwerkperformance
- Ideal für erste Schritte und Analyse

### IPS Mode (Inline)

```
Internet ──▶ Firewall ──▶ Suricata ──▶ Switch ──▶ Internes Netz
                              (blockt aktiv)
```

- Sitzt direkt im Datenpfad
- Kann Verbindungen blockieren
- Höhere Anforderungen an Stabilität und Performance
- Risiko: Fehlkonfiguration kann den gesamten Netzwerkverkehr blockieren

---

## Rule-Ökosystem

Suricata nutzt **Signaturen (Rules)**, um Angriffe zu erkennen. Die wichtigsten Quellen:

| Rule-Set | Beschreibung | Kosten |
|----------|--------------|--------|
| **Emerging Threats Open (ET Open)** | Größtes kostenloses Rule-Set, täglich aktualisiert | Kostenlos |
| **Emerging Threats Pro (ET Pro)** | Erweiterte Regeln, schnellere Updates | Lizenz |
| **Snort Community Rules** | Offizielle Snort-Regeln (Suricata-kompatibel) | Kostenlos |
| **Talos VRT Rules** | Cisco Talos Rules (Snort, teilweise Suricata) | Registrierung |
| **Eigene Regeln** | Benutzerdefinierte Signaturen | Kostenlos |

### Rule-Syntax (Beispiel)

```
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 \
  (msg:"ET SCAN Potential SSH Scan";
   flags:S,12;
   threshold: type both, track by_src, count 5, seconds 120;
   classtype:attempted-recon;
   sid:2000001; rev:1;)
```

| Feld | Bedeutung |
|------|-----------|
| `alert` | Aktion (alert, drop, pass, reject) |
| `tcp` | Protokoll |
| `$EXTERNAL_NET any` | Quelle: Netzwerk, Port |
| `->` | Richtung |
| `$HOME_NET 22` | Ziel: Netzwerk, Port |
| `msg` | Alarmmeldung |
| `sid` | Signature ID |
| `rev` | Revision |

---

## Architektur-Integration

Suricata arbeitet am besten in Kombination mit anderen Tools:

```
┌─────────────────────────────────────────────┐
│              SOC-Umgebung                    │
├─────────────┬─────────────┬─────────────────┤
│  Suricata   │   Wazuh     │   EveBox        │
│  (Netzwerk) │   (Host)    │   (Visualisierung)│
├─────────────┼─────────────┼─────────────────┤
│  eve.json   │   Alerts    │   Dashboard     │
│  fast.log   │   FIM       │   Echtzeit      │
└─────────────┴─────────────┴─────────────────┘
              │
              ▼
        ┌─────────────┐
        │  Analyst    │
        │  (Mensch)   │
        └─────────────┘
```

### Typische Integrationen

| Tool | Funktion |
|------|----------|
| **Wazuh** | SIEM, zentrale Alert-Verwaltung, Host-Monitoring |
| **EveBox** | Web-Dashboard für Suricata-Events |
| **Zeek** | Ergänzende Netzwerkanalyse (Bro-Framework) |
| **Moloch/Arkime** | Vollständige PCAP-Speicherung und Suche |
| **Elasticsearch / OpenSearch** | Langzeit-Speicherung und Suche |
| **Grafana** | Visualisierung und Dashboards |
| **MISP** | Threat Intelligence-Plattform |

---

## Vorteile von Suricata

| Vorteil | Beschreibung |
|---------|--------------|
| **Open Source** | Keine Lizenzkosten, volle Transparenz |
| **Multi-Threaded** | Nutzt alle CPU-Kerne, hohe Performance |
| **Snort-kompatibel** | Bestehende Snort-Regeln können übernommen werden |
| **EVE JSON** | Einheitliches, maschinenlesbares Log-Format |
| **Protokoll-Erkennung** | Automatische Erkennung von 20+ Protokollen |
| **File Extraction** | Dateien aus dem Netzwerk rekonstruieren |
| **JA3-Fingerprinting** | Erkennung von TLS-Clients/Servern |
| **Lua Scripting** | Dynamische, komplexe Detektionslogik |
| **Aktive Community** | Regelmäßige Updates, große Community |

---

## Vergleich mit anderen IDS/IPS

| Kriterium | Suricata | Snort | Zeek | OSSEC |
|-----------|----------|-------|------|-------|
| **Typ** | NIDS/NIPS | NIDS/NIPS | NSM/Analyzer | HIDS |
| **Open Source** | ✅ Ja | ✅ Ja | ✅ Ja | ✅ Ja |
| **Multi-Threaded** | ✅ Ja | ❌ Nein (bis v3) | ✅ Ja | N/A |
| **EVE JSON** | ✅ Ja | ❌ Nein | ✅ Ja (JSON-Logs) | ❌ Nein |
| **File Extraction** | ✅ Ja | ⚠️ Begrenzt | ✅ Ja | ❌ Nein |
| **JA3 Support** | ✅ Ja | ⚠️ Plugin | ✅ Ja | ❌ Nein |
| **IPS Mode** | ✅ Ja | ✅ Ja | ❌ Nein | ❌ Nein |
| **Protocol Detection** | ✅ 20+ | ⚠️ Weniger | ✅ Extensiv | N/A |
| **Regel-Syntax** | Snort + Erweiterungen | Snort | eigene (Scripts) | eigene |
| **Ideal für** | Enterprise IDS/IPS | Klassisches IDS | Deep Network Analysis | Host-Monitoring |

---

## Typische Use Cases

### 🏠 Homelab / Lernen

- Aufbau eines IDS-Labs
- Verständnis für Netzwerkangriffe
- Üben von PCAP-Analyse
- Vorbereitung auf Security-Zertifizierungen (OSCP, CEH, etc.)

### 🏢 Unternehmensnetzwerk

- Überwachung des Internet-Gateways
- Erkennung von Malware-Kommunikation
- Erkennung von Port Scans und Exploit-Versuchen
- Compliance-Nachweis (Netzwerküberwachung)

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

## Zusammenfassung

Suricata ist eine **leistungsstarke, moderne und vielseitige Netzwerk-Überwachungs-Engine**, die alles bietet, was für die Netzwerksicherheit benötigt wird:

- ✅ **Erkennung** — NIDS mit tausenden Signaturen, Protokoll-Erkennung
- ✅ **Prävention** — NIPS-Modus zum aktiven Blockieren
- ✅ **Monitoring** — Vollständige Netzwerkprotokollierung (NSM)
- ✅ **Analyse** — EVE JSON, JA3-Fingerprinting, File Extraction
- ✅ **Integration** — Wazuh, EveBox, Zeek, MISP, Elasticsearch
- ✅ **Flexibilität** — IDS, IPS, NSM, Homelab bis Enterprise

Ob du ein Netzwerk überwachen, Angriffe erkennen, Dateien extrahieren oder einfach nur verstehen willst, was in deinem Netzwerk passiert — Suricata ist eine der besten Open-Source-Lösungen, die es gibt.

---

*Allgemeine Einführung zu Suricata*
