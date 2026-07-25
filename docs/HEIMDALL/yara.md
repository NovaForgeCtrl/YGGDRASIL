# 🔬 YARA — Allgemeine Einführung

> *"The pattern matching Swiss knife for malware researchers."*

---

## Was ist YARA?

**YARA** ist eine Open-Source Pattern-Matching-Engine, die speziell für die Erkennung und Klassifizierung von Malware entwickelt wurde. Der Name ist ein Akronym für **"Yet Another Recursive Acronym"** — oder auch liebevoll als **"Yet Another Ridiculous Acronym"** bezeichnet.

YARA ermöglicht es Sicherheitsforschern, Analysten und Incident-Respondern, sogenannte **YARA-Rules** zu schreiben — textbasierte Signaturen, die Dateien, Prozesse oder Speicherinhalte nach charakteristischen Mustern durchsuchen. Es wird weltweit in der Cybersecurity-Industrie eingesetzt und ist das De-facto-Standardwerkzeug für Malware-Erkennung auf Basis von Inhalten.

YARA wurde ursprünglich von **Victor M. Alvarez** bei VirusTotal entwickelt und wird seitdem kontinuierlich von der Community und Unternehmen weiterentwickelt.

---

## Für wen ist YARA gedacht?

| Zielgruppe | Einsatzzweck |
|-----------|--------------|
| **Malware-Analysten** | Erstellung von Signaturen für neue Malware-Familien |
| **Threat Intelligence Analysten** | IOC-Erstellung, Threat-Hunting, Attribution |
| **SOC-Teams** | Automatisierte Malware-Erkennung in SIEMs |
| **Incident Responder** | Schnelle Identifikation bekannter Bedrohungen |
| **Forensiker** | Dateisystem-Scans nach verdächtigen Mustern |
| **Homelab-Enthusiasten** | Lernen von Malware-Analyse, Reverse Engineering |
| **AV/EDR-Hersteller** | Ergänzende Erkennung neben heuristischen Methoden |

---

## Kernkonzept

YARA arbeitet nach einem einfachen Prinzip: **Du beschreibst, wonach du suchst — YARA findet es.**

```
┌─────────────────┐
│   YARA Rule     │  ← Du schreibst die Signatur
│  (Text-Pattern) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  YARA Engine    │  ← Scannt Dateien/Prozesse/Speicher
│  (Pattern Match)│
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
 MATCH      NO MATCH
```

### Was kann YARA scannen?

| Ziel | Beschreibung |
|------|--------------|
| **Dateien** | Einzelne Dateien oder ganze Verzeichnisse |
| **Prozesse** | Laufende Prozesse und deren Speicher |
| **Speicher-Dumps** | RAM-Dumps, Crash-Dumps, Hibernation-Files |
| **Netzwerk-Streams** | Daten aus dem Netzwerkverkehr |
| **Archive** | ZIP, RAR, etc. (mit entsprechenden Modulen) |

---

## Hauptfunktionen

### 🔍 Pattern Matching

YARA durchsucht Binärdateien nach:

- **Textstrings** — Menschenlesbare Wörter, URLs, Registry-Pfade
- **Hexadezimale Sequenzen** — Byte-Muster, Opcodes, Header-Signaturen
- **Reguläre Ausdrücke** — Komplexe Muster in Dateien
- **Datei-Header** — Magic Bytes, PE-Header, ELF-Header

### 🏷️ Malware-Klassifizierung

YARA-Rules können nicht nur erkennen, sondern auch klassifizieren:

- Malware-Familie identifizieren (z. B. "TrickBot", "Emotet", "Cobalt Strike")
- Attribution zu Threat Actors
- Versions- und Varianten-Erkennung

### 🔗 Modular erweiterbar

YARA unterstützt Module für spezialisierte Analyse:

| Modul | Funktion |
|-------|----------|
| **PE** | Portable Executable Analyse (Windows-Dateien) |
| **ELF** | Executable and Linkable Format (Linux-Dateien) |
| **Math** | Mathematische Berechnungen auf Dateiinhalten |
| **Hash** | MD5, SHA1, SHA256 Berechnung |
| **Dotnet** | .NET Assembly Analyse |
| **Cuckoo** | Integration mit Cuckoo Sandbox |
| **Magic** | Dateityp-Erkennung (libmagic) |

### ⚡ Hochperformant

- Multi-threaded Scanning
- Schnelle Suche in großen Dateimengen
- Speicher-effiziente Verarbeitung

---

## YARA Rule Syntax

Eine YARA-Rule besteht aus vier Teilen:

```yara
rule Beispiel_Malware
{
    meta:
        description = "Erkennt Beispiel-Malware Familie"
        author = "Analyst"
        date = "2026-07-25"
        hash = "d41d8cd98f00b204e9800998ecf8427e"

    strings:
        $a = "suspicious_domain.com" ascii wide
        $b = { 4D 5A 90 00 }              // PE-Header
        $c = /https?:\/\/evil\.[a-z]{2,6}/i  // Regex
        $d = "C:\Users\Admin\AppData\Roaming" wide

    condition:
        uint16(0) == 0x5A4D and          // PE-Datei
        ($a or $c) and
        filesize < 5MB
}
```

### Rule-Struktur

| Abschnitt | Beschreibung |
|-----------|--------------|
| `rule Name` | Eindeutiger Name der Regel |
| `meta` | Metadaten (Beschreibung, Autor, Hash, etc.) |
| `strings` | Die zu suchenden Muster |
| `condition` | Logische Bedingung für einen Match |

### String-Modifier

| Modifier | Bedeutung |
|----------|-----------|
| `ascii` | Suche nach ASCII-Encoding |
| `wide` | Suche nach UTF-16LE (Windows-Strings) |
| `nocase` | Groß-/Kleinschreibung ignorieren |
| `fullword` | Nur ganze Wörter matchen |
| `xor` | XOR-verschlüsselte Varianten finden |
| `base64` | Base64-kodierte Strings finden |

### Condition-Operatoren

| Operator | Beschreibung |
|----------|--------------|
| `and` | Alle Bedingungen müssen erfüllt sein |
| `or` | Mindestens eine Bedingung muss erfüllt sein |
| `not` | Bedingung darf nicht erfüllt sein |
| `uint16(0)` | Prüft 2 Bytes an Offset 0 (z. B. PE-Magic `MZ`) |
| `filesize` | Dateigröße prüfen |
| `pe.number_of_sections` | PE-Modul: Anzahl der Sektionen |
| `elf.type` | ELF-Modul: Dateityp |
| `math.entropy` | Entropie-Berechnung (verschlüsselte/packte Dateien) |

---

## YARA in der Praxis

### Einzelne Datei scannen

```bash
yara rule.yar datei.exe
```

### Verzeichnis rekursiv scannen

```bash
yara -r rule.yar /pfad/zum/verzeichnis/
```

### Prozess-Speicher scannen

```bash
yara -p 1234 rule.yar
```

### Mehrere Rules gleichzeitig

```bash
yara -r /pfad/zu/rules/ /pfad/zum/scan/
```

### Mit Tags filtern

```bash
yara -t trojan rules/ verzeichnis/
```

---

## Integration in Security-Tools

YARA ist das Bindglied zwischen manueller Analyse und automatisierter Erkennung:

```
┌─────────────────────────────────────────────┐
│           Security-Stack                     │
├─────────────┬─────────────┬─────────────────┤
│   YARA      │   Sandbox   │   SIEM          │
│   (Rules)   │   (Cuckoo)  │   (Wazuh)       │
├─────────────┼─────────────┼─────────────────┤
│   VirusTotal│   MISP      │   TheHive       │
│   (Scan)    │   (Intel)   │   (Case Mgmt)   │
└─────────────┴─────────────┴─────────────────┘
```

### Typische Integrationen

| Tool | Integration |
|------|-------------|
| **VirusTotal** | YARA-Suche über Milliarden von Samples |
| **MISP** | Austausch von YARA-Rules als IOCs |
| **Wazuh** | Automatischer Datei-Scan mit YARA-Rules |
| **Cuckoo Sandbox** | YARA-Scan nach Sandbox-Analyse |
| **TheHive / Cortex** | YARA-Analysator für Cases |
| **Sigma** | Konvertierung von Sigma zu YARA |
| **Velociraptor** | YARA-Scan auf Endpoints |
| **LOKI** | IOC-Scanner mit YARA-Unterstützung |

---

## YARA Rule Repositories

Die Community teilt YARA-Rules frei:

| Repository | Beschreibung |
|-----------|--------------|
| **YARA-Rules Project** | Große Sammlung von Community-Rules |
| **Elastic Security** | Offizielle Rules von Elastic |
| **ReversingLabs** | Hochwertige Malware-Familien-Rules |
| **Florian Roth (Neo23x0)** | Extensive Sammlung, sehr aktiv |
| **InQuest** | Enterprise-Grade Rules |
| **VirusTotal Hunting** | Community-Rules auf VT |

---

## Vorteile von YARA

| Vorteil | Beschreibung |
|---------|--------------|
| **Open Source** | Kostenlos, transparent, erweiterbar |
| **Einfache Syntax** | Menschenlesbare Rules, schnell zu erlernen |
| **Sehr schnell** | Effiziente Pattern-Matching-Engine |
| **Flexibel** | Dateien, Speicher, Prozesse, Netzwerk |
| **Modular** | PE, ELF, Math, Hash, etc. |
| **Community** | Riesige Sammlung geteilter Rules |
| **Industrie-Standard** | De-facto-Standard für Malware-Signaturen |
| **SIEM-fähig** | Einfache Integration in Wazuh, Splunk, etc. |

---

## Vergleich mit anderen Tools

| Kriterium | YARA | ClamAV | ssdeep | Hash-Based |
|-----------|------|--------|--------|------------|
| **Methode** | Pattern-Matching | Signaturen + Heuristik | Fuzzy Hashing | Exakte Hashes |
| **Open Source** | ✅ Ja | ✅ Ja | ✅ Ja | N/A |
| **Fuzzy Matching** | ⚠️ Teilweise | ❌ Nein | ✅ Ja | ❌ Nein |
| **Malware-Familien** | ✅ Ja | ✅ Ja | ⚠️ Begrenzt | ❌ Nein |
| **False Positives** | Niedrig | Mittel | Niedrig | Sehr niedrig |
| **Einfach zu schreiben** | ✅ Ja | ⚠️ Komplex | N/A | N/A |
| **Speicher-Scan** | ✅ Ja | ❌ Nein | ❌ Nein | ❌ Nein |
| **Ideal für** | Malware-Familien, IOCs | Allgemeiner Virenschutz | Ähnlichkeitsvergleich | Exakte Treffer |

---

## Typische Use Cases

### 🕵️ Malware-Analyse

- Erstellung von Signaturen für neue Malware-Familien
- Identifikation von Code-Überlappungen zwischen Samples
- Attribution zu bekannten Threat Actors

### 🚨 Incident Response

- Schneller Scan kompromittierter Systeme
- Erkennung bekannter Tools (Mimikatz, Cobalt Strike, etc.)
- Speicher-Dump-Analyse nach laufender Malware

### 🏹 Threat Hunting

- Proaktive Suche nach IoCs im Unternehmen
- Hunt nach unbekannten Varianten bekannter Malware
- Erkennung von Living-off-the-Land-Tools

### 🔒 Compliance & Forensik

- Dateisystem-Scans auf verdächtige Dateien
- Beweissicherung durch Hash- und Pattern-Matching
- Dokumentation von Malware-Befall

### 🧪 Homelab / Lernen

- Verständnis für Malware-Strukturen
- Üben von Reverse Engineering
- Aufbau eigener Rule-Sammlungen

---

## Zusammenfassung

YARA ist das **Schweizer Taschenmesser der Malware-Erkennung** — ein unverzichtbares Werkzeug für jeden, der im Bereich Cybersecurity arbeitet:

- ✅ **Erkennung** — Pattern-Matching auf Dateien, Speicher, Prozesse
- ✅ **Klassifizierung** — Zuordnung zu Malware-Familien und Threat Actors
- ✅ **Flexibilität** — Einfache Syntax, mächtige Conditions, modular erweiterbar
- ✅ **Integration** — VirusTotal, MISP, Wazuh, Cuckoo, TheHive, Velociraptor
- ✅ **Community** — Tausende geteilte Rules, aktiv gepflegte Repositories
- ✅ **Standard** — De-facto-Industriestandard für Malware-Signaturen

Ob du Malware analysierst, einen Incident bearbeitest, proaktiv huntst oder einfach nur lernen willst, wie Cyber-Bedrohungen erkannt werden — YARA ist eines der wichtigsten Tools in deinem Arsenal.

---

*Allgemeine Einführung zu YARA*
