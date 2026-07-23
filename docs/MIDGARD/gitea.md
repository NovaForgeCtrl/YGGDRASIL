# Gitea Git-Server Dokumentation

> **Projekt:** Lokaler Git-Server mit Ubuntu Server 26.04  
> **Version:** 1.0  
> **System:** Ubuntu Server 26.04 LTS  
> **Dienst:** Gitea  
> **Bereitstellung:** Docker Compose  
> **Datenbank:** SQLite3  
> **Zweck:** Lokale Versionsverwaltung und Synchronisation mit GitHub

---


---

## 1. Projektübersicht

### Ziel

Auf einem eigenen Ubuntu Server wird ein lokaler Git-Server bereitgestellt.

Der Server dient zur Verwaltung und Versionierung von:

- Bash-Skripten
- PowerShell-Skripten
- Python-Projekten
- Sicherheitsregeln
- Firewall-Regeln
- Docker-Konfigurationen
- Dokumentationen
- Homelab-Projekten

### Die lokale Git-Infrastruktur ermöglicht

- ✅ Vollständige Kontrolle über eigene Projekte
- ✅ Interne Versionsverwaltung
- ✅ Backup-Möglichkeit
- ✅ Spätere Synchronisation mit GitHub

---

## 2. Architektur

```
                    ┌─────────────┐
                    │  Internet   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ GitHub.com  │
                    └──────┬──────┘
                           │
                      git push
                           │
                    ┌──────▼──────┐
                    │Ubuntu Server│
                    │  26.04 LTS  │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │    Gitea    │
                    │             │
                    │ Web :3000   │
                    │ SSH :2222   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │    Git      │
                    │ Repository  │
                    │             │
                    │ • Projekte  │
                    │ • Regeln    │
                    │ • Scripts   │
                    │ • Docs      │
                    └─────────────┘
```

---

## 3. Systemanforderungen

### Hardware Empfehlung

| Komponente | Empfehlung |
|------------|------------|
| CPU        | 2 vCPU     |
| RAM        | 4 GB       |
| Speicher   | 100 GB     |
| Netzwerk   | Statische IP |

---

## 4. Verwendete Software

| Software            | Zweck                          |
|---------------------|--------------------------------|
| Ubuntu Server 26.04 | Betriebssystem                 |
| Git                 | Versionsverwaltung             |
| Docker              | Containerisierung              |
| Docker Compose      | Verwaltung der Container       |
| Gitea               | Webbasierter Git-Server        |
| SQLite3             | Datenbank                      |

---

## 5. System vorbereiten

### System aktualisieren

```bash
sudo apt update
sudo apt upgrade -y
```

### Neustart

```bash
sudo reboot
```

---

## 6. Git Installation

### Installation

```bash
sudo apt install git -y
```

### Prüfen

```bash
git --version
```

**Beispiel Ausgabe:**
```
git version 2.x
```

---

## 7. Git Benutzer konfigurieren

### Name setzen

```bash
git config --global user.name "Benutzername"
```

### E-Mail setzen

```bash
git config --global user.email "benutzer@example.local"
```

### Kontrolle

```bash
git config --list
```

---

## 8. Docker Installation

### Benötigte Pakete

```bash
sudo apt install ca-certificates curl gnupg -y
```

> **Hinweis:** Docker Repository hinzufügen (offizielle Docker-Doku beachten)

### Installation

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

### Test

```bash
docker --version
```

---

## 9. Docker Benutzerrechte

### Benutzer zu Docker-Gruppe hinzufügen

```bash
sudo usermod -aG docker $USER
```

> **Wichtig:** Danach neu anmelden (ausloggen und wieder einloggen)

### Test

```bash
docker ps
```

---

## 10. Gitea Installation

### Verzeichnis erstellen

```bash
mkdir ~/gitea
cd ~/gitea
```

### Docker Compose Datei erstellen

```bash
nano docker-compose.yml
```

### Inhalt

```yaml
services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: always
    ports:
      - "3000:3000"
      - "2222:22"
    volumes:
      - ./data:/data
    environment:
      - USER_UID=1000
      - USER_GID=1000
```

---

## 11. Gitea starten

### Start

```bash
docker compose up -d
```

### Status prüfen

```bash
docker ps
```

**Erwartete Ausgabe:**
```
gitea   Up
```

---

## 12. Firewall Konfiguration

### UFW installieren

```bash
sudo apt install ufw -y
```

### SSH

```bash
sudo ufw allow ssh
```

### Gitea Webinterface

```bash
sudo ufw allow 3000/tcp
```

### Git SSH

```bash
sudo ufw allow 2222/tcp
```

### Aktivieren

```bash
sudo ufw enable
```

### Status

```bash
sudo ufw status
```

---

## 13. Gitea Webinstallation

### Aufrufen

```
http://SERVER-IP:3000
```

**Beispiel:**
```
http://192.168.1.10:3000
```

---

## 14. Datenbank Einstellungen

| Einstellung   | Wert                     |
|---------------|--------------------------|
| Datenbanktyp  | SQLite3                  |
| Datenbankpfad | `/data/gitea/gitea.db`   |

---

## 15. Server Einstellungen

| Einstellung   | Wert                          |
|---------------|-------------------------------|
| Server Domain | `192.168.1.10`                |
| SSH Port      | `2222`                        |
| HTTP Port     | `3000`                        |
| Basis URL     | `http://192.168.1.10:3000/`   |

---

## 16. Speicherstruktur

### Nach Installation

```
~/gitea/
└── data/
    ├── gitea/
    │   ├── gitea.db
    │   ├── conf/
    │   └── log/
    └── git/
        ├── repositories/
        └── lfs/
```

---

## 17. Erstes Repository erstellen

**Beispiel:** `Projekt`

### Inhalt

- `README.md`
- `Scripts/`
- `Dokumentation/`
- `Regeln/`

---

## 18. Lokales Repository verbinden

### Projekt erstellen

```bash
mkdir Projekt
cd Projekt
```

### Git initialisieren

```bash
git init
```

### Dateien hinzufügen

```bash
git add .
```

### Commit

```bash
git commit -m "Initial Commit"
```

### Remote hinzufügen

```bash
git remote add origin ssh://git@192.168.1.10:2222/benutzer/Projekt.git
```

### Push

```bash
git push -u origin main
```

---

## 19. Verbindung zu GitHub

### GitHub Remote hinzufügen

```bash
git remote add github git@github.com:BENUTZERNAME/Projekt.git
```

### Hochladen

```bash
git push github main
```

---

## 20. SSH Key für GitHub

### Erstellen

```bash
ssh-keygen -t ed25519
```

### Public Key anzeigen

```bash
cat ~/.ssh/id_ed25519.pub
```

### Bei GitHub hinterlegen

```
Settings → SSH Keys → New SSH Key
```

### Test

```bash
ssh -T git@github.com
```

---

## 21. Backup

### Gitea Backup

```bash
docker stop gitea
tar czvf gitea-backup.tar.gz ~/gitea/data
docker start gitea
```

---

## 22. Wartung

### Container aktualisieren

```bash
cd ~/gitea
docker compose pull
docker compose up -d
```

### Logs prüfen

```bash
docker logs gitea
```

---

## 23. Sicherheitsmaßnahmen

### Empfohlen

- ✅ SSH Keys verwenden
- ✅ Root Login deaktivieren
- ✅ Firewall aktivieren
- ✅ Regelmäßige Backups
- ✅ Updates durchführen
- ✅ HTTPS über Reverse Proxy später ergänzen

---

## 24. Erweiterungsmöglichkeiten

### Später möglich

- Nginx Reverse Proxy
- HTTPS Zertifikate
- Eigene Domain
- PostgreSQL statt SQLite
- CI/CD Runner
- Docker Registry
- Automatische GitHub Spiegelung
- Monitoring mit Grafana

---

## 🏁 Ergebnis

Der Server stellt eine eigene interne Git-Plattform bereit:

```
    Ubuntu Server 26.04
            +
        Docker
            +
        Gitea
            +
    GitHub Synchronisation
            =
    Professionelle Versionsverwaltung im Homelab
```

| Status           | ✅ Erfolgreich geplant und eingerichtet |
|------------------|----------------------------------------|
| Einsatzbereich   | Homelab / Entwicklungsumgebung         |

---

*Ende der Dokumentation*
