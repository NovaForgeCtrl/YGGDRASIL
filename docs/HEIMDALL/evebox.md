# 📊 EVEBOX — Der Visualisierer {: #evebox-main }

> *"A web-based event viewer and alert management tool for Suricata. Jeder Alert erzählt eine Geschichte — EveBox macht sie sichtbar."*

---

## 📋 Systeminformationen

| Attribut | Wert |
|----------|------|
| **Status** | 🟢 Aktiv |
| **Version** | 0.x |
| **Erstellt** | 2016 (Jason Ish) |
| **Zuletzt geändert** | 2026-07 |
| **Lizenz** | Open Source (MIT) |
| **Sprache** | Go + Vue.js |

---

## 🎯 Ziel / Zweck

EveBox ist eine Open-Source Web-Anwendung zur Visualisierung und Verwaltung von Suricata-Ereignissen. Es liest die `eve.json`-Logs von Suricata ein und stellt sie in einer übersichtlichen, interaktiven Web-Oberfläche dar — vergleichbar mit einem Mini-SIEM, das speziell auf Netzwerk-IDS-Events zugeschnitten ist.

---

## 🛠️ Komponenten

[EVE-JSON Import](evebox-import.md){: .world-link } · [Inbox Workflow](evebox-inbox.md){: .world-link } · [Suche & Filter](evebox-search.md){: .world-link } · [Reports](evebox-reports.md){: .world-link }

---

## 🏗️ Architektur

```
┌─────────────────┐
│   Suricata      │  ← Erzeugt eve.json
│   (IDS/IPS)     │
└────────┬────────┘
         │ eve.json
         ▼
┌─────────────────┐
│    EveBox       │  ← Liest, indiziert, visualisiert
│  (Web-Engine)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Browser       │  ← Analyst sieht Dashboard
│   (Analyst)     │
└─────────────────┘
```

---

## 🔍 Hauptfunktionen

### EVE-JSON Import

EveBox liest Suricatas `eve.json` direkt ein:

- **File-based** — Überwacht die Log-Datei auf neue Zeilen
- **Event-getrieben** — Zeigt neue Alerts nahezu in Echtzeit an
- **SQLite-Backend** — Eigene, eingebettete Datenbank

### Suche & Filter

Schnelle Suche über alle Events:

- Nach **Source/Destination IP**
- Nach **Port**
- Nach **Signature-ID** oder **Signature-Name**
- Nach **Event-Type** (alert, http, dns, tls, flow)
- Nach **Severity**
- Nach **Zeitraum**

### Inbox-Workflow

EveBox bringt ein einfaches Ticket-System mit:

| Status | Bedeutung |
|--------|-----------|
| **New** | Unbearbeitete Alerts |
| **Archived** | Geprüft und harmlos |
| **Escalated** | Weiterleitung an SOC/Senior-Analyst |

### Statistiken & Reports

- Top-Alerts nach Häufigkeit
- Alerts über Zeit (Trends)
- Verteilung nach Kategorien
- Quell- und Ziel-IP-Übersichten

### Web-Oberfläche

- Responsive Design (Desktop & Mobile)
- Keine Installation auf Client-Seite nötig
- Direkter Zugriff über Browser

---

## 📊 Datenbank-Optionen

| Backend | Beschreibung | Einsatz |
|---------|--------------|---------|
| **SQLite** | Eingebettet, Datei-basiert | Homelab, Kleinumgebungen |
| **PostgreSQL** | Externe Datenbank | Enterprise, viele Events |
| **Elasticsearch** | Direkte Verbindung zu ES | Bestehende ELK-Stack-Umgebungen |

---

## ✅ Vorteile

| Vorteil | Beschreibung |
|---------|--------------|
| **Open Source** | Kostenlos, quelloffen |
| **Leichtgewichtig** | Geringer Ressourcenverbrauch |
| **Einfache Installation** | Ein Binary, keine Abhängigkeiten |
| **Kein Elasticsearch nötig** | SQLite reicht für die meisten Fälle |
| **Echtzeit** | Nahezu live-Anzeige neuer Alerts |
| **Inbox-Workflow** | Einfaches Alert-Management |
| **Mobil-freundlich** | Auch auf dem Handy nutzbar |
| **Multi-Input** | Mehrere Suricata-Instanzen möglich |

---

## 📊 Vergleich mit anderen Tools

| Kriterium | EveBox | Kibana | Wazuh Dashboard | SGUIL |
|-----------|--------|--------|-----------------|-------|
| **Fokus** | Suricata-Alerts | Generisch (ELK) | SIEM (Host + Netzwerk) | Snort/Suricata |
| **Datenbank** | SQLite / PG / ES | Elasticsearch | OpenSearch | MySQL |
| **Installation** | Sehr einfach | Komplex | Mittel | Komplex |
| **Ressourcen** | Sehr gering | Hoch | Mittel | Mittel |
| **Echtzeit** | ✅ Ja | ✅ Ja | ✅ Ja | ⚠️ Verzögert |
| **Inbox-Workflow** | ✅ Ja | ❌ Nein | ⚠️ Begrenzt | ✅ Ja |
| **Ideal für** | Schnelle Übersicht | Big Data Analyse | Voll-SIEM | Classic IDS |

---

## 🎯 Typische Use Cases

### 🏠 Homelab / IDS-Lab

- Schönes Dashboard für Suricata-Alerts
- Lernen von Netzwerk-Angriffserkennung
- Schneller Überblick ohne SIEM-Aufwand

### 🏢 Kleine & Mittlere Unternehmen

- Netzwerküberwachung ohne teures SIEM
- Schnelle Reaktion auf Netzwerk-Angriffe
- Compliance-Nachweis (Netzwerk-Logs)

### 🚨 SOC / Incident Response

- Erste Anlaufstelle für Suricata-Alerts
- Triage vor Weiterleitung an Wazuh/Splunk
- Eskalations-Workflow für kritische Events

### 🔍 Threat Hunting

- Suche nach verdächtigen Mustern
- Trend-Analyse über Zeit
- Quell-IP-Recherche

---

## 📝 Notizen

!!! note "Wichtig"
    - SQLite reicht für die meisten Homelabs
    - Für Enterprise: PostgreSQL oder Elasticsearch empfohlen
    - Regelmäßige Backups der EveBox-Datenbank
    - Suricata eve.json-Format muss kompatibel sein

---

> *"Ein Bild sagt mehr als tausend Logs. EveBox verwandelt rohe Daten in Erkenntnisse."*
