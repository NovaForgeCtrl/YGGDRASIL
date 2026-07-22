# Shortwave auf Raspberry Pi 5

## Projektziel

Installation und Einrichtung von **Shortwave** auf einem Raspberry Pi 5 mit Touchdisplay.

Ziel ist der Betrieb eines eigenständigen Internetradios mit lokaler Bedienung über den Touchscreen und Audioausgabe über Bluetooth.

| Ziel | Beschreibung |
|---|---|
| Internetradio | Direkte Wiedergabe auf dem Raspberry Pi |
| Audioausgabe | Über Bluetooth-Kopfhörer |
| Bedienung | Über Touchdisplay |
| Verwaltung | Per SSH |

---

# Hardware

Verwendete Hardware:

- Raspberry Pi 5
- Raspberry Pi OS
- Touchdisplay
- Bluetooth-Kopfhörer

---

# Bluetooth einrichten

## Geräte suchen

Bluetooth-Konfiguration starten:

```bash
bluetoothctl
```

Gefundenes Gerät:

```text
XX:XX:XX:XX:XX:XX  Bluetooth-Audio-Gerät
```

> Hinweis: MAC-Adressen wurden aus Datenschutzgründen anonymisiert.

---

## Bluetooth-Status prüfen

```bash
info XX:XX:XX:XX:XX:XX
```

Beispielausgabe:

```text
Paired: yes
Bonded: yes
Trusted: yes
Connected: no
```

!!! tip
    Die Bluetooth-Kopplung wurde erfolgreich eingerichtet.

---

# Audiotest

Zur Überprüfung der Audioausgabe:

```bash
speaker-test -t wav
```

!!! success
    Audiowiedergabe erfolgreich getestet.

---

# PipeWire prüfen

Aktuelle Audio-Ausgänge anzeigen:

```bash
pactl list short sinks
```

Beispiel:

```text
bluez_output.BLUETOOTH_DEVICE_ID
```

!!! tip
    Bluetooth-Audioausgabe wurde von PipeWire erkannt.

---

# Shortwave installieren

## 1. Flatpak installieren

```bash
sudo apt install flatpak
```

---

## 2. Flathub hinzufügen

```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

---

## 3. Shortwave installieren

```bash
flatpak install flathub de.haeckerfelix.Shortwave
```

!!! warning
    Die GNOME Runtime benötigt zusätzlichen Speicherplatz (mehrere hundert MB).
    
    Während der Installation kann eine SSH-Verbindung kurzzeitig abbrechen.
    Die Installation kann danach fortgesetzt werden.

---

## 4. Installation prüfen

```bash
flatpak list
```

---

## 5. Shortwave starten

```bash
flatpak run de.haeckerfelix.Shortwave
```

!!! success
    Shortwave wurde erfolgreich gestartet und konnte über das Touchdisplay bedient werden.

---

# Fehlerbehebung

## Problem 1: "Unable to reach station data"

### Symptom

Beim ersten Start erschien:

```text
Unable to reach station data
```

---

## Ursachenprüfung

| Prüfung | Befehl | Ergebnis |
|---|---|---|
| Internetverbindung | `ping google.de` | ✅ OK |
| Radio Browser Erreichbarkeit | `ping radio-browser.info` | ✅ OK |
| API-Test | `curl API-Adresse` | ✅ Antwort erhalten |
| Zeitsynchronisation | `timedatectl` | ✅ korrekt |
| Flatpak Berechtigungen | `flatpak info --show-permissions de.haeckerfelix.Shortwave` | ✅ Netzwerk + Audio erlaubt |

---

## Lösung

Nach einem Neustart von Shortwave funktionierte die Verbindung.

Der Startbildschirm:

```text
Welcome to Shortwave
```

wurde angezeigt.

---

# Problem 2: Keine Suchfunktion

## Symptom

Die Anwendung zeigte nur:

```text
Add new station
```

Die normale Sendersuche war nicht verfügbar.

## Lösung

Sender wurden manuell hinzugefügt.

---

# Problem 3: GTK-Warnungen

## Symptom

Beim Start erschienen Meldungen:

```text
Gtk-WARNING
Unable to acquire accessibility bus

Creating portal monitor failed
```

!!! warning
    Diese Meldungen können auf Raspberry Pi OS mit GTK-Anwendungen auftreten und haben keine Auswirkungen auf die Funktion.

---

# Getestete Streams

## Deutschland

| Sender | Status |
|---|---|
| Internetradio-Teststream | ✅ Erfolgreich getestet |

---

## Großbritannien

| Sender | Status |
|---|---|
| BBC World Service | ✅ Erfolgreich getestet |

---

## Japan

| Sender | Status | Bemerkung |
|---|---|---|
| Verschiedene NHK Streams | ⚠️ Unterschiedlich | Einige Streams liefen nicht stabil |

!!! note
    Die Probleme lagen an einzelnen HLS-Streams und nicht am Raspberry Pi.

---

# API-Tests

Radio Browser API getestet:

```bash
curl "https://example.api.radio-browser.info/json/stations/search?name=NHK&limit=10"
```

Weitere Suche:

```bash
curl "https://example.api.radio-browser.info/json/stations/search?name=Tokyo&limit=20"
```

!!! success
    Senderdaten konnten erfolgreich über die API gefunden werden.

---

# Zusammenfassung

## Erfolgreich getestet

- ✅ Internetverbindung
- ✅ Bluetooth
- ✅ PipeWire Audioausgabe
- ✅ Touchdisplay
- ✅ Shortwave Installation
- ✅ Flatpak Einrichtung
- ✅ Radio Browser API Zugriff

---

## Aufgetretene Probleme

- SSH-Verbindung während großer Flatpak-Installation kurzzeitig getrennt
- GNOME Runtime benötigt zusätzlichen Speicherplatz
- GTK-Warnungen ohne Funktionsauswirkung
- Stationskatalog beim ersten Start nicht erreichbar
- Einige HLS-Streams liefen nicht stabil
- Einige Streams bestanden nur aus kurzen Loops

---

# Aktueller Stand

!!! success "Projektstatus"
    ✅ Erfolgreich abgeschlossen

Der Raspberry Pi 5 kann nun als eigenständiges Internetradio verwendet werden.

Eigenschaften:

- Audioausgabe über Bluetooth
- Bedienung über Touchdisplay
- Verwaltung über SSH
- Nutzung von Shortwave als Radio-Anwendung

Das Projekt dient als Dokumentation für Raspberry Pi, Bluetooth-Audio und Linux Desktop-Anwendungen.