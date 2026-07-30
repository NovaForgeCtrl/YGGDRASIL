
<!-- ═══════════════════════════════════════════════════════════════
     YARA × WAZUH – BLUE TEAM HOMELAB
     Teil 2: Von der Theorie zum Alert (und zurück)
     ═══════════════════════════════════════════════════════════════ -->
# 🛡️ YARA × WAZUH INTEGRATION {: #soc-lab-teil2 }
<div align="center">

# 🛡️ YARA × WAZUH INTEGRATION
## Blue Team Homelab – Teil 2

```
    ██╗   ██╗ █████╗ ██████╗  █████╗     ██╗    ██╗ █████╗ ███████╗██╗   ██╗██╗  ██╗
    ╚██╗ ██╔╝██╔══██╗██╔══██╗██╔══██╗    ██║    ██║██╔══██╗╚══███╔╝██║   ██║██║  ██║
     ╚████╔╝ ███████║██████╔╝███████║    ██║ █╗ ██║███████║  ███╔╝ ██║   ██║███████║
      ╚██╔╝  ██╔══██║██╔══██╗██╔══██║    ██║███╗██║██╔══██║ ███╔╝  ██║   ██║██╔══██║
       ██║   ██║  ██║██║  ██║██║  ██║    ╚███╔███╔╝██║  ██║███████╗╚██████╔╝██║  ██║
       ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝  ╚═╝     ╚══╝╚══╝ ╚═╝  ╚═╝╚══════╝ ╚═════╝ ╚═╝  ╚═╝
```

**Datum:** 2026-07-30  
**Autor:** Marcel  
**Status:** ✅ YARA → Wazuh Alert Pipeline aktiv

</div>

---

## 📡 Architektur

```
                         ┌─────────────────┐
                         │   Netzwerk      │
                         └────────┬────────┘
                                  │
                         ┌────────▼────────┐
                         │  Suricata IDS   │  ← EveBox (Port 5636)
                         │  xxx.xxx.xxx.xxx|
                         └────────┬────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
    ┌─────────▼────────┐ ┌──────▼──────┐ ┌─────────▼────────┐
    │   Wazuh Manager    │ │  Wazuh      │ │   YARA Server    │
    │   + Indexer        │ │  Dashboard  │ │   Ubuntu 26.04   │
    │   + Dashboard      │ │             │ │   xxx.xxx.xxx.xxx│
    │   + EveBox         │ │             │ │                  │
    └─────────┬──────────┘ └─────────────┘ └────────┬─────────┘
              │                                     │
              │         Wazuh Agent (TCP 1514)      │
              │◄────────────────────────────────────┤
              │                                     │
              │    /opt/yara/logs/yara.log          │
              │    → Rule 100200 (Level 12)         │
              │                                     │
              └─────────────────────────────────────┘
```

---

## 🎯 Projektübersicht

Ziel dieses Labs: **Dateien mit YARA-Signaturen prüfen** und die Erkennungen als **Level-12-Alert** im Wazuh Dashboard sichtbar machen.

| Komponente | System | IP | Status |
|---|---|---|---|
| Wazuh Manager | Ubuntu 26.04 | xxx.xxx.xxx.xxx | ✅ |
| Wazuh Dashboard | — | — | ✅ |
| Suricata IDS | — | — | ✅ |
| EveBox | — | — | ✅ (no-auth) |
| YARA Server | Ubuntu 26.04 | xxx.xxx.xxx.xxx | ✅ |
| Wazuh Agent | — | — | ✅ (ID 002) |
| YARA → Wazuh Pipeline | — | — | ✅ |

---

## 🧬 Teil 1 – YARA Server Setup

### 1.1 Installation

```bash
sudo apt update
sudo apt install yara -y
yara --version
```

### 1.2 Verzeichnisstruktur

```bash
sudo mkdir -p /opt/yara/{rules,logs,quarantine,scripts,signature-base}
```

```
/opt/yara
├── rules/
│   └── test.yar
├── logs/
│   └── yara.log
├── quarantine/
├── scripts/
└── signature-base/
```

### 1.3 Testregel erstellen

```bash
sudo tee /opt/yara/rules/test.yar << 'EOF'
rule TestFile
{
    strings:
        $malware = "malware"
    condition:
        $malware
}
EOF
```

### 1.4 Testdatei & Scan

```bash
echo "malware" > /tmp/test.txt
sudo yara /opt/yara/rules/test.yar /tmp/test.txt
```

**Erwartete Ausgabe:**
```
TestFile /tmp/test.txt
```

### 1.5 Logging (wichtig!)

> ⚠️ **Nie** mit `sudo ... >> file` arbeiten – die Shell-Redirection läuft ohne sudo!

**Falsch:**
```bash
sudo yara rule.yar file >> yara.log   # Permission denied!
```

**Richtig:**
```bash
sudo yara /opt/yara/rules/test.yar /tmp/test.txt | sudo tee -a /opt/yara/logs/yara.log
```

---

## 🔌 Teil 2 – Wazuh Agent Konfiguration (yara Server)

### 2.1 Agent registrieren

Der Agent kommuniziert mit dem Manager über **TCP 1514**:

```xml
<client>
  <server>
    <address>xxx.xxx.xxx.xxx</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

### 2.2 YARA Log an Wazuh senden

In `/var/ossec/etc/ossec.conf` den folgenden Block **innerhalb** von `<ossec_config>` einfügen:

```xml
  <!-- YARA Malware Detection -->
  <localfile>
    <location>/opt/yara/logs/yara.log</location>
    <log_format>syslog</log_format>
    <label key="log_type">yara</label>
  </localfile>
```

> 💡 **Tipp:** Alle `<localfile>`-Einträge müssen **innerhalb** des `<ossec_config>`-Tags liegen. Ein verschobenes `</ossec_config>` zerstört die XML-Struktur!

### 2.3 XML validieren

```bash
sudo apt install libxml2-utils -y
sudo xmllint --noout /var/ossec/etc/ossec.conf && echo "XML SAUBER!" || echo "XML KAPUTT!"
```

### 2.4 Agent restart

```bash
sudo systemctl restart wazuh-agent
sudo /var/ossec/bin/wazuh-control status
```

---

## 🚨 Teil 3 – Wazuh Manager Regeln (suricata Server)

### 3.1 Custom Rule anlegen

```bash
sudo tee /var/ossec/etc/rules/local_rules.xml << 'EOF'
<group name="yara,">
  <rule id="100200" level="12">
    <match>TestFile</match>
    <description>YARA Detection: TestFile signature matched</description>
    <group>yara,malware,</group>
  </rule>
</group>
EOF
```

### 3.2 Manager restart

```bash
sudo systemctl restart wazuh-manager
```

### 3.3 Testalarm auslösen

**Auf dem yara Server:**
```bash
sudo yara /opt/yara/rules/test.yar /tmp/test.txt | sudo tee -a /opt/yara/logs/yara.log
```

**Auf dem suricata Manager prüfen:**
```bash
sudo grep "100200" /var/ossec/logs/alerts/alerts.log
```

**Erwartet:**
```
Rule: 100200 (level 12) -> 'YARA Detection: TestFile signature matched'
```

### 3.4 Im Dashboard filtern

Filter im Wazuh Dashboard → Security Events:
```
rule.id:100200
```

Oder über die Feld-Suche:
```
data.YARA.rule_name
```

---

## 📊 Teil 4 – EveBox Integration

### 4.1 Ohne Authentifizierung starten

**Manuell (zum Testen):**
```bash
sudo evebox server --config /etc/evebox/evebox.yaml --no-auth --host 0.0.0.0
```

> ⚠️ Port 5636 war belegt? Der systemd-Service läuft bereits im Hintergrund!

**Dauerhaft über systemd:**
```bash
sudo mkdir -p /etc/systemd/system/evebox.service.d/
sudo tee /etc/systemd/system/evebox.service.d/override.conf << 'EOF'
[Service]
ExecStart=
ExecStart=/usr/bin/evebox server --config /etc/evebox/evebox.yaml --no-auth --host 0.0.0.0
EOF
sudo systemctl daemon-reload
sudo systemctl restart evebox
```

---

## 🔥 Troubleshooting – Die Highlights

### ❌ XML Parser Error: "Extra content at the end of the document"

**Ursache:** `</ossec_config>` war doppelt vorhanden. Einige `<localfile>`-Einträge hingen außerhalb des Root-Tags.

**Fix:** Komplette Datei sauber neu schreiben mit `sudo tee` (nicht `sudo cat >`!):

```bash
sudo tee /var/ossec/etc/ossec.conf > /dev/null << 'EOF'
... komplette saubere XML ...
EOF
```

> 🔑 **Merke:** `sudo cat > file` funktioniert **nicht**, weil die `>`-Redirection als normaler User läuft, **bevor** sudo greift. Immer `sudo tee` verwenden!

### ❌ Rule 100200 feuert nicht

**Ursache:** Die Regel hat nicht gematcht, weil der Log zwar ankam, aber nicht als Alert gewertet wurde.

**Fix:** `local_rules.xml` mit `<match>` auf den Raw-Log-Text (hier: `TestFile`) erstellen. Nicht auf geparste Felder matchen, solange kein Custom Decoder existiert.

### ❌ EveBox: "Address in use (os error 98)"

**Ursache:** Der systemd-Service läuft bereits auf Port 5636.

**Fix:** `sudo systemctl stop evebox` vor dem manuellen Start, oder den Service-Override verwenden.

### ❌ Permission denied bei Log-Dateien

**Ursache:** Shell-Umleitung `>>` läuft ohne sudo-Rechte.

**Fix:** `| sudo tee -a /pfad/zur/datei` verwenden.

---

## 🖼️ Dashboard-Erklärungen

### Screenshot 1: YARA Events (falsch interpretiert)
- Zeigt **Rule 510** (rootcheck) – *nicht* YARA!
- "Trojaned version of file detected" ist ein Standard-rootcheck-Alert
- Die `rule.groups: yara` im Filter zeigt, dass die Gruppe existiert, aber die Events selbst stammen von rootcheck

![yara](../../assets/images/yara.png)

### Screenshot 2: Malware Detection (korrekt!)
- **1 Total**, **1 Level 12 or above**
- **Rule 100200** sichtbar im "Top 5 alerts"
- **Rule Groups:** `malware` + `yara` als Donut-Chart
- Das ist der **echte YARA-Alert**!

![yara](../../assets/images/malewaredetection.png)

---

## 🚀 Teil 2+ – Was noch kommen könnte

> Diese Sektion ist der Blick in die Glaskugel. Alles hier ist geplant, aber noch nicht umgesetzt.

### 🔗 Suricata → YARA File Extraction

```
Suricata (file.extract)
        │
        ▼
Datei landet in /var/log/suricata/files/
        │
        ▼
YARA scannt extrahierte Dateien
        │
        ▼
Wazuh Alarm (neue Rule, z.B. 100210)
```

**Benötigt:**
- Suricata `file-store` aktivieren
- Cronjob oder Inotify-Trigger für YARA-Scans
- Neue Wazuh-Regel für File-Extraction-Treffer

### 🛡️ Automatische Quarantäne

```
YARA Treffer
     │
     ▼
Wazuh Active Response
     │
     ▼
Script: mv Datei → /opt/yara/quarantine/
     │
     ▼
Alert: "Datei in Quarantäne verschoben"
```

**Benötigt:**
- Custom Active-Response-Script
- Wazuh `command` + `active-response` Konfiguration
- Passende Rechte auf Quarantäne-Verzeichnis

### 🔄 YARA Rule Auto-Updates

- Täglicher `git pull` aus [signature-base](https://github.com/Neo23x0/signature-base)
- Oder eigene private Rule-Repo
- Cronjob: `0 3 * * * /opt/yara/scripts/update-rules.sh`

### 🧩 Custom Decoder

Statt `<log_format>syslog</log_format>` könnte ein **Custom Decoder** die YARA-Ausgabe in schöne Felder parsen:

```
TestFile /tmp/test.txt
   │          │
   │          └── data.YARA.scanned_file
   └── data.YARA.rule_name
```

**Benötigt:**
- `/var/ossec/etc/decoders/local_decoder.xml`
- Regex-Pattern für YARA-Output

### 📈 File Integrity Monitoring für YARA Rules

```xml
<directories check_all="yes">/opt/yara/rules</directories>
```

Alarm wenn sich Regeln unerwartet ändern (Tampering-Erkennung).

### 🎯 Erweiterte Wazuh-Regeln

| Rule ID | Level | Trigger | Beschreibung |
|---------|-------|---------|--------------|
| 100200 | 12 | `TestFile` | Test-Signatur |
| 100201 | 12 | Any match | Generischer YARA-Match |
| 100210 | 15 | File extraction + YARA | Kritischer Malware-Fund |
| 100299 | 5 | Rule file changed | YARA-Regel modifiziert |

---

## 📋 Cheatsheet

```bash
# ═══ YARA ═══
sudo yara /opt/yara/rules/test.yar /tmp/test.txt | sudo tee -a /opt/yara/logs/yara.log
sudo cat /opt/yara/logs/yara.log

# ═══ Wazuh Agent (yara) ═══
sudo systemctl restart wazuh-agent
sudo /var/ossec/bin/wazuh-control status
sudo xmllint --noout /var/ossec/etc/ossec.conf

# ═══ Wazuh Manager (suricata) ═══
sudo systemctl restart wazuh-manager
sudo grep "100200" /var/ossec/logs/alerts/alerts.log
sudo tail -f /var/ossec/logs/alerts/alerts.json | grep -i yara

# ═══ EveBox ═══
sudo systemctl restart evebox
sudo evebox server --config /etc/evebox/evebox.yaml --no-auth --host 0.0.0.0

# ═══ Backup ═══
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
sudo cp /var/ossec/etc/rules/local_rules.xml /var/ossec/etc/rules/local_rules.xml.bak
```

---

## 🏁 Fazit

Die Pipeline ist **komplett durch**:

```
YARA Scan → yara.log → Wazuh Agent → Manager → Rule 100200 → Level 12 Alert → Dashboard 🚨
```

Nächster Meilenstein: **Suricata File-Extraction → YARA → Quarantäne**

> *"Ein Blue Team ohne Automation ist nur ein Red Team mit schlechterem Karma."*

---

<div align="center">

**Made with 💙 and too much caffeine**  
*Marcel @ Blue Team Homelab – 2026*

```
 ██████╗ ██╗     ██╗   ██╗███████╗    ████████╗███████╗ █████╗ ███╗   ███╗
 ██╔══██╗██║     ██║   ██║██╔════╝    ╚══██╔══╝██╔════╝██╔══██╗████╗ ████║
 ██████╔╝██║     ██║   ██║█████╗         ██║   █████╗  ███████║██╔████╔██║
 ██╔══██╗██║     ██║   ██║██╔══╝         ██║   ██╔══╝  ██╔══██║██║╚██╔╝██║
 ██████╔╝███████╗╚██████╔╝███████╗       ██║   ███████╗██║  ██║██║ ╚═╝ ██║
 ╚═════╝ ╚══════╝ ╚═════╝ ╚══════╝       ╚═╝   ╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝  
```

</div>
