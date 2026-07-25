# 📊 EveBox — Allgemeine Einführung

> *"A web-based event viewer and alert management tool for Suricata."*

---

## Was ist EveBox?

**EveBox** ist eine Open-Source Web-Anwendung zur Visualisierung und Verwaltung von Suricata-Ereignissen. Es liest die `eve.json`-Logs von Suricata ein und stellt sie in einer übersichtlichen, interaktiven Web-Oberfläche dar — vergleichbar mit einem Mini-SIEM, das speziell auf Netzwerk-IDS-Events zugeschnitten ist.

EveBox wurde von **Jason Ish** entwickelt und ist in **Go** (Backend) und **Vue.js** (Frontend) geschrieben. Es ist leichtgewichtig, schnell zu installieren und benötigt keine komplexe Infrastruktur wie Elasticsearch oder eine Datenbank-Cluster.

---

## Für wen ist EveBox gedacht?

| Zielgruppe | Einsatzzweck |
|-----------|--------------|
| **SOC-Teams** | Schnelle Übersicht über Suricata-Alerts |
| **Blue Teams** | Netzwerk-Threat-Hunting, Alert-Analyse |
| **Netzwerkadministratoren** | Überwachung des Netzwerkverkehrs ohne komplexes SIEM |
| **Homelab-Enthusiasten** | Schönes Dashboard für das IDS-Lab |
| **Incident Responder** | Ereignis-Timeline, Alert-Details, Eskalation |
| **MSSP** | Mehrere Suricata-Instanzen zentral überwachen |

---

## Kernkonzept

EveBox vereinfacht die Arbeit mit Suricata-Logs erheblich:

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

### Was EveBox kann

| Funktion | Beschreibung |
|----------|--------------|
| **Echtzeit-Alerts** | Live-Anzeige eingehender Suricata-Alerts |
| **Ereignis-Timeline** | Chronologische Darstellung aller Events |
| **Filter & Suche** | Nach IP, Port, Signature, Severity filtern |
| **Alert-Details** | Vollständige JSON-Daten jedes Events |
| **Inbox-Workflow** | Alerts als "Neu", "Archiviert", "Eskaliert" markieren |
| **Kommentare** | Notizen zu einzelnen Alerts hinterlegen |
| **Reports** | Übersichten über Top-Alerts, Trends |
| **Multi-Input** | Mehrere Suricata-Instanzen gleichzeitig |

---

## Hauptfunktionen

### 📥 EVE-JSON Import

EveBox liest Suricatas `eve.json` direkt ein:

- **File-based** — Überwacht die Log-Datei auf neue Zeilen
- **Event-getrieben** — Zeigt neue Alerts nahezu in Echtzeit an
- **SQLite-Backend** — Eigene, eingebettete Datenbank (kein externer Server nötig)

### 🔍 Suche & Filter

Schnelle Suche über alle Events:

- Nach **Source/Destination IP**
- Nach **Port**
- Nach **Signature-ID** oder **Signature-Name**
- Nach **Event-Type** (alert, http, dns, tls, flow)
- Nach **Severity**
- Nach **Zeitraum**

### 📬 Inbox-Workflow

EveBox bringt ein einfaches Ticket-System mit:

| Status | Bedeutung |
|--------|-----------|
| **New** | Unbearbeitete Alerts |
| **Archived** | Geprüft und harmlos |
| **Escalated** | Weiterleitung an SOC/Senior-Analyst |

### 📈 Statistiken & Reports

- Top-Alerts nach Häufigkeit
- Alerts über Zeit (Trends)
- Verteilung nach Kategorien
- Quell- und Ziel-IP-Übersichten

### 🌐 Web-Oberfläche

- Responsive Design (Desktop & Mobile)
- Keine Installation auf Client-Seite nötig
- Direkter Zugriff über Browser

---

## Architektur

EveBox ist bewusst schlank gehalten:

```
┌─────────────────────────────────────────┐
│              EveBox Server               │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │   Go-API    │  │   SQLite-DB     │   │
│  │  (Backend)  │  │  (Embedded)     │   │
│  └──────┬──────┘  └─────────────────┘   │
│         │                                │
│  ┌──────┴──────┐  ┌─────────────────┐   │
│  │  Vue.js UI  │  │  EVE-JSON Input │   │
│  │  (Frontend) │  │  (File Watcher) │   │
│  └─────────────┘  └─────────────────┘   │
└─────────────────────────────────────────┘
```

### Datenbank-Optionen

| Backend | Beschreibung | Einsatz |
|---------|--------------|---------|
| **SQLite** | Eingebettet, Datei-basiert | Homelab, Kleinumgebungen |
| **PostgreSQL** | Externe Datenbank | Enterprise, viele Events |
| **Elasticsearch** | Direkte Verbindung zu ES | Bestehende ELK-Stack-Umgebungen |

---

## Integration in den Security-Stack

EveBox ergänzt Suricata und andere Tools ideal:

```
┌─────────────────────────────────────────────┐
│           Netzwerk-Monitoring                │
├─────────────┬─────────────┬─────────────────┤
│  Suricata   │   EveBox    │   Wazuh         │
│  (Erkennung)│  (Dashboard)│  (SIEM)         │
├─────────────┼─────────────┼─────────────────┤
│  eve.json   │   SQLite    │   Alerts        │
│  fast.log   │   Web-UI    │   FIM           │
└─────────────┴─────────────┴─────────────────┘
              │
              ▼
        ┌─────────────┐
        │   Analyst   │
        └─────────────┘
```

### Typische Workflows

| Workflow | Beschreibung |
|----------|--------------|
| **Suricata + EveBox** | Schnelle Alert-Übersicht ohne SIEM-Komplexität |
| **Suricata + EveBox + Wazuh** | EveBox für schnelle Checks, Wazuh für tiefe Analyse |
| **Suricata + EveBox + Elasticsearch** | Langzeit-Speicherung + Echtzeit-Dashboard |

---

## Vorteile von EveBox

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

## Vergleich mit anderen Tools

| Kriterium | EveBox | Kibana | Wazuh Dashboard | SGUIL |
|-----------|--------|--------|-----------------|-------|
| **Fokus** | Suricata-Alerts | Generisch (ELK) | SIEM (Host + Netzwerk) | Snort/Suricata |
| **Datenbank** | SQLite / PG / ES | Elasticsearch | OpenSearch | MySQL |
| **Installation** | Sehr einfach | Komplex | Mittel | Komplex |
| **Ressourcen** | Sehr gering | Hoch | Mittel | Mittel |
| **Echtzeit** | ✅ Ja | ✅ Ja | ✅ Ja | ⚠️ Verzögert |
| **Inbox-Workflow** | ✅ Ja | ❌ Nein | ⚠️ Begrenzt | ✅ Ja |
| **Alert-Management** | ✅ Ja | ❌ Nein | ✅ Ja | ✅ Ja |
| **Ideal für** | Schnelle Übersicht | Big Data Analyse | Voll-SIEM | Classic IDS |

---

## Typische Use Cases

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

## Zusammenfassung

EveBox ist das **perfekte Dashboard für Suricata** — schlank, schnell und auf den Punkt gebracht:

- ✅ **Visualisierung** — Übersichtliche Darstellung von Suricata-Alerts
- ✅ **Echtzeit** — Nahezu live-Anzeige neuer Events
- ✅ **Management** — Inbox-Workflow für Alert-Triage
- ✅ **Einfachheit** — Keine komplexe Infrastruktur nötig
- ✅ **Flexibilität** — SQLite, PostgreSQL oder Elasticsearch
- ✅ **Integration** — Ergänzt Wazuh, Kibana oder steht allein

Wenn du Suricata betreibst und eine schnelle, schöne Möglichkeit suchst, die Alerts zu sehen und zu verwalten — ohne ein vollwertiges SIEM aufzusetzen — ist EveBox genau das Richtige.

---

*Allgemeine Einführung zu EveBox*
