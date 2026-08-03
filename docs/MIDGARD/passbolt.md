# 🔐 Passbolt {: #passbolt-main }

# Passbolt Community Edition – Installation auf Ubuntu Server 26.04 LTS

> Diese Dokumentation beschreibt die Installation eines eigenen Passwortmanagers auf einem Ubuntu Server ohne öffentliche Domain. Der Zugriff erfolgt intern über die IP-Adresse.

---

## Systemumgebung

| Attribut         | Wert                          |
|------------------|-------------------------------|
| Betriebssystem   | Ubuntu Server 26.04 LTS       |
| Hostname         | passbolt                      |
| Virtualisierung  | KVM                           |
| Architektur      | x86_64                        |
| Installation     | Docker / Docker Compose       |
| Anwendung        | Passbolt Community Edition    |
| Datenbank        | MariaDB 10.11                 |
| Webserver        | Nginx (innerhalb des Containers) |
| Zugriff          | HTTPS über interne IP-Adresse |
| Server-IP        | `xxx.xxx.xxx.xxx`               |
| Zugriffs-URL     | `https://xxx.xxx.xxx.xxx`       |

---

## Hardware

| Komponente | Ausstattung |
|------------|-------------|
| Festplatte | 30 GB       |
| CPU        | nach Bedarf |
| RAM        | nach Bedarf |

> Für einen privaten Server oder kleines Team ist diese Ausstattung ausreichend.

---

## Schritt 1 – Ubuntu aktualisieren

System aktualisieren:

```bash
sudo apt update
sudo apt upgrade -y
```

Systeminformationen prüfen:

```bash
lsb_release -a
hostnamectl
```

---

## Schritt 2 – Docker prüfen

Docker war bereits installiert.

Docker Version prüfen:

```bash
docker --version
```

Ausgabe:

```
Docker version 29.x.x
```

Docker Compose prüfen:

```bash
docker compose version
```

Ausgabe:

```
Docker Compose version 2.xx.x
```

Dienststatus:

```bash
sudo systemctl status docker
```

Ergebnis:

```
Active: active (running)
```

> Docker und Docker Compose waren bereits vorinstalliert.

---

## Schritt 3 – Passbolt Verzeichnis erstellen

Arbeitsverzeichnis anlegen:

```bash
sudo mkdir -p /opt/passbolt
cd /opt/passbolt
```

---

## Schritt 4 – Passbolt Docker Repository laden

Repository herunterladen:

```bash
sudo git clone https://github.com/passbolt/passbolt_docker.git .
```

Prüfen:

```bash
ls -la
```

---

## Schritt 5 – Docker Compose vorbereiten

Community Edition Compose-Datei kopieren:

```bash
sudo cp docker-compose/docker-compose-ce.yaml docker-compose.yml
```

Kontrolle:

```bash
ls -lh docker-compose.yml
```

Ergebnis:

```
docker-compose.yml
```

---

## Schritt 6 – Konfiguration anpassen

Die Standardadresse:

```
https://passbolt.local
```

wurde ersetzt durch:

```
https://xxx.xxx.xxx.xxx
```

Änderung:

```bash
sudo sed -i 's#https://passbolt.local#https://xxx.xxx.xxx.xxx#g' docker-compose.yml
```

> **Wichtig:** Das Datenbankpasswort wurde angepasst und muss in beiden Stellen identisch sein:
>
> - `MYSQL_PASSWORD`
> - `DATASOURCES_DEFAULT_PASSWORD`

---

## Schritt 7 – Passbolt starten

Container starten:

```bash
sudo docker compose up -d
```

Docker lädt automatisch:

- MariaDB Container
- Passbolt Container

Ergebnis:

```
✔ Network passbolt_default Created
✔ Volume passbolt_database_volume Created
✔ Volume passbolt_gpg_volume Created
✔ Volume passbolt_jwt_volume Created
✔ Container passbolt-db-1 Started
✔ Container passbolt-passbolt-1 Started
```

> Die Container werden automatisch im Hintergrund gestartet (`-d` = detached mode).

---

## Schritt 8 – Container prüfen

Status:

```bash
sudo docker ps
```

Aktive Container:

| Container             | Ports        |
|-----------------------|--------------|
| passbolt-passbolt-1   | 80/tcp, 443/tcp |
| passbolt-db-1         | –            |

---

## Schritt 9 – Installation prüfen

Logs anzeigen:

```bash
sudo docker logs passbolt-passbolt-1 --tail=50
```

Erfolgreiche Installation:

```
Passbolt installation success! Enjoy! ☮
```

Dienste:

```
nginx entered RUNNING state
php-fpm entered RUNNING state
```

> Beide Dienste (nginx und php-fpm) müssen im RUNNING-Zustand sein.

---

## Schritt 10 – Administrator erstellen

Admin-Benutzer anlegen:

```bash
sudo docker exec -it passbolt-passbolt-1 \
  su -m -c "/usr/share/php/passbolt/bin/cake passbolt register_user \
    -u admin@passbolt.local \
    -f Vorname \
    -l Nachname \
    -r admin" -s /bin/bash www-data
```

> Danach wurde ein Einrichtungslink erzeugt. Dieser Link wurde im Browser geöffnet.

---

## Anmeldung

Adresse:

```
https://xxx.xxx.xxx.xxx
```

> **Warnung:** Da keine öffentliche Domain verwendet wird, erscheint eine Zertifikatswarnung.
>
> **Grund:**
> - selbstsigniertes Zertifikat
> - keine öffentliche CA
>
> Für ein internes Netzwerk ist dies akzeptabel.

---

## Verwendete Docker Volumes

| Volume                     | Inhalt        |
|----------------------------|---------------|
| passbolt_database_volume   | Datenbank     |
| passbolt_gpg_volume        | GPG Schlüssel |
| passbolt_jwt_volume        | JWT Schlüssel |

> **Wichtig:** Diese Volumes müssen regelmäßig gesichert werden!

---

## Backup Empfehlung

Wichtige Daten:

- Docker Compose Datei
- MariaDB Volume
- GPG Volume
- JWT Volume

Beispiel – Volumes auflisten:

```bash
sudo docker volume ls
```

> **Warnung:** Backup der Volumes regelmäßig durchführen!

---

## Wartung

### Container aktualisieren

```bash
cd /opt/passbolt
sudo docker compose pull
sudo docker compose up -d
```

### Container prüfen

```bash
sudo docker ps
```

### Logs einsehen

```bash
sudo docker logs passbolt-passbolt-1
```

---

## Ergebnis

Passbolt Community Edition wurde erfolgreich installiert.

Eigenschaften:

- ✅ eigener Passwortmanager
- ✅ interne HTTPS-Verbindung
- ✅ Docker-basiert
- ✅ MariaDB Datenbank
- ✅ Open-Source
- ✅ selbst gehostet

> Die Installation ist bereit für die Verwaltung von Passwörtern und sicheren Zugangsdaten.

---

*Ende der Dokumentation*
