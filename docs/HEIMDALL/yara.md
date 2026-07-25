# 🔬 YARA — Das Artefakt {: #yara-main }

> *"The pattern matching Swiss knife for malware researchers. Jede Signatur ist ein Schlüssel — jedes Muster eine Wahrheit."*

---

## 📋 Systeminformationen

| Attribut | Wert |
|----------|------|
| **Status** | 🟢 Aktiv |
| **Version** | 4.x |
| **Erstellt** | 2008 (Victor M. Alvarez / VirusTotal) |
| **Zuletzt geändert** | 2026-07 |
| **Lizenz** | Open Source (BSD-3-Clause) |

---

## 🎯 Ziel / Zweck

YARA ist eine Open-Source Pattern-Matching-Engine, die speziell für die Erkennung und Klassifizierung von Malware entwickelt wurde. Sie ermöglicht es, textbasierte Signaturen (YARA-Rules) zu schreiben, die Dateien, Prozesse oder Speicherinhalte nach charakteristischen Mustern durchsuchen.

---

## 🛠️ Komponenten

[YARA Engine](yara-engine.md){: .world-link } · [Rule Syntax](yara-rules.md){: .world-link } · [PE Modul](yara-pe.md){: .world-link } · [Integration](yara-integration.md){: .world-link }

---

## 🏗️ Architektur

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

---

## 🔍 Hauptfunktionen

### Pattern Matching

YARA durchsucht Binärdateien nach Textstrings, hexadezimalen Sequenzen, regulären Ausdrücken und Datei-Headern — alles in einer einzigen Rule kombinierbar.

### Malware-Klassifizierung

YARA-Rules erkennen nicht nur Bedrohungen, sondern klassifizieren sie auch:

- Malware-Familie identifizieren (z. B. "TrickBot", "Emotet", "Cobalt Strike")
- Attribution zu Threat Actors
- Versions- und Varianten-Erkennung

### Modular erweiterbar

Durch Module lassen sich spezialisierte Analysen durchführen:

- **PE / ELF** — Windows- und Linux-Dateien analysieren
- **Math** — Mathematische Berechnungen auf Dateiinhalten
- **Hash** — MD5, SHA1, SHA256 Berechnung
- **Dotnet** — .NET Assembly Analyse
- **Magic** — Dateityp-Erkennung

### Hochperformant

- Multi-threaded Scanning
- Schnelle Suche in großen Dateimengen
- Speicher-effiziente Verarbeitung

---

## 📊 Was kann YARA scannen?

| Ziel | Beschreibung |
|------|--------------|
| **Dateien** | Einzelne Dateien oder ganze Verzeichnisse |
| **Prozesse** | Laufende Prozesse und deren Speicher |
| **Speicher-Dumps** | RAM-Dumps, Crash-Dumps, Hibernation-Files |
| **Netzwerk-Streams** | Daten aus dem Netzwerkverkehr |
| **Archive** | ZIP, RAR, etc. |

---

## ✅ Vorteile

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

## 📊 Vergleich mit anderen Tools

| Kriterium | YARA | ClamAV | ssdeep | Hash-Based |
|-----------|------|--------|--------|------------|
| **Methode** | Pattern-Matching | Signaturen + Heuristik | Fuzzy Hashing | Exakte Hashes |
| **Open Source** | ✅ Ja | ✅ Ja | ✅ Ja | N/A |
| **Fuzzy Matching** | ⚠️ Teilweise | ❌ Nein | ✅ Ja | ❌ Nein |
| **Malware-Familien** | ✅ Ja | ✅ Ja | ⚠️ Begrenzt | ❌ Nein |
| **Speicher-Scan** | ✅ Ja | ❌ Nein | ❌ Nein | ❌ Nein |
| **Einfach zu schreiben** | ✅ Ja | ⚠️ Komplex | N/A | N/A |
| **Ideal für** | Malware-Familien, IOCs | Allgemeiner Virenschutz | Ähnlichkeitsvergleich | Exakte Treffer |

---

## 🎯 Typische Use Cases

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

## 📝 Notizen

!!! note "Wichtig"
    - Rules regelmäßig aktualisieren und testen
    - False Positives minimieren durch präzise Conditions
    - Community-Repositories im Blick behalten
    - Backup eigener Rule-Sammlungen empfohlen

---

> *"Jede Zeile Code hinterlässt eine Spur. YARA ist die Lupe, die sie sichtbar macht."*
