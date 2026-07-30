# 🛡️ SOC-LAB — BLUE TEAM HQ {: #soc-lab-main}

> *"In the noise of a billion packets, we find the one that doesn't belong."*
> 
> `SIEM: Wazuh 4.x | IDS: Suricata 8.x | Status: OPERATIONAL`



```
Kali Linux
      |
      v
Ubuntu Server (Wazuh + Suricata)
      |
      v
Wazuh Agents

```
![Sentinel](../../assets/images/sentinel.png)
Das Bild wurde mit Gemini erzeugt :3
---


### Vorbereitung Ubuntu Server

System aktualisieren:

```bash
sudo apt update
sudo apt upgrade -y
```

Benötigte Werkzeuge installieren:

```bash
sudo apt install curl wget gnupg unzip -y
```

---

### Hostname konfigurieren

Hostname prüfen:

```bash
hostnamectl
```

Hostname setzen:

```bash
sudo hostnamectl set-hostname wazuh-server
```

Kontrolle:

```bash
hostname
```

---

### Statische IP-Adresse konfigurieren

Netplan Datei bearbeiten:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Beispiel:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - xxx.xxx.xxx.xxx/24
      routes:
        - to: default
          via: xxx.xxx.xxx.xxx
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Konfiguration anwenden:

```bash
sudo netplan apply
```

Kontrolle:

```bash
ip a
```

---

### Wazuh Installation

Installationsskript herunterladen:

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
```

Datei ausführbar machen:

```bash
chmod +x wazuh-install.sh
```

Wazuh All-in-One Installation starten:

```bash
sudo ./wazuh-install.sh -a
```

Installierte Komponenten:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

---

### Wazuh Status prüfen

Version prüfen:

```bash
sudo /var/ossec/bin/wazuh-control info
```

Beispiel:

```bash
WAZUH_VERSION="v4.x.x"
WAZUH_TYPE="server"
```

Services prüfen:

```bash
systemctl status wazuh-manager
systemctl status wazuh-indexer
systemctl status wazuh-dashboard
```

---

### Wazuh Dashboard


![wazuh](../../assets/images/wazuh.png)

| Attribut | Wert |
|----------|------|
| URL | `https://WAZUH-IP` |
| Standard-Port | 443 |

---

### Suricata Installation

Paketlisten aktualisieren:

```bash
sudo apt update
```

Suricata installieren:

```bash
sudo apt install suricata -y
```

Version prüfen:

```bash
suricata --build-info
```

---

### Suricata konfigurieren

Konfigurationsdatei:

```bash
sudo nano /etc/suricata/suricata.yaml
```

Wichtige Bereiche:

- `HOME_NET`
- `AF_PACKET`
- `Network Interface`
- `Rules`

Interface prüfen:

```bash
ip a
```

Beispiel:

```bash
ens18
```

---

### Suricata Rules installieren

Update der Regeln:

```bash
sudo suricata-update
```

Regeln prüfen:

```bash
ls /var/lib/suricata/rules/
```

---

### Suricata testen

Konfiguration prüfen:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Erwartet:

```bash
Configuration provided was successfully loaded
```

---

### Suricata Dienst aktivieren

Start:

```bash
sudo systemctl start suricata
```

Autostart aktivieren:

```bash
sudo systemctl enable suricata
```

Status:

```bash
systemctl status suricata
```

---

### Suricata Logs

Speicherort:

```bash
/var/log/suricata/
```

Wichtige Dateien:

| Datei | Beschreibung |
|-------|--------------|
| `eve.json` | JSON-Formatierte Ereignisse |
| `fast.log` | Kurze Alarmmeldungen |
| `stats.log` | Statistiken |
| `suricata.log` | Allgemeine Logs |

Beispiel:

```bash
sudo ls -lh /var/log/suricata/
```

---

### Suricata in Wazuh einbinden

Wazuh Konfiguration öffnen:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Eintrag hinzufügen:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Wazuh Konfiguration testen:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

Wazuh neu starten:

```bash
sudo systemctl restart wazuh-manager
```

---

### Wazuh Agent Installation

Der Agent sammelt Informationen von Clients und sendet diese an den Manager.

| Komponente | Wert |
|------------|------|
| Agent | `Agent-Name` |
| Agent IP | `xxx.xxx.xxx.xxx` |
| Manager | `xxx.xxx.xxx.xxx` |

---

### Eigene Wazuh Regeln

Regelpfad:

```bash
/var/ossec/etc/rules/local_rules.xml
```

Backup erstellen:

```bash
sudo cp /var/ossec/etc/rules/local_rules.xml \
  /var/ossec/etc/rules/local_rules.xml.backup
```

Bearbeiten:

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

Nach Änderungen:

Test:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

Neustart:

```bash
sudo systemctl restart wazuh-manager
```

---

### Kontrollbefehle

Wazuh Status:

```bash
sudo systemctl status wazuh-manager
```

Suricata Status:

```bash
sudo systemctl status suricata
```

Wazuh Logs:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

Suricata Logs:

```bash
sudo tail -f /var/log/suricata/fast.log
```

---

### Ergebnis

```
              Kali Linux
                  |
                  v
          Netzwerkverkehr
                  |
                  v
            Suricata IDS
                  |
                  v
           Wazuh Manager
                  |
                  v
          Wazuh Dashboard
                  |
                  v
          Security Analyst
```

```
    Kali Linux
          +
Netzwerkverkehr
          +
Suricata IDS
          +
Wazuh Manager
          +
Wazuh Dashboard
          +
Security Analyst
          =
Professionelles SOC-Lab im Homelab
```

| Status | ✅ Erfolgreich eingerichtet |
|--------|------------------------------|
| Einsatzbereich | SOC-Lab / Security Monitoring / Blue Team |

---

## 2. SOC-Lab Dokumentation

> **Projekt:** Blue-Team SOC-Lab  
> **Komponenten:** Wazuh + Suricata + Kali Angriffssimulation  
> **Stand:** 2026

### Projektübersicht

Dieses Labor simuliert eine kleine Security Operations Center (SOC)-Umgebung.

Ziel des Labs:

- Netzwerkangriffe simulieren
- Angriffe erkennen
- Logs sammeln
- Sicherheitsereignisse analysieren
- Detection Engineering üben
- MITRE ATT&CK Techniken nachvollziehen

---

### Netzwerkaufbau

| Komponente | Rolle | IP-Adresse |
|------------|-------|------------|
| Kali Linux | Angreifer-Simulation | – |
| Zielsystem | Angriffsziel | `xxx.xxx.xxx.xxx` |
| Ubuntu Server (Wazuh) | Wazuh + Suricata | `xxx.xxx.xxx.xxx` |
| Agent-System | Wazuh Agent | `xxx.xxx.xxx.xxx` |

Datenfluss: `Kali Linux → Zielsystem → Suricata IDS → Wazuh Manager → Wazuh Dashboard`

---

### Netzwerkplanung

#### Wazuh / Suricata Server

| Attribut | Wert |
|----------|------|
| Hostname | `wazuh-server` |
| IP-Adresse | `xxx.xxx.xxx.xxx` |

#### Zielsystem

| Attribut | Wert |
|----------|------|
| IP-Adresse | `xxx.xxx.xxx.xxx` |

#### Wazuh Agent

| Attribut | Wert |
|----------|------|
| Hostname | `agent-host` |
| IP-Adresse | `xxx.xxx.xxx.xxx` |

---

### Statische IP Konfiguration

Der Wazuh-Server benötigt eine feste IP-Adresse, damit Agents und Sicherheitskomponenten ihn zuverlässig erreichen.

Netplan Datei:

```bash
/etc/netplan/50-cloud-init.yaml
```

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - xxx.xxx.xxx.xxx/24
      routes:
        - to: default
          via: xxx.xxx.xxx.xxx
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Aktivieren:

```bash
sudo netplan apply
```

---

### Wazuh Installation

Download des Installationsskripts:

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
```

Ausführbar machen:

```bash
chmod +x wazuh-install.sh
```

Installation:

```bash
sudo ./wazuh-install.sh -a
```

Installierte Version prüfen:

```bash
sudo /var/ossec/bin/wazuh-control info
```

Ergebnis:

```bash
WAZUH_VERSION="v4.x.x"
WAZUH_TYPE="server"
```

---

### Wazuh Agent Einrichtung

Agents melden sich beim Wazuh Manager an.

| Agent | IP | Manager |
|-------|-----|---------|
| `agent-host` | `xxx.xxx.xxx.xxx` | `xxx.xxx.xxx.xxx` |

Erfolgreiche Registrierung im Log:

```bash
wazuh-authd:
Received request for a new agent (agent-host)
Agent key generated
```

---

### Suricata IDS

Suricata ist ein Network Intrusion Detection System (NIDS).

Aufgaben:

- Netzwerkverkehr analysieren
- Angriffe erkennen
- Signaturen anwenden
- Alerts erzeugen

Log-Verzeichnis:

```bash
/var/log/suricata/
```

Wichtige Dateien:

| Datei | Zweck |
|-------|-------|
| `eve.json` | Strukturierte Events im JSON-Format |
| `fast.log` | Kurzform der Alerts |
| `stats.log` | Performance-Statistiken |
| `suricata.log` | Allgemeine Logs |

---

### Suricata Integration in Wazuh

Suricata schreibt Events nach:

```bash
/var/log/suricata/eve.json
```

Wazuh liest diese Datei ein.

Konfiguration in:

```bash
/var/ossec/etc/ossec.conf
```

Eintrag:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Wazuh Neustart:

```bash
sudo systemctl restart wazuh-manager
```

---

### Suricata Regeln

Wazuh besitzt bereits Suricata-Regeln.

Prüfung:

```bash
sudo ls /var/ossec/ruleset/rules/ | grep -i suricata
```

Ergebnis:

```bash
0475-suricata_rules.xml
```

Diese Regeln ermöglichen die Erkennung von Suricata Events.

---

### Suricata und JSON Decoder Problem

Während der Integration kann folgende Meldung auftreten:

```bash
wazuh-analysisd: ERROR: Too many fields for JSON decoder
```

Ursache:

Die Datei `eve.json` enthält sehr viele verschachtelte Daten:

- HTTP
- DNS
- TLS
- Flow
- Fileinfo
- Alerts

Lösung / Fokus:

Für ein SOC sind hauptsächlich echte Alerts interessant:

```bash
event_type = alert
```

---

### Kali als Angreifer-Simulation

Kali Linux wird als kontrolliertes Red-Team-System verwendet.

Ziel: `xxx.xxx.xxx.xxx`

---

### Netzwerkaufklärung

#### Host Discovery

```bash
sudo nmap -sn xxx.xxx.xxx.xxx/24
```

Erkennt aktive Systeme im Netzwerk.

MITRE ATT&CK: `T1018` Remote System Discovery

#### Portscan

```bash
sudo nmap -sS xxx.xxx.xxx.xxx
```

Erkennt offene Ports.

#### Service Detection

```bash
sudo nmap -sV xxx.xxx.xxx.xxx
```

Erkennt:

- Dienste
- Versionen
- Banner

#### Erweiterter Scan

```bash
sudo nmap -A xxx.xxx.xxx.xxx
```

Erkennt:

- Betriebssystem
- Services
- Standard-Skripte

---

### SSH Simulation

SSH-Verbindung:

```bash
ssh user@xxx.xxx.xxx.xxx
```

Fehlversuche:

```bash
ssh testuser@xxx.xxx.xxx.xxx
```

Wazuh kann erkennen:

- Authentication Failure
- Brute Force Muster

MITRE ATT&CK: `T1110` Brute Force

---

### Netzwerkverkehr mit tcpdump

tcpdump zeigt den Rohverkehr.

| Befehl | Beschreibung |
|--------|--------------|
| `sudo tcpdump -i eth0 host xxx.xxx.xxx.xxx` | Grundlegender Verkehr |
| `sudo tcpdump -i eth0 -nn -vv host xxx.xxx.xxx.xxx` | Mehr Details |
| `sudo tcpdump -i eth0 -nn host xxx.xxx.xxx.xxx and port 22` | Nur SSH beobachten |
| `sudo tcpdump -i eth0 host xxx.xxx.xxx.xxx -w test.pcap` | Paketmitschnitt speichern |

Analyse der PCAP-Datei: Wireshark

---

### Monitoring

#### Suricata Live Logs

```bash
sudo tail -f /var/log/suricata/fast.log
```

#### Wazuh Alerts

```bash
sudo tail -f /var/ossec/logs/alerts/alerts.json
```

---

### Wazuh Dashboard

Analyse-Pfad:

```bash
Threat Hunting → Security Events
```

Filter:

```bash
rule.groups : suricata
```

oder:

```bash
agent.name : wazuh-server
```

---

### MITRE ATT&CK Zuordnung

| Simulation | Technik |
|------------|---------|
| Nmap Scan | `T1046` Network Service Discovery |
| Host Discovery | `T1018` Remote System Discovery |
| SSH Fehlversuche | `T1110` Brute Force |
| Service Enumeration | Discovery |
| Dateiänderungen | File Activity |

---

### SOC Datenfluss

| Schritt | Komponente |
|---------|------------|
| 1 | Kali Linux (Angriff) |
| 2 | Netzwerkverkehr |
| 3 | Suricata IDS (Erkennung) |
| 4 | Wazuh Manager (Aggregation) |
| 5 | Wazuh Dashboard (Visualisierung) |
| 6 | Security Analyst (Analyse) |

---

### Aktueller Status

- ✅ Wazuh Server installiert
- ✅ Wazuh Dashboard aktiv
- ✅ Wazuh Indexer aktiv
- ✅ Suricata IDS installiert
- ✅ Suricata Regeln vorhanden
- ✅ Wazuh Agent verbunden
- ✅ Kali Angriffssimulation durchgeführt
- ✅ Netzwerkverkehr mit tcpdump analysiert
- ✅ SOC-Datenfluss aufgebaut

> Hinweis zur Architektur: Diese Dokumentation beschreibt den Wazuh + Suricata + Kali-Stack. Die EveBox-Alert-Auswertung ist bewusst in einer separaten Dokumentation erfasst.

---

## 3. Wazuh SOC Rulebase

### Projektübersicht

#### Ziel

Diese Dokumentation beschreibt den Aufbau einer eigenen Wazuh-Regelbasis für ein Blue-Team / SOC-Labor.

Die Regeln erweitern die Standard-Erkennung von Wazuh um zusätzliche Erkennungen für:

- Authentifizierungsangriffe
- SSH- und PAM-Aktivitäten
- Suricata IDS Ereignisse
- File Integrity Monitoring
- Malware-Verhalten
- Docker Security
- Windows Sysmon Events
- Compliance-Überwachung

Die Regelbasis wurde für folgende Umgebung entwickelt:

| Komponente | Version |
|------------|---------|
| Betriebssystem | Ubuntu Server 26.04 LTS |
| Wazuh Manager | 4.x |
| Wazuh Indexer | 4.x |
| Wazuh Dashboard | 4.x |
| IDS Integration | Suricata |
| Visualisierung | EveBox / Wazuh Dashboard |

---

### Architektur

Die Überwachung erfolgt nach folgendem Prinzip:

```
                +----------------+
                |   Endpoints    |
                | Linux/Windows  |
                +-------+--------+
                        |
                        v
                +---------------+
                | Wazuh Agent   |
                +-------+-------+
                        |
                        v
                +---------------+
                | Wazuh Manager |
                +-------+-------+
                        |
        +---------------+---------------+
        |                               |
        v                               v

  Wazuh Rules                    Suricata IDS
  Detection Engine               Network Events

        |
        v

+----------------+
| Wazuh Dashboard|
| Alerts/SOC     |
+----------------+
```

---

### Verzeichnisstruktur

Die Regeln wurden modular aufgebaut:

```bash
/var/ossec/etc/rules/

├── local_rules.xml
├── ssh_pam_rules.xml
├── suricata_rules.xml
├── fim_rules.xml
├── malware_rules.xml
├── docker_rules.xml
├── windows_sysmon_rules.xml
└── compliance_rules.xml
```

Durch die Trennung bleiben die Regeln:

- wartbar
- einfacher zu testen
- leichter zu erweitern
- sicherer bei Änderungen

---

### Regel-ID Struktur

![regeln](../../assets/images/regeln.png)

Eigene Regeln verwenden den Bereich:


```bash
100000 - 100999
```

Aufteilung:

| Bereich | Funktion |
|---------|----------|
| 100000-100099 | Basis-Systemregeln |
| 100100-100199 | SSH/PAM/Sudo |
| 100200-100299 | Suricata IDS |
| 100300-100399 | File Integrity Monitoring |
| 100400-100499 | Malware Detection |
| 100500-100599 | Docker Security |
| 100600-100699 | Windows/Sysmon |
| 100700-100799 | Compliance |

---

### Basis Regeln

Datei: `local_rules.xml`

Aufgaben:

- Allgemeine Authentifizierungsüberwachung
- SSH Login Events
- Root Login Erkennung
- sudo Nutzung
- Account Änderungen
- Systemfehler

#### SSH Login Fehler

Erkennt:

```bash
Failed password
```

Ziel: Erkennung von Passwortangriffen, Brute Force, unerlaubten Zugriffen

MITRE Mapping: `T1110` - Brute Force

---

### SSH / PAM / Sudo Überwachung

Datei: `ssh_pam_rules.xml`

#### SSH Brute Force

Erkennung:

- mehrere fehlgeschlagene Logins
- gleiche Quelle
- kurze Zeitspanne

Beispiel:

```bash
5 Fehlversuche innerhalb 60 Sekunden
```

Alarmierung: Level 10

#### Root Login

Erkennt:

```bash
Accepted login for root
```

Risiko: Direkte Administratorzugriffe, kompromittierte Konten

MITRE: `T1078` Valid Accounts

#### Sudo Missbrauch

Erkennt:

- privilegierte Befehle
- Shell-Ausführung über sudo
- verdächtige Rechteerweiterung

MITRE: `T1548.003` Sudo Abuse

---

### Suricata IDS Integration

![suricata](../../assets/images/suricata.png)

Datei: `suricata_rules.xml`

Ziel: Verbindung zwischen Suricata und Wazuh

Überwachte Ereignisse:

- Port Scans
- Exploit-Versuche
- Malware Traffic
- Command & Control Kommunikation
- Brute Force Angriffe

#### Port Scan

Erkennung:

```bash
Nmap
Port Scan
ET SCAN
```

MITRE: `T1046` Network Service Scanning

---

### File Integrity Monitoring (FIM)

Datei: `fim_rules.xml`

Überwacht kritische Dateien:

Linux:

```bash
/etc/passwd
/etc/shadow
/etc/sudoers
```

SSH:

```bash
authorized_keys
```

Webserver:

```bash
/var/www
/usr/share/nginx
```

Ziel: Erkennung von Manipulationen, Backdoors, unautorisierten Änderungen

---

### Malware Detection

Datei: `malware_rules.xml`

#### Reverse Shell

Beispiele:

```bash
nc -e
bash -i
python socket
```

MITRE: `T1059` Command Shell

#### Remote Script Execution

Erkennt:

```bash
curl | bash
wget | bash
```

Risiko: Malware Downloads, Initial Access, Persistence

MITRE: `T1105` Ingress Tool Transfer

#### Crypto Miner

Erkennt:

```bash
xmrig
cpuminer
stratum
```

MITRE: `T1496` Resource Hijacking

---

### Docker Security

Datei: `docker_rules.xml`

Überwacht:

- Container Starts
- privilegierte Container
- Container Shell Zugriff
- Image Downloads

Kritisches Beispiel:

```bash
docker --privileged
```

Risiko: Container Escape, Host Kompromittierung

MITRE: `T1610` Deploy Container

---

### Windows / Sysmon Monitoring

Datei: `windows_sysmon_rules.xml`

Vorbereitet für Windows Agents.

#### Fehlgeschlagene Logins

Event: `4625`

MITRE: `T1110` Brute Force

#### Benutzeranlage

Event: `4720`

Risiko: Persistenz, unerlaubte Konten

MITRE: `T1136` Create Account

#### Security Log gelöscht

Event: `1102`

Risiko: Spurenvernichtung

MITRE: `T1070.001` Clear Windows Logs

#### PowerShell Detection

Erkennt:

```bash
EncodedCommand
-enc
```

Risiko: Malware, Obfuscation

MITRE: `T1059.001` PowerShell

---

### Compliance Monitoring

Datei: `compliance_rules.xml`

#### ISO 27001

Überwachung:

- Zugriffskontrolle
- Änderungen privilegierter Dateien
- Audit Ereignisse

#### PCI-DSS

Überwachung:

- Authentifizierung
- privilegierte Aktionen
- Systemänderungen

#### BSI Grundschutz

Überwachung:

- Logging
- Schutzmaßnahmen
- Sicherheitsereignisse

---

### Testverfahren

Nach Änderungen:

Syntaxprüfung:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

Erwartete Ausgabe:

```bash
Configuration OK
```

Neustart:

```bash
sudo systemctl restart wazuh-manager
```

Status:

```bash
sudo systemctl status wazuh-manager
```

Logs prüfen:

```bash
sudo tail -50 /var/ossec/logs/ossec.log
```

---

### Wartung

Empfohlene Vorgehensweise:

Vor Änderungen:

```bash
sudo cp -r /var/ossec/etc/rules /root/rules_backup
```

Nach Änderungen:

1. Syntax prüfen
2. Manager neu starten
3. Testalarm erzeugen
4. Dashboard prüfen

---

### Erweiterungsmöglichkeiten

Geplante Erweiterungen:

- Sigma Rule Integration
- MITRE ATT&CK Dashboard
- Active Response
- AbuseIPDB Integration
- VirusTotal API
- GeoIP Threat Intelligence
- YARA Integration
- Atomic Red Team Tests
- Detection Engineering Framework

---

### Zusammenfassung

Die entwickelte Wazuh Rulebase bildet eine modulare SOC-Überwachungsplattform.

Sie kombiniert:

- Host Monitoring
- Network Detection
- Threat Detection
- Compliance Monitoring
- Windows/Linux Security

und bildet damit eine solide Grundlage für ein Blue-Team Labor.

---

## 4. Wazuh SOC-Lab Security Monitoring & Incident Response

> Blue-Team Homelab mit Orientierung an NIS2 und BSI IT-Grundschutz  
> **Server:** `wazuh-server` (`xxx.xxx.xxx.xxx`)  
> **OS:** Ubuntu Server 26.04  
> **Wazuh Version:** 4.x

### Projektübersicht

#### Ziel

Aufbau einer praxisnahen Blue-Team Security Operations Center (SOC) Laborumgebung zur zentralisierten Überwachung, Erkennung und Reaktion auf Sicherheitsereignisse.

Die Implementierung orientiert sich an ausgewählten Anforderungen aus:

- NIS2 Artikel 21 (Risikomanagementmaßnahmen)
- BSI IT-Grundschutz (u.a. Angriffserkennung, Protokollierung, Identitäts- und Berechtigungsmanagement)

Die Umgebung dient als technische Demonstration von:

- zentralem Log-Management
- Angriffserkennung
- Sicherheitsanalyse
- automatisierter Incident Response
- Sicherheitsbenachrichtigung

#### Komponenten

| Komponente | Funktion | Status |
|------------|----------|--------|
| Wazuh Manager | SIEM, Log-Analyse, Regelwerk, Alerting | Aktiv |
| Suricata IDS | Netzwerkbasierte Angriffserkennung | Aktiv |
| EveBox | Visualisierung von IDS-Ereignissen | Aktiv |
| Postfix/Dovecot | Interne SMTP/IMAP Testumgebung | Aktiv |
| Custom Rules | Erweiterte Detection Rules für Linux Security Events | Aktiv |
| Active Response | Automatisierte Reaktion auf definierte Ereignisse | Aktiv |

---

### Systemarchitektur

```
┌──────────────────┐
│  Netzwerkverkehr │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Suricata IDS   │
│ Detection Engine │
└────────┬─────────┘
         │
         │ eve.json
         ▼
┌──────────────────┐
│ Wazuh Manager    │
│ SIEM + Rules     │
│ Alerting         │
└────────┬─────────┘
         │
  ┌──────┴────────┐
  ▼               ▼
Email Alert   Active Response
Postfix       Firewall DROP
```

---

### Wazuh Custom Rules (NIS2/BSI)

Datei: `/var/ossec/etc/rules/local_rules.xml`

Rule IDs: `100000 – 100199`

#### Technische Zuordnung zu NIS2-relevanten Sicherheitsmaßnahmen

Die folgenden Regeln unterstützen technische Kontrollmechanismen, die in NIS2 Artikel 21 beschrieben werden.

Die Zuordnung stellt keine vollständige NIS2-Konformitätsbewertung dar, sondern zeigt die technische Umsetzung einzelner Sicherheitsmaßnahmen.

| NIS2 Artikel | Anforderung | Rule IDs |
|--------------|-------------|----------|
| Art. 21(a) | Risikoanalyse & Sicherheitsmaßnahmen | 100034, 100050, 100060-100062 |
| Art. 21(e) | Erkennung & Behandlung von Vorfällen | 100002, 100090-100092 |
| Art. 21(g) | Sicherheit von Netzwerk- & Informationssystemen | 100050-100052 |
| Art. 23 | Vorfallmeldung & Nachweisführung | 100080-100081 |

#### Orientierung an BSI IT-Grundschutz

Die implementierten Funktionen orientieren sich an ausgewählten BSI IT-Grundschutz-Bausteinen.

Die Zuordnung dient zur Dokumentation der technischen Umsetzung und ersetzt keine vollständige Schutzbedarfsfeststellung oder Auditierung.

| BSI Baustein | Anforderung | Rule IDs |
|--------------|-------------|----------|
| ORP.4 | Identitäts- & Berechtigungsmanagement | 100030-100034 |
| IND.2 | Angriffserkennung | 100002, 100090-100092 |
| OPS.1.1.3 | Protokollierung & Monitoring | 100080-100081 |

#### Regel-Übersicht

| ID | Level | Beschreibung | MITRE | NIS2/BSI |
|----|-------|--------------|-------|----------|
| 100000 | 5 | SSH Auth Failure | T1110 | ✓ |
| 100001 | 8 | SSH Success | T1078 | ✓ |
| 100002 | 10 | SSH Brute Force | T1110.001 | ✓ |
| 100003 | 6 | SSH Key Login | T1078.004 | ✓ |
| 100004 | 7 | MFA/2FA Failure | T1111 | ✓ |
| 100010 | 10 | Root Login | T1078 | ✓ |
| 100011 | 7 | Privilege Escalation (su) | T1548 | ✓ |
| 100020 | 6 | Sudo Execution | T1548.003 | ✓ |
| 100021 | 8 | Sudo Fail | T1548.003 | ✓ |
| 100022 | 9 | Sudoers Modified | T1548.003 | ✓ |
| 100030 | 7 | User Created | T1136.001 | ✓ |
| 100031 | 7 | Password Changed | T1098 | ✓ |
| 100032 | 7 | User Deleted | T1531 | ✓ |
| 100033 | 6 | Group Modified | T1098 | ✓ |
| 100034 | 10 | /etc/passwd or /etc/shadow Modified | T1098 | ✓ |
| 100040 | 9 | Kernel Panic/OOM | – | ✓ |
| 100041 | 8 | Disk Full | – | ✓ |
| 100042 | 6 | System Reboot | – | ✓ |
| 100043 | 7 | High CPU Load | – | ✓ |
| 100050 | 8 | Firewall Modified | T1562.004 | ✓ |
| 100051 | 7 | New Network Connection | T1021 | ✓ |
| 100052 | 8 | Port Scan (Lab-only) | T1046 | ✓ |
| 100060 | 8 | SSH Config Modified | T1098 | ✓ |
| 100061 | 9 | Critical System File Modified | T1098 | ✓ |
| 100062 | 6 | Package Install/Remove | T1535 | ✓ |
| 100070 | 6 | Service Start/Stop | – | ✓ |
| 100071 | 7 | Cron Job Modified | T1053.003 | ✓ |
| 100072 | 8 | Kernel Module Loaded | T1547.006 | ✓ |
| 100080 | 10 | Log Tampering | T1070 | ✓ |
| 100081 | 9 | Logging Service Stopped | T1070 | ✓ |
| 100090 | 10 | Malware Detected | T1204.002 | ✓ |
| 100091 | 10 | Rootkit Detected | T1014 | ✓ |
| 100092 | 8 | Integrity Check Failed | T1565 | ✓ |
| 100095 | 6 | Backup Operation | – | ✓ |
| 100096 | 8 | Backup Failed | – | ✓ |
| 100199 | 10 | Active Response Test | – | ✓ |

---

### Active Response (Automatisches IP-Blocking)

Script: `/var/ossec/active-response/bin/firewall-drop`

#### Funktion

Blockt IPs automatisch bei SSH-Brute-Force.

#### Ablauf

```
[SSH Brute-Force] → [Rule 100002: Level 10] → [firewall-drop] → [iptables DROP]
                                    ↓
                           [Auto-Unblock nach 3600s]
```

#### Features

- IP-Blocking via `iptables -A INPUT -s <IP> -j DROP`
- Auto-Unblock nach 1 Stunde (3600s)
- Logging in `/var/ossec/logs/active-responses.log`
- Whitelist für `xxx.xxx.xxx.xxx` und `xxx.xxx.xxx.xxx`

#### Konfiguration in ossec.conf

```xml
<active-response>
    <disabled>no</disabled>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>100002</rules_id>
    <timeout>3600</timeout>
</active-response>
```

#### Sicherheitsbetrachtung

Die automatische Blockierung wurde ausschließlich für die Homelab-Testumgebung aktiviert.

In produktiven Umgebungen müssen vor automatisierten Maßnahmen zusätzliche Prüfungen erfolgen:

- Whitelist für Administrationssysteme
- Eskalationsprozesse
- Freigabe kritischer Aktionen
- Monitoring möglicher False Positives

---

### Email-Alerts

#### Mailserver

| Attribut | Wert |
|----------|------|
| Server | Postfix/Dovecot auf `xxx.xxx.xxx.xxx` |
| Domain | `intern.local` |
| Empfänger | `admin@intern.local` |

#### Wazuh Email-Konfiguration

```xml
<global>
    <email_notification>yes</email_notification>
    <smtp_server>xxx.xxx.xxx.xxx</smtp_server>
    <email_from>wazuh@wazuh-server.intern.local</email_from>
    <email_to>admin@intern.local</email_to>
    <email_maxperhour>12</email_maxperhour>
</global>

<alerts>
    <log_alert_level>3</log_alert_level>
    <email_alert_level>8</email_alert_level>
</alerts>
```

#### Getestete Email

- Betreff: "Wazuh Notification"
- Inhalt: Rule 100061 fired (Level 9) – Critical system configuration file modified
- Empfangen in: Thunderbird (Windows)

---

### Suricata-Integration

#### Suricata Status

| Attribut | Wert |
|----------|------|
| Version | 8.x.x RELEASE |
| Status | active (running) |
| Logs | `/var/log/suricata/eve.json` |
| Fast.log | Wird von Wazuh eingelesen |

#### Ereignisfluss

Suricata analysiert Netzwerkverkehr und schreibt erkannte Ereignisse in das EVE-JSON Format.

Wazuh übernimmt diese Logs und führt anschließend durch:

- Dekodierung
- Regelprüfung
- Severity-Bewertung
- Alert-Erstellung

#### Wazuh empfängt Suricata

- Alerts: 21.404 Suricata-Events
- Log-Quelle: `/var/log/suricata/fast.log`
- Integration: Über `<localfile>` in `ossec.conf`

---

### EveBox Dashboard

#### Einrichtung

```bash
# EveBox Config
sudo tee /etc/evebox/evebox.yaml > /dev/null <<'EOF'
http:
  host: "0.0.0.0"
  port: 5636
  tls:
    enabled: false

database:
  type: sqlite
  sqlite:
    path: /var/lib/evebox/evebox.sqlite

input:
  enabled: true
  paths:
    - "/var/log/suricata/eve.json"
EOF

# Starten
sudo evebox server --config /etc/evebox/evebox.yaml --no-auth
```

#### Zugriff

| Attribut | Wert |
|----------|------|
| URL | `http://xxx.xxx.xxx.xxx:5636` |
| Auth | Keine (für Homelab) |

> Hinweis: EveBox wurde im Homelab ohne Authentifizierung betrieben. Für produktive Umgebungen sollte eine Zugriffskontrolle (z.B. Reverse Proxy mit Authentifizierung) vorgeschaltet werden.

---

### Testprotokolle

#### Test 1: Active Response (IP-Blocking)

| Schritt | Befehl | Ergebnis |
|---------|--------|----------|
| 1 | `sudo /var/ossec/active-response/bin/firewall-drop add root xxx.xxx.xxx.xxx 12345 100002 10 /var/log/auth.log` | ✅ IP geblockt |
| 2 | `sudo iptables -L INPUT -n | grep DROP` | ✅ DROP-Regel sichtbar |
| 3 | `sudo /var/ossec/active-response/bin/firewall-drop delete root xxx.xxx.xxx.xxx 12345 100002 10 /var/log/auth.log` | ✅ IP freigegeben |

#### Test 2: Email-Alerts

| Schritt | Befehl | Ergebnis |
|---------|--------|----------|
| 1 | `echo "Test von Wazuh" | mail -s "Wazuh Test" -r wazuh@wazuh-server.intern.local admin@intern.local` | ✅ Email gesendet |
| 2 | Prüfung in Thunderbird | ✅ "Wazuh Test" angekommen |
| 3 | Wazuh Alert (Rule 100061) | ✅ "Wazuh Notification" angekommen |

#### Test 3: Brute-Force Simulation

| Schritt | Befehl | Ergebnis |
|---------|--------|----------|
| 1 | `for i in {1..5}; do ssh -o BatchMode=yes wronguser@127.0.0.1; done` | ✅ 5 Auth-Failures |
| 2 | Wazuh Dashboard: Rule 100002 | ✅ Alert generiert |
| 3 | Active Response Log | ✅ "BLOCKED IP" |

#### Test 4: Suricata + Wazuh Integration

| Prüfung | Ergebnis |
|---------|----------|
| Suricata Status | active (1h 51min) |
| EVE-JSON Größe | 8.2 MB |
| Wazuh Suricata Alerts | 21.404 Events |
| EveBox erreichbar | ✅ Port 5636 |

---

### Zuordnung zu NIS2-Anforderungen

| NIS2 Anforderung | Implementiert | Nachweis |
|------------------|---------------|----------|
| Art. 21(a) Risikoanalyse | ✅ | Rules 100034, 100050, 100060-100062 |
| Art. 21(e) Vorfallbehandlung | ✅ | Rules 100002, 100090-100092, Active Response |
| Art. 21(g) Netzwerksicherheit | ✅ | Rules 100050-100052, Suricata IDS |
| Art. 23 Meldung & Nachweis | ✅ | Email-Alerts, Log-Retention |
| BSI ORP.4 Identitätsmanagement | ✅ | Rules 100030-100034 |
| BSI IND.2 Angriffserkennung | ✅ | Rules 100002, 100090-100092 |
| BSI OPS.1.1.3 Logging | ✅ | Rules 100080-100081 |

---

### Fehlerbehebung

#### Problem: EveBox Login-Fenster trotz authentication: false

Ursache: EveBox ignoriert die Config und nutzt interne SQLite-DB

Lösung:

```bash
sudo rm -f /var/lib/evebox/config.sqlite
sudo evebox server --config /etc/evebox/evebox.yaml --no-auth
```

#### Problem: mail Command not found

Lösung:

```bash
sudo apt install mailutils -y
```

#### Problem: XML-Fehler in ossec.conf

Ursache: Elemente außerhalb von `<ossec_config>`

Lösung:

```bash
sudo sed -i '/^<\/ossec_config>$/q' /var/ossec/etc/ossec.conf
sudo /var/ossec/bin/wazuh-analysisd -t
```

#### Problem: Doppelte `<global>` Blöcke

Lösung: Alle `<global>` Blöcke in einen zusammenfassen

---

### Security Operations Lifecycle

1. **Detection**
   - Suricata Netzwerküberwachung
   - Wazuh Loganalyse
   - Custom Detection Rules

2. **Analysis**
   - Severity Bewertung
   - MITRE ATT&CK Mapping
   - Ereigniskorrelation

3. **Response**
   - Active Response
   - Firewall Blocking
   - Administrator Benachrichtigung

4. **Documentation**
   - Alert Logs
   - Testprotokolle
   - Sicherheitsnachweise

---

### Anhänge

#### A. Wichtige Dateien

| Datei | Pfad | Beschreibung |
|-------|------|--------------|
| Custom Rules | `/var/ossec/etc/rules/local_rules.xml` | NIS2/BSI Regeln |
| Wazuh Config | `/var/ossec/etc/ossec.conf` | Manager-Konfiguration |
| Active Response Script | `/var/ossec/active-response/bin/firewall-drop` | IP-Blocking |
| EveBox Config | `/etc/evebox/evebox.yaml` | Suricata Dashboard |
| Alert Logs | `/var/ossec/logs/alerts/alerts.log` | Wazuh Alerts |
| Active Response Logs | `/var/ossec/logs/active-responses.log` | IP-Blocking Logs |

#### B. Nützliche Befehle

```bash
# Wazuh validieren
sudo /var/ossec/bin/wazuh-analysisd -t

# Wazuh restart
sudo systemctl restart wazuh-manager

# Suricata Status
sudo systemctl status suricata

# EveBox starten
sudo evebox server --config /etc/evebox/evebox.yaml --no-auth

# Geblockte IPs
sudo iptables -L INPUT -n | grep DROP

# Letzte Alerts
sudo tail -20 /var/ossec/logs/alerts/alerts.log

# Email-Test
echo "Test" | mail -s "Test" admin@intern.local
```

---

### Fazit

Die implementierte Umgebung stellt ein praxisnahes Blue-Team SOC-Labor dar.

Sie demonstriert zentrale Fähigkeiten moderner Security Operations:

- ✅ zentrale Loganalyse mit Wazuh
- ✅ Netzwerküberwachung mit Suricata IDS
- ✅ automatisierte Reaktion über Active Response
- ✅ Sicherheitsbenachrichtigungen per E-Mail
- ✅ Dokumentation und Zuordnung zu Security Frameworks

Die Umgebung bildet technische Aspekte von Incident Detection, Analyse und Response ab und dient als Grundlage für zukünftige Erweiterungen, wie zusätzliche Agents, Vulnerability Management, Threat Intelligence Integration und erweitertes Monitoring.

#### Komponenten

- ✅ 25+ Custom Wazuh Rules
- ✅ Automatisches IP-Blocking (Active Response)
- ✅ Email-Alerts bei Sicherheitsvorfällen
- ✅ Suricata IDS Integration
- ✅ EveBox Dashboard
- ✅ Lokaler Mailserver

#### Status

Erfolgreich getestete Blue-Team SOC-Laborumgebung 🚀

---

## 5. SOC Attack Generator v3

### Projektübersicht

Der SOC Attack Generator ist eine eigene Testumgebung zur Simulation von Sicherheitsereignissen für ein Security Operations Center (SOC).

Ziel ist es, kontrollierte Angriffs- und Bedrohungsszenarien in einer isolierten Laborumgebung zu erzeugen und die Erkennung durch Wazuh SIEM zu testen.

Die Simulation erzeugt typische Sicherheitsereignisse aus verschiedenen MITRE ATT&CK Bereichen.

---

### Umgebung

#### Verwendete Komponenten

| Komponente | Funktion |
|------------|----------|
| Kali Linux | Angriffs- und Simulationsterminal |
| Wazuh Manager | SIEM / Detection Engine |
| Wazuh Dashboard | Visualisierung und Analyse |
| Ubuntu Server | Zielsystem / Agent |
| Suricata | Netzwerküberwachung |
| SOC Attack Generator | Angriffssimulation |

---

### Projektstruktur

Aktuelle Struktur:

```bash
soc-attack-generator-v3
├── attack-chain.sh
├── modules/
│   ├── recon.sh
│   ├── initial_access.sh
│   ├── execution.sh
│   ├── privilege.sh
│   ├── persistence.sh
│   ├── defense_evasion.sh
│   ├── discovery.sh
│   ├── credential_access.sh
│   ├── lateral_movement/
│   ├── collection.sh
│   ├── c2/
│   ├── exfiltration.sh
│   ├── impact.sh
│   ├── web_attack.sh
│   ├── docker_security.sh
│   ├── windows_sysmon.sh
│   ├── cloud.sh
│   ├── fim_test.sh
│   ├── reporting.sh
│   ├── threat_intel.sh
│   ├── ueba.sh
│   └── ransomware_sim.sh
```

---

### Installieren und Starten

#### Rechte setzen

Nach neuen Modulen:

```bash
chmod +x attack-chain.sh
chmod +x modules/*.sh
```

#### Attack Chain starten

Im Hauptverzeichnis:

```bash
./attack-chain.sh
```

Die einzelnen Module werden automatisch ausgeführt.

---

### Module und Simulationen

#### Reconnaissance

Datei: `modules/recon.sh`

Simuliert:

- Netzwerkaufklärung
- Host Discovery
- Port Scan Ereignisse
- Informationssammlung

MITRE: `TA0043` Reconnaissance

#### Initial Access

Datei: `modules/initial_access.sh`

Simuliert:

- SSH Angriffe
- Fehlgeschlagene Logins
- Zugriffversuche

MITRE: `TA0001` Initial Access

#### Execution

Datei: `modules/execution.sh`

Simuliert:

- Shell-Ausführung
- verdächtige Befehle
- Script-Ausführung

MITRE: `TA0002` Execution

#### Persistence

Datei: `modules/persistence.sh`

Simuliert:

- Cron Persistence
- SSH Key Manipulation
- dauerhafte Zugriffsmöglichkeiten

MITRE: `TA0003` Persistence

#### Privilege Escalation

Datei: `modules/privilege.sh`

Simuliert:

- sudo Missbrauch
- Root Zugriff
- Rechteerweiterung

MITRE: `TA0004` Privilege Escalation

#### Defense Evasion

Datei: `modules/defense_evasion.sh`

Simuliert:

- Log Manipulation
- Verschleierung
- Umgehen von Überwachung

MITRE: `TA0005` Defense Evasion

#### Credential Access

Datei: `modules/credential_access.sh`

Simuliert:

- Passwortzugriffe
- Credential Dumping Indikatoren
- Authentifizierungsangriffe

MITRE: `TA0006` Credential Access

#### Discovery

Datei: `modules/discovery.sh`

Simuliert:

- Benutzerabfragen
- Systeminformationen
- Prozessinformationen

Beispiele:

```bash
whoami
uname
ps
```

MITRE: `TA0007` Discovery

#### Lateral Movement

Ordner: `modules/lateral_movement/`

Simuliert:

- interne Bewegungen
- Zugriff auf weitere Systeme

MITRE: `TA0008` Lateral Movement

#### Collection

Datei: `modules/collection.sh`

Simuliert:

- Datensammlung
- Zugriff auf Dateien
- Informationssammlung

MITRE: `TA0009` Collection

#### Command and Control

Ordner: `modules/c2/`

Simuliert:

- verdächtige Kommunikation
- C2 Indikatoren

MITRE: `TA0011` Command and Control

#### Exfiltration

Datei: `modules/exfiltration.sh`

Simuliert:

- Datenabflussindikatoren
- verdächtige Transfers

MITRE: `TA0010` Exfiltration

#### Impact

Datei: `modules/impact.sh`

Simuliert:

- Systemausfälle
- Schadensereignisse
- Manipulation

MITRE: `TA0040` Impact

#### Docker Security

Datei: `modules/docker_security.sh`

Simuliert:

- Container Sicherheitsereignisse
- privilegierte Container
- Container Risiken

#### Windows Sysmon

Datei: `modules/windows_sysmon.sh`

Vorbereitung für:

- Windows Events
- Sysmon Logs
- PowerShell Erkennung

#### Cloud Security

Datei: `modules/cloud.sh`

Simuliert:

- Cloud Sicherheitsereignisse
- IAM Änderungen
- Cloud Fehlkonfigurationen

#### Web Attacks

Datei: `modules/web_attack.sh`

Simuliert:

- SQL Injection
- XSS
- Directory Traversal
- Web Scanner

#### Threat Intelligence

Datei: `modules/threat_intel.sh`

Simuliert:

- bekannte Schad-IP
- IOC Treffer
- Malware Indicators
- Blacklist Treffer

Beispiele:

```bash
Threat Intel: Known malicious IP detected
Threat Intel: IOC hash match detected
Threat Intel: Suspicious domain detected
```

#### UEBA

Datei: `modules/ueba.sh`

Simuliert User Behavior Analytics:

- ungewöhnliche Logins
- ungewöhnliche Zugriffe
- privilegierte Kontoanomalien

Beispiele:

```bash
UEBA: Unusual login detected
UEBA: Abnormal access pattern detected
UEBA: Privileged account anomaly
```

#### Ransomware Simulation

Datei: `modules/ransomware_sim.sh`

Simuliert typische Erkennungen:

- Mass File Modification
- Verschlüsselungsverhalten
- Shadow Copy Manipulation

Beispiele:

```bash
Ransomware: Mass file modification detected
Ransomware: Encryption behavior detected
```

---

### Wazuh prüfen

Nach Start des Generators werden Events an das System-Logging gesendet.

Auf dem Wazuh Server prüfen:

```bash
sudo tail -f /var/ossec/logs/alerts/alerts.json
```

oder:

```bash
sudo tail -f /var/ossec/logs/alerts/alerts.log
```

#### Erwartete Events

```bash
Threat Intel: Known malicious IP detected
UEBA: Unusual login detected
Ransomware: Mass file modification detected
```

#### Im Dashboard

Navigation:

```bash
Wazuh Dashboard → Security Events → Threat Hunting → MITRE ATT&CK → Rule Level
```

Mögliche Anzeigen:

- Credential Access
- Execution
- Persistence
- Defense Evasion
- Discovery
- Lateral Movement
- Command and Control
- Impact

---

### Geplante Erweiterungen

Folgende Module sind geplant:

| Modul | Datei | Beschreibung |
|-------|-------|--------------|
| Phishing Simulation | `modules/phishing_sim.sh` | Phishing Events, Suspicious Mail Indicators, User Interaction Events |
| Malware Delivery | `modules/malware_delivery.sh` | verdächtige Dateien, Download Events, Ausführungsmuster |
| API Security | `modules/api_attack.sh` | API Abuse, Fehlende Authentifizierung, ungewöhnliche Requests |
| Insider Threat | `modules/insider_threat.sh` | ungewöhnliche Benutzeraktivität, Datenzugriffe, Rechteänderungen |
| Automatische Reports | `reports/` | daily_report.json, mitre_matrix.json, wazuh_summary.txt |

---

### Status

#### Aktuell umgesetzt

- ✅ Recon
- ✅ Initial Access
- ✅ Execution
- ✅ Persistence
- ✅ Privilege Escalation
- ✅ Defense Evasion
- ✅ Credential Access
- ✅ Discovery
- ✅ Lateral Movement
- ✅ Collection
- ✅ Command & Control
- ✅ Exfiltration
- ✅ Impact
- ✅ Docker Security
- ✅ Windows Sysmon Vorbereitung
- ✅ Cloud Simulation
- ✅ Web Attack Simulation
- ✅ Threat Intelligence
- ✅ UEBA
- ✅ Ransomware Simulation

#### Projektstatus

🟢 SOC Attack Generator v3 aktiv

---

## 6. Wazuh Passwort zurücksetzen

> **Projekt:** SOC-Lab  
> **Dienst:** Wazuh Dashboard / Wazuh Indexer  
> **Version:** Wazuh 4.x  
> **System:** Ubuntu Server

### Zweck

Diese Anleitung beschreibt das Zurücksetzen des Passworts eines Wazuh Indexer Benutzers.

Beispiel:

| Feld     | Wert  |
|----------|-------|
| Benutzer | admin |

---

### Voraussetzungen

Benötigt:

- Root-Zugriff auf den Wazuh Server
- Laufender Wazuh Indexer
- Laufendes Wazuh Dashboard

Prüfen:

```bash
systemctl status wazuh-indexer
systemctl status wazuh-dashboard
```

---

### Zum Passwort-Tool wechseln

Verzeichnis:

```bash
cd /usr/share/wazuh-indexer/plugins/opensearch-security/tools/
```

Prüfen:

```bash
ls
```

Erwartet:

```
wazuh-passwords-tool.sh
```

---

### Hilfe anzeigen

```bash
./wazuh-passwords-tool.sh --help
```

---

### Passwort für Benutzer ändern

Beispiel:

| Feld     | Wert       |
|----------|------------|
| Benutzer | admin      |
| Passwort | `NeuesSicheresPasswort!` |

Befehl:

```bash
./wazuh-passwords-tool.sh   -u admin   -p 'NeuesSicheresPasswort!'
```

---

### Dienste neu starten

```bash
systemctl restart wazuh-indexer
systemctl restart wazuh-dashboard
```

---

### Status prüfen

```bash
systemctl status wazuh-indexer --no-pager
systemctl status wazuh-dashboard --no-pager
```

Erwartet:

```
Active: active (running)
```

---

### Anmeldung testen

URL:

```
https://WAZUH-IP
```

Login:

| Feld     | Wert              |
|----------|-------------------|
| Benutzer | admin             |
| Passwort | neues Passwort    |

---

### Fehlerbehebung

#### Fehler: Invalid username or password

Prüfen:

- Benutzername korrekt
- Passwort ohne Tippfehler
- Dienste neu gestartet
- Wazuh Indexer läuft

#### Logs prüfen

Wazuh Dashboard:

```bash
journalctl -u wazuh-dashboard -f
```

Wazuh Indexer:

```bash
journalctl -u wazuh-indexer -f
```

---

### Ergebnis

Das Passwort des Wazuh Dashboard Benutzers wurde erfolgreich geändert.

```
    Wazuh Dashboard
            +
    Wazuh Indexer
            +
wazuh-passwords-tool.sh
            =
    Passwort erfolgreich zurückgesetzt
```

| Status         | ✅ Erfolgreich durchgeführt       |
|----------------|-----------------------------------|
| Einsatzbereich | SOC-Lab / Security Monitoring     |

---

*Ende der Masterdokumentation*
