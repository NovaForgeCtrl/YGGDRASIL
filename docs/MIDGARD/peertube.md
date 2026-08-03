
<html><h1 id="peertube-main">PeerTube Docker</h1></html>
# PeerTube Docker Installation – Problemdokumentation

> **Server:** peertube (Ubuntu Server, SSH) | **Installation:** Docker Compose | **Zugriff:** http://xxx.xxx.xxx.xxx:9000

---

## Systemübersicht

| Komponente | Image / Version | Status |
|-----------|-----------------|--------|
| Host | peertube (Ubuntu Server) | ✅ SSH Zugriff |
| PeerTube | chocobozzz/peertube:production-bookworm | ⚠️ instabil |
| PostgreSQL | 13 | ✅ läuft |
| Redis | 7 | ✅ läuft |
| Zugriff | http://xxx.xxx.xxx.xxx:9000 | ✅ erreichbar |

---

## Hauptproblem

> **❌ PeerTube startet nicht stabil** – Container befindet sich in einem inkonsistenten Zustand mit Restart-Loop.

---

## Fehleranalyse (Kernprobleme)

### 1. Unique Constraint Error (Initial Crash Loop)

| Attribut | Wert |
|----------|------|
| Fehler | `SequelizeUniqueConstraintError` |
| Ursache | `actor.url must be unique` |
| Problem | PeerTube versucht beim Start erneut einen Application Actor anzulegen: `https://192.168.2.195:9000/accounts/peertube` |
| Ergebnis | Actor existiert bereits → DB Konflikt → Restart Loop |

### 2. Wiederholter Installationsversuch

- Installer läuft mehrfach
- Versucht:
  - Application Account zu erstellen
  - Admin anzulegen
- → Crash → Restart → Loop

### 3. PostgreSQL Rollenproblem

```
role "postgres" does not exist
```

> **Ursache:** DB User ist `peertube`, nicht `postgres`.  
> **Lösung:** `psql -U peertube`

### 4. Inkonsistente Datenbank

| Tabelle | Status | Bemerkung |
|---------|--------|-----------|
| `actor` | ✅ enthält Daten | accounts/peertube vorhanden |
| `user` | ✅ enthält root | Admin existiert |
| `application` | ❌ leer | nicht initialisiert |
| `server` | ❌ leer | nicht initialisiert |

> **Ergebnis:** Halb initialisiertes Setup

### 5. Login Problem

- Kein funktionierender Admin Login
- Kein klarer Setup Wizard
- Kein sauberer Initial Setup Zustand

### 6. CLI Tools fehlen

```
Cannot find module /app/dist/server/tools/peertube.js
```

> **Ursache:** Neue PeerTube Docker Images enthalten CLI Pfade nicht mehr.

### 7. RAM Vollauslastung & Video Playback Probleme

| Symptom | Ursache |
|---------|---------|
| RAM 100% Auslastung | FFmpeg parallele Encodes |
| Videos starten nicht / hängen | Keine Worker Limits |
| Hohe CPU + RAM Last | PeerTube Transcoding ohne Begrenzung |

---

## Root Cause

> **PeerTube befindet sich in einem inkonsistenten Installationszustand:**
> - DB wurde teilweise initialisiert
> - Installer läuft erneut
> - Konflikt durch bereits existierende Daten

---

## Fehlerkette

```
1. Container startet
2. Installer läuft
3. Actor wird erstellt
4. Existiert bereits in DB
5. PostgreSQL blockiert (unique constraint)
6. Container crash
7. Restart Loop
```

---

## Datenbankzustand

| Tabelle | Zustand |
|---------|---------|
| Actor | ✅ vorhanden (accounts/peertube) |
| Application | ❌ leer |
| Server | ❌ leer |

---

## Bewertung

| Bereich | Status |
|---------|--------|
| PostgreSQL | ✅ läuft |
| Redis | ✅ läuft |
| PeerTube | ⚠️ instabil / teilweise stabil |
| DB Struktur | ✅ vorhanden |
| Installationsstatus | ❌ inkonsistent |
| Admin Login | ❌ unklar |

---

## Lösungen

### 🟢 Option 1 (empfohlen)

**DB korrigieren + Installation sauber abschließen**

### 🔴 Option 2

**Komplette Neuinstallation**

```bash
docker compose down -v
docker compose up -d
```

---

## Weitere Probleme im Verlauf

### Setup Wizard Problem

| Attribut | Wert |
|----------|------|
| Symptom | Kein Wizard sichtbar |
| Ursache | `user` Tabelle nicht leer |
| Lösung | DB Reset (`peertube_peertube_db` gelöscht) |

### Login Problem

- `root` existiert
- Passwort funktionierte nicht

> **Ursache:** Falscher bcrypt Hash oder inkompatibler Hash

### CLI Tool Problem

- `peertube.js` nicht vorhanden
- Neue Docker Images haben anderen Aufbau

### Postgres Login Problem

- Falscher User (`postgres`)
- Richtig: `peertube`

### RAM + Transcoding Problem

- FFmpeg erzeugt hohe Last
- Keine Worker Limits
- Videos erst nach Encoding abspielbar

---

## Aktueller Systemzustand

| Komponente | Status |
|-----------|--------|
| PeerTube | ✅ läuft |
| PostgreSQL | ✅ aktiv |
| Redis | ✅ aktiv |
| DB neu initialisiert | ✅ |
| Admin vorhanden (root) | ✅ |
| Login ggf. noch zu fixen | ⚠️ |
| Transcoding aktiv (RAM Last) | ⚠️ |

---

## Bekannte Schwachstellen

- Kein SMTP konfiguriert
- Keine Resource Limits (RAM/CPU)
- Initiale Passwortprobleme
- CLI Tools im Container nicht vorhanden

---

## Empfohlene Verbesserungen

### Stabilität

- Docker RAM/CPU Limits setzen
- Transcoding Worker begrenzen

### Sicherheit

- Neues Admin Konto statt root
- HTTPS (Let's Encrypt)

### Betrieb

- Backup PostgreSQL Volume
- Monitoring (CPU/RAM)

---

## Fazit

| Status | Bemerkung |
|--------|-----------|
| ✅ | Docker Stack erfolgreich aufgebaut |
| ✅ | Datenbank initialisiert |
| ✅ | PeerTube grundsätzlich funktionsfähig |
| ⚠️ | Setup inkonsistent durch DB Zustand |
| ⚠️ | Login & Performance noch optimierbar |

---

*Dokumentation erstellt am: 2026-07-07 | Server: peertube*
