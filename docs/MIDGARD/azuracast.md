# Bestandteil von Midgard.

**Midgard** ist die sichtbare Welt deines Homelabs. Folgende Dienste laufen hier:

- **AzuraCast** – Webradio‑Plattform  
- **Shortwave** – (optionaler Client)  
- **Gitea** – Git‑Hosting  
- **Passbolt** – Passwortmanager  
- **Mailserver** – E‑Mail‑Dienst  
- **Docker Services** – weitere containerisierte Anwendungen  

---

## Projektübersicht

**MapOS Wave – Webradio mit AzuraCast**

Ziel ist die Installation und Konfiguration einer selbst gehosteten Webradio-Plattform mit **AzuraCast** auf **Ubuntu Server 26.04 LTS**.  

Die Plattform ermöglicht:

- Verwaltung einer eigenen Radiostation  
- Verwaltung einer Musikbibliothek  
- Automatische Musikwiedergabe über den AutoDJ  
- Streaming im lokalen Netzwerk oder über das Internet  
- Verwaltung über eine komfortable Weboberfläche  

---

## Systemumgebung

| Komponente | Beschreibung |
| :--- | :--- |
| Betriebssystem | Ubuntu Server 26.04 LTS |
| Virtualisierung | Eigenes Homelab |
| Hostname | `azuracast` |
| Containerisierung | Docker |
| Orchestrierung | Docker Compose v2 |
| Software | AzuraCast Stable |
| Streamingserver | Icecast 2.4 |
| AutoDJ | Liquidsoap |
| Musikformat | MP3 |
| Radiostation | MapOS Wave |
| Stationsname | `mapos_wave` |

---

## Voraussetzungen

- Ubuntu Server 26.04 LTS installiert  
- System vollständig aktualisiert  
- Docker installiert  
- Docker Compose v2 installiert  
- Netzwerkverbindung vorhanden  

---

## System aktualisieren

```bash
sudo apt update
sudo apt upgrade -y

Hostname konfigurieren
bash

sudo hostnamectl set-hostname azuracast
hostnamectl

Installationsverzeichnis erstellen
bash

sudo mkdir -p /var/azuracast
cd /var/azuracast

AzuraCast installieren
bash

sudo curl -fsSL https://raw.githubusercontent.com/AzuraCast/AzuraCast/main/docker.sh -o docker.sh
sudo chmod +x docker.sh
sudo ./docker.sh install

Installationsoptionen
Einstellung	Wert
Sprache	Deutsch (de_DE)
Release Channel	Stable
Docker Images	Aktiviert
Automatische Updates	Aktiviert
Bot-Schutz	Aktiviert
Standardports	Ja
Netzwerkports
Dienst	Port
HTTP	80
HTTPS	443
SFTP	2022
Icecast	8000–8496
Installation erfolgreich

Nach Abschluss des Installationsskripts sind alle Container gestartet.
Du kannst nun die Weboberfläche unter http://<DEINE_SERVER_IP> aufrufen.
Die Standard-Anmeldedaten werden im Terminal während der Installation angezeigt.
Ersteinrichtung

    Rufe http://<DEINE_SERVER_IP> im Browser auf.

    Lege den Administrator‑Account an (Benutzername, E‑Mail, Passwort).

    Nach der Anmeldung gelangst du in das Dashboard.

Radiostation konfigurieren

Nach der Erstanmeldung erstellst du deine erste Radiostation:
Einstellung	Wert
Name	MapOS Wave
Kurzname	mapos_wave
Streamingserver	Icecast 2.4
AutoDJ	Liquidsoap

Streaming‑Quelle:

    Der Server stellt automatisch einen Icecast‑Mountpunkt bereit.

    Standard‑Mount: /radio.mp3

    Der Stream ist unter http://<SERVER_IP>:8000/radio.mp3 erreichbar.

AutoDJ konfigurieren

Der AutoDJ (Liquidsoap) verwaltet die automatische Musikwiedergabe:
Einstellung	Wert
Crossfade	Smart Mode
Überblendzeit	5 Sekunden
Queue	5 Titel
Leistungsprofil	Ausgeglichen

Zusätzliche Optionen (in der Weboberfläche unter Einstellungen → AutoDJ):

    ReplayGain – automatische Lautstärkeanpassung

    Duplikaterkennung – vermeidet doppelte Titel innerhalb der letzten X Songs

    Fallback‑Playlist – wird abgespielt, wenn keine aktuelle Playlist verfügbar ist

Musikbibliothek

Die Musikbibliothek kann auf verschiedene Weisen befüllt werden:

    Weboberfläche – Einzelne Dateien hochladen

    SFTP – Zugriff auf /var/azuracast/stations/mapos_wave/media (Port 2022)

    Drag & Drop – im Dateimanager der Weboberfläche

Unterstützte Formate: MP3, FLAC, AAC, OGG, WAV.
Nach dem Hochladen werden die Metadaten automatisch analysiert und indiziert.
Playlist konfigurieren

Erstelle Playlists für verschiedene Einsatzzwecke:

    Standard‑Playlist – läuft im normalen Betrieb

    Fallback‑Playlist – bei Leerstand oder Fehlern

    Schedule‑Playlist – zeitgesteuerte Sendungen (z. B. Nachrichten zur vollen Stunde)

Tipps:

    Vergib aussagekräftige Namen (z. B. Main_Rotation, Schlager, Rock)

    Lege die Gewichtung fest (häufigkeit der Songs aus dieser Playlist)

    Aktiviere „Shuffle“ für zufällige Reihenfolge

Öffentliche Stationsseite

AzuraCast generiert automatisch eine öffentliche Seite für deine Station.
Diese ist unter http://<SERVER_IP>/station/mapos_wave erreichbar.

Du kannst das Aussehen anpassen:

    Logo – hochladen (PNG oder JPG)

    Farben – im Theme‑Editor anpassen

    Widgets – z. B. „Jetzt läuft“, „Nächste Titel“, „Playlist“ ein‑/ausblenden

Für die Einbindung auf einer externen Webseite stellt AzuraCast fertige Embed‑Codes bereit (iFrame, JavaScript-Widget).
Systemprüfung

Überprüfe den Status aller Dienste:
bash

sudo docker compose -f /var/azuracast/docker-compose.yml ps

Oder in der Weboberfläche: System → Status – dort siehst du alle Container mit ihrem Zustand.

Sollte ein Dienst nicht laufen, hilft:
bash

sudo docker compose -f /var/azuracast/docker-compose.yml logs <dienstname>

Zugriff
Zweck	Adresse / Methode
Weboberfläche (Admin)	http://<SERVER_IP>
Öffentliche Station	http://<SERVER_IP>/station/mapos_wave
Stream (MP3)	http://<SERVER_IP>:8000/radio.mp3
SFTP (Musik)	sftp://<SERVER_IP>:2022 (Benutzer: azuracast, Passwort: siehe Installation)
API (für Entwickler)	http://<SERVER_IP>/api
Sicherheitsmaßnahmen

    Firewall – öffne nur die benötigten Ports (80, 443, 2022, 8000–8496)

    HTTPS – richte Let's Encrypt über die Weboberfläche ein (unter System → SSL)

    Regelmäßige Updates – aktiviere automatische Updates (in den Installationsoptionen bereits gesetzt)

    Starke Passwörter – für Admin, SFTP und Icecast‑Mountpunkte

    Backup – siehe nächster Punkt

Empfohlene Erweiterungen

    Monitoring – Prometheus + Grafana zur Überwachung von CPU, RAM und Netzwerk

    Log‑Analyse – ELK‑Stack oder Graylog für zentrale Logauswertung

    Zusätzliche Stream‑Ausgaben – z. B. AAC‑ oder OGG‑Stream über Icecast konfigurieren

    Mobile App – eigene App mit der öffentlichen API von AzuraCast

Backup

Erstelle regelmäßig ein vollständiges Backup:
bash

cd /var/azuracast
sudo ./docker.sh backup

Das Backup enthält:

    Datenbank (MariaDB)

    Konfigurationsdateien (Icecast, Liquidsoap)

    Station‑spezifische Einstellungen

    (optional) die Musikbibliothek – diese kann separat gesichert werden (z. B. mit rsync)

Wiederherstellung:
bash

sudo ./docker.sh restore /pfad/zum/backup.tar.gz

Wartung

Regelmäßige Wartungsaufgaben:
bash

cd /var/azuracast

# Updates einspielen
sudo ./docker.sh update

# Container neu starten (z.B. nach Konfigurationsänderungen)
sudo docker compose restart

# Logs anzeigen (alle Dienste)
sudo docker compose logs -f

# Nur Logs eines bestimmten Dienstes (z.B. AutoDJ)
sudo docker compose logs -f azuracast_auto_dj

Fehlerbehebung
Problem	Mögliche Lösung
Stream ist nicht erreichbar	Prüfe sudo docker ps – läuft Icecast?
Prüfe ss -tulpn | grep 8000 – ist der Port offen?
AutoDJ spielt keine Musik	Checke die Musikbibliothek – sind Titel vorhanden?
Prüfe sudo docker exec azuracast supervisorctl status
Weboberfläche langsam	Erhöhe den Arbeitsspeicher für Docker oder skaliere Container neu
Fehler beim Hochladen von Musik	Stelle sicher, dass die SFTP‑Zugangsdaten korrekt sind und genügend Speicherplatz vorhanden ist
Allgemeine Loginschwierigkeiten	Setze das Admin‑Passwort über das Terminal zurück: sudo ./docker.sh reset-admin
Projektfazit

Mit AzuraCast wurde erfolgreich eine moderne Docker-basierte Webradio-Plattform aufgebaut.
Das Projekt eignet sich hervorragend als Homelab- und Lernprojekt für Linux, Docker, Streaming-Technologien und Serveradministration.

Highlights:

    Einfache Installation dank Docker

    Umfangreiche Verwaltung über eine intuitive Weboberfläche

    Flexibilität durch zahlreiche Konfigurationsoptionen

    Stabilität durch erprobte Komponenten (Icecast, Liquidsoap)

Erreichte Projektziele

    ✅ Installation von AzuraCast auf Ubuntu Server 26.04 LTS

    ✅ Einrichtung einer eigenen Radiostation „MapOS Wave“

    ✅ Konfiguration des AutoDJ mit Crossfade und Playlist‑Steuerung

    ✅ Einrichtung der Musikbibliothek mit Upload‑Möglichkeiten (Web, SFTP)

    ✅ Bereitstellung eines öffentlichen Streams im lokalen Netzwerk

    ✅ Grundlegende Sicherheitsmaßnahmen (Firewall, HTTPS geplant)

    ✅ Backup‑ und Wartungsroutine implementiert

    ✅ Dokumentation der gesamten Installation und Konfiguration