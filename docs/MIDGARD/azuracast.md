# 🎵 AZURACAST — Der Sender {: #azuracast-main }

> *"Jede Frequenz erzählt eine Geschichte. Jeder Stream verbindet Welten."*

---

## 📋 Systeminformationen

| Attribut | Wert |
|----------|------|
| **Status** | 🟢 Aktiv |
| **Version** | Stable |
| **Erstellt** | 2026 |
| **Zuletzt geändert** | 2026-07 |
| **Lizenz** | Open Source (Apache 2.0) |

---

## 🎯 Ziel / Zweck

Installation und Konfiguration einer selbst gehosteten Webradio-Plattform mit AzuraCast auf Ubuntu Server 26.04 LTS. Die Plattform ermöglicht das Verwalten von Musik, automatisches Abspielen per AutoDJ sowie das Bereitstellen eines Livestreams über das Netzwerk oder das Internet.

---

## 🛠️ Komponenten

[Icecast Server](icecast.md){: .world-link } · [Liquidsoap AutoDJ](liquidsoap.md){: .world-link } · [Docker Engine](docker.md){: .world-link } · [Weboberfläche](webui.md){: .world-link }

---

## 🏗️ Systemumgebung

| Komponente | Beschreibung |
|------------|--------------|
| **Betriebssystem** | Ubuntu Server 26.04 LTS |
| **Virtualisierung** | Eigenes Homelab |
| **Benutzer** | `<BENUTZER>` |
| **Hostname** | `<HOSTNAME>` |
| **Software** | Docker Engine |
| **Container** | AzuraCast (Stable) |
| **Streamingserver** | Icecast 2.4 |
| **AutoDJ** | Liquidsoap |
| **Musikbibliothek** | MP3-Dateien |
| **Station** | Demo Station |
| **Stations-Kurzname** | demo_station |

---

## 🔧 Voraussetzungen

- Ubuntu Server 26.04 LTS installiert
- System aktualisiert
- Docker installiert
- Docker Compose v2 installiert
- Server besitzt Netzwerkverbindung

---

## 🚀 Installation

### Hostname konfigurieren

```bash
sudo hostnamectl set-hostname <HOSTNAME>
hostnamectl
```

### Installationsverzeichnis erstellen

```bash
sudo mkdir -p /var/azuracast
cd /var/azuracast
```

### Installationsskript herunterladen

```bash
sudo curl -fsSL https://raw.githubusercontent.com/AzuraCast/AzuraCast/main/docker.sh -o docker.sh
sudo chmod +x docker.sh
```

### Installation starten

```bash
sudo ./docker.sh install
```

---

## ⚙️ Installationsoptionen

### Sprache

`de_DE`

### Release Channel

`Stable`

### Ports

| Dienst | Port |
|--------|------|
| HTTP | 80 |
| HTTPS | 443 |
| SFTP | 2022 |
| Radio | 8000–8496 |

### Docker Image Updates

`Ja`

### Bots blockieren

`Ja`

### System-URL

Während der Installation: `http://<ÖFFENTLICHE-IP>`

Später z. B.: `https://radio.example.de`

---

## ✅ Installation erfolgreich

```text
[OK] AzuraCast-Installation abgeschlossen!
```

Container prüfen:

```bash
sudo docker ps
```

---

## 🎛️ Ersteinrichtung

Weboberfläche: `http://<SERVER-IP>`

Administrator anlegen.

---

## 📻 Erste Radiostation

| Einstellung | Wert |
|-------------|------|
| **Name** | Demo Station |
| **Kurzname** | demo_station |
| **Streamingserver** | Icecast 2.4 |
| **AutoDJ** | Liquidsoap |

---

## 🎚️ AutoDJ-Konfiguration

| Einstellung | Wert |
|-------------|------|
| **Dienst** | Liquidsoap |
| **Crossfade** | Smart Mode |
| **Überblendungszeit** | 5 Sekunden |
| **AutoCue** | Deaktiviert |
| **ReplayGain** | Deaktiviert |
| **Post Processing** | Keine Nachbearbeitung |
| **Queue** | 5 Titel |
| **Zeichensatz** | UTF-8 |
| **Leistungsoptimierung** | Ausgeglichen |
| **Wiederholungsvermeidung** | 60 Minuten |

---

## 🎶 Musik hinzufügen

- Weboberfläche
- SFTP
- Drag & Drop

---

## 📋 Playlist

| Einstellung | Wert |
|-------------|------|
| **Name** | default |
| **Typ** | Allgemeine Rotation |

---

## 🔍 Dienste überprüfen

```bash
sudo docker exec azuracast supervisorctl status
```

Wichtige Dienste:

- mariadb
- redis
- nginx
- php-fpm
- station_1_backend
- station_1_frontend

---

## 🌐 Zugriff

`http://<SERVER-IP>`

Beispiel: `http://<LOKALE-IP>`

---

## 🔒 Sicherheitsmaßnahmen

Aktiviert:

- Stable Release
- automatische Docker-Updates
- Bot-Schutz
- UTF-8
- Docker-Container
- getrennte Streamingdienste

Empfohlen:

- HTTPS mit Let's Encrypt
- Firewall-Regeln
- regelmäßige Backups
- Reverse Proxy
- Monitoring mit Uptime Kuma

---

## 💾 Backup

```bash
cd /var/azuracast
sudo ./docker.sh backup
```

---

## 🔧 Wartung

```bash
cd /var/azuracast
sudo ./docker.sh update
sudo docker compose restart
sudo docker compose logs -f
```

---

## 🛠️ Fehlerbehebung

```bash
sudo docker exec azuracast supervisorctl status
sudo docker ps
ss -tulpn
```

Backend und Frontend müssen den Status **RUNNING** besitzen.

---

## 📝 Notizen

!!! note "Wichtig"
    - Container regelmäßig aktualisieren
    - Backups vor Updates erstellen
    - Ports im Router freigeben für externen Stream
    - Firewall-Regeln für Radio-Ports prüfen

---

## 🏆 Projektfazit

Mit AzuraCast wurde erfolgreich eine moderne Webradio-Plattform auf Ubuntu Server 26.04 LTS installiert. Die Installation basiert vollständig auf Docker und kombiniert den Streamingserver Icecast mit dem AutoDJ Liquidsoap. Durch die Containerisierung ist das System leicht wartbar, einfach zu sichern und gut skalierbar.

---

## ✅ Erreichte Projektziele

- Erfolgreiche Installation von AzuraCast
- Docker-basierter Betrieb
- Einrichtung einer Radiostation
- AutoDJ mit Liquidsoap eingerichtet
- Icecast erfolgreich konfiguriert
- Musikbibliothek aufgebaut
- Playlist erstellt
- Stream erfolgreich getestet
- Weboberfläche funktionsfähig
- Grundlage für einen dauerhaft betriebenen Radioserver geschaffen

---

> *"Der Rhythmus verbindet. Der Beat bleibt. Der Stream fließt ewig."*
