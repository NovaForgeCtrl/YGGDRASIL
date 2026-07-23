# Shortwave auf Raspberry Pi 5 - Projektdokumentation
Status: Erfolgreich abgeschlossen


---

## 1. Projektziel & Übersicht
Installation und Einrichtung von Shortwave auf einem Raspberry Pi 5 mit Touchdisplay.
Ziel ist ein eigenständiges Internetradio mit lokaler Bedienung über den Touchscreen und Audioausgabe über Bluetooth.

- **Internetradio:** Shortwave auf Raspberry Pi
- **Bedienung:** Touchdisplay
- **Audioausgabe:** Bluetooth-Kopfhörer
- **Verwaltung:** SSH-Zugriff
- **Betriebssystem:** Raspberry Pi OS

---

## 2. Hardware
- **Raspberry Pi:** Raspberry Pi 5
- **Betriebssystem:** Raspberry Pi OS
- **Display:** Touchdisplay
- **Audio:** Bluetooth-Kopfhörer
- **Netzwerk:** LAN / WLAN

---

## 3. Bluetooth Einrichtung

### Geräte suchen
Bluetooth-Konsole starten:
```bash
bluetoothctl
```
Geräte suchen:
```bash
scan on
```
Beispiel:
```text
XX:XX:XX:XX:XX:XX [Bluetooth-Gerätename]
```

### Verbindung prüfen
```bash
info XX:XX:XX:XX:XX:XX
```
Ergebnis:
```text
Paired: yes
Bonded: yes
Trusted: yes
Connected: no
```
✅ Bluetooth wurde erfolgreich eingerichtet.

---

## 4. Audiotest & PipeWire

### Audiotest
Test der Audioausgabe:
```bash
speaker-test -t wav
```
✅ Ton vorhanden

### PipeWire prüfen
Verfügbare Audioausgänge anzeigen:
```bash
pactl list short sinks
```
Bluetooth-Ausgabe erkannt:
```text
bluez_output.XX_XX_XX_XX_XX_XX.1
```
✅ Bluetooth-Audio über PipeWire funktioniert.

---

## 5. Shortwave Installation

### 5.1 Flatpak & Flathub einrichten
```bash
sudo apt install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

### 5.2 Shortwave installieren & starten
```bash
flatpak install flathub de.haeckerfelix.Shortwave
```
Während der Installation wird die GNOME Runtime geladen.
- Größe ca. 400 MB
- Bei SSH-Verbindungen kann die Verbindung während großer Downloads abbrechen
- Installation läuft normalerweise im Hintergrund weiter

Installation prüfen & starten:
```bash
flatpak list
flatpak run de.haeckerfelix.Shortwave
```
✅ Shortwave öffnet sich erfolgreich auf dem Touchdisplay.

---

## 6. Fehlerbehebung & Diagnose

### 6.1 Problem 1: "Unable to reach station data"
- **Symptom:** Beim ersten Start erscheint die Fehlermeldung.
- **Diagnose-Befehle:**
```bash
ping google.de
ping radio-browser.info
curl "https://de1.api.radio-browser.info/json/stations/search?limit=5"
timedatectl
flatpak info --show-permissions de.haeckerfelix.Shortwave
```
✅ Internet, API, Systemzeit und Berechtigungen (Netzwerk & PulseAudio) waren korrekt.
- **Lösung:** Nach einem Neustart von Shortwave funktionierte die Verbindung. Danach erschien das Willkommensfenster.

### 6.2 Problem 2: Keine Suchfunktion
- **Symptom:** Nach dem Start wurde nur *„Add new station“* angezeigt.
- **Lösung:** Sender wurden manuell hinzugefügt.

### 6.3 Problem 3: GTK Warnungen
Beim Start erschienen:
```text
Gtk-WARNING Unable to acquire accessibility bus
Creating portal monitor failed
```
- **GTK Accessibility:** Kosmetische Warnung
- **Portal Monitor:** Keine Funktionsbeeinträchtigung

✅ Shortwave funktioniert trotzdem vollständig.

---

## 7. Getestete Radiosender & API Tests

- 🇩🇪 **Deutschland:** Blasmusikradio (`https://stream.laut.fm/blasmusikradio_mit_bernd`) – ✅ Funktioniert
- 🇬🇧 **Großbritannien:** BBC World Service – ✅ Funktioniert
- 🇯🇵 **Japan:** NHK Streams (HLS Streams) – ⚠️ Teilweise (Loops)

**Radio Browser API Tests:**
```bash
curl "https://de1.api.radio-browser.info/json/stations/search?name=NHK&limit=10"
curl "https://de1.api.radio-browser.info/json/stations/search?name=Tokyo&limit=20"
```
✅ Japanische Sender erfolgreich gefunden. (Probleme bei einzelnen HLS-Streams lagen am Sender, nicht am Raspberry Pi).

---

## 8. Projektstatus & Fazit
- **Internetverbindung, Bluetooth Audio & PipeWire:** ✅ Erfolgreich
- **Touchdisplay & Flatpak Einrichtung:** ✅ Erfolgreich
- **Shortwave Installation & Radio Browser API:** ✅ Erfolgreich

**Bekannte Randnotizen:**
- SSH Abbruch während Flatpak Download (Normal bei großen Downloads)
- Große GNOME Runtime (Erwartet)
- GTK Warnungen (Nur kosmetisch)
- Einige japanische HLS Streams liefen in Schleifen (Senderproblem)

🎉 **Ergebnis:** Der Raspberry Pi 5 funktioniert erfolgreich als eigenständiges Internetradio mit Touch-Bedienung und Bluetooth-Audio!