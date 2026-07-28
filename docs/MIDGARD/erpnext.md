# ERPNext v15 Installation auf Debian 12



<html><h1 id="erpnext-main">ERPNext v15</h1></html>
---

## 📋 Server-Voraussetzungen

| Umgebung | Minimum |
|----------|---------|
| Test / Entwicklung | 2 vCPU, 4 GB RAM, 20 GB SSD |
| Produktion (bis 25 User) | 4 vCPU, 8 GB RAM, 40 GB SSD |

---

## Schritt 1: System vorbereiten

Als `root` oder mit `sudo` ausführen:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y   git curl wget   python3-dev python3-pip python3-venv   build-essential   libffi-dev libssl-dev   libjpeg-dev zlib1g-dev   libmariadb-dev-compat libmariadb-dev pkg-config   xvfb libfontconfig1 fontconfig   redis-server supervisor nginx   cron
```

> ⚠️ **Hinweis:** `libmysqlclient-dev` existiert nicht mehr in Debian 12. Stattdessen `libmariadb-dev-compat` und `libmariadb-dev` verwenden.

---

## Schritt 2: MariaDB 10.11 installieren

```bash
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash -s -- --mariadb-server-version=10.11
sudo apt update
sudo apt install -y mariadb-server mariadb-client
sudo mysql_secure_installation
```

### Antworten während der Installation

| Frage | Antwort |
|-------|---------|
| `Switch to unix_socket authentication?` | **N** |
| `Set root password?` | **Y** *(Passwort merken!)* |
| Alle weiteren Fragen | **Y** |

### MariaDB-Konfiguration für ERPNext

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-erpnext.cnf
```

Inhalt:

```ini
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
innodb-file-format = barracuda
innodb-file-per-table = 1
innodb-large-prefix = 1
innodb_buffer_pool_size = 2G
wait_timeout = 28800
interactive_timeout = 28800

[mysql]
default-character-set = utf8mb4

[client]
default-character-set = utf8mb4
```

Dienste neustarten und aktivieren:

```bash
sudo systemctl restart mariadb
sudo systemctl enable mariadb
```

---

## Schritt 3: Node.js 18 LTS installieren

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
node --version   # Muss v18.x.x anzeigen
sudo npm install -g yarn
yarn --version
```

---

## Schritt 4: wkhtmltopdf installieren

```bash
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.bookworm_amd64.deb
sudo apt install -y ./wkhtmltox_0.12.6.1-3.bookworm_amd64.deb
wkhtmltopdf --version
```

> ✅ Die Ausgabe muss enthalten: **`(with patched qt)`**

---

## Schritt 5: User erstellen

```bash
sudo adduser --disabled-password --gecos '' frappe
sudo usermod -aG sudo frappe
sudo visudo
```

Folgende Zeile hinzufügen:

```
frappe ALL=(ALL) NOPASSWD: ALL
```

---

## Schritt 6–7: Bench & ERPNext installieren

```bash
sudo su - frappe
pip3 install --break-system-packages --user frappe-bench
export PATH="$PATH:~/.local/bin"
echo 'export PATH="$PATH:~/.local/bin"' >> ~/.bashrc
```

### uv installieren (wird von Bench 5.30+ benötigt)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.cargo/bin:$PATH"
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
```

### Bench initialisieren

```bash
bench --version   # Sollte 5.x.x zeigen
bench init --frappe-branch version-15 frappe-bench
cd ~/frappe-bench
bench get-app erpnext --branch version-15
bench get-app hrms --branch version-15
```

---

## Schritt 8: Site erstellen

```bash
bench new-site xxx.xxx.xxx.xxx \
  --mariadb-root-password 'MARIA_DB_ROOT_PASS' \
  --admin-password 'ERP_ADMIN_PASS' \
  --mariadb-user-host-login-scope='%'
```

### Apps installieren

```bash
bench --site xxx.xxx.xxx.xxx install-app erpnext
bench --site xxx.xxx.xxx.xxx install-app hrms
bench use xxx.xxx.xxx.xxx
bench --site xxx.xxx.xxx.xxx enable-scheduler
```

---

## Schritt 9: Produktion (Nginx + Supervisor)

```bash
exit  # zurück zu root/sudo-User
sudo bench setup production frappe --yes
sudo bench setup nginx
```

### Nginx-Config anpassen

> `log_format main` fehlt in der Standard-Debian-12-Nginx-Config.

```bash
sudo tee /etc/nginx/nginx.conf << 'EOF'
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;

    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
EOF
```

### Standard-Site entfernen, ERPNext aktivieren

```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -sf /home/frappe/frappe-bench/config/nginx.conf /etc/nginx/sites-enabled/frappe-bench
```

### Berechtigungen fixen *(wichtig!)*

```bash
sudo chown -R frappe:frappe /home/frappe/frappe-bench/sites
sudo chmod -R 755 /home/frappe/frappe-bench/sites
sudo chmod 755 /home/frappe
sudo chmod 755 /home/frappe/frappe-bench
sudo nginx -t
sudo systemctl reload nginx
```

### Supervisor einrichten

```bash
sudo ln -sf /home/frappe/frappe-bench/config/supervisor.conf /etc/supervisor/conf.d/frappe-bench.conf
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl status
```

---

## Schritt 10: Assets bauen

```bash
sudo su - frappe
cd ~/frappe-bench
bench build
```

---

## 🌐 Zugriff

| URL | Beschreibung |
|-----|-------------|
| `http://xxx.xxx.xxx.xxx` | ERPNext Login |
| `http://xxx.xxx.xxx.xxx/app/home` | Dashboard |

| Feld | Wert |
|------|------|
| Benutzer | `administrator@administrator.de` |
| Passwort | Das bei `--admin-password` gesetzte |

---

## 🔧 Bekannte Probleme & Lösungen

| Problem | Lösung |
|---------|--------|
| `libmysqlclient-dev` nicht gefunden | `libmariadb-dev-compat` + `libmariadb-dev` verwenden |
| `externally-managed-environment` | `pip3 install --break-system-packages --user` oder `pipx` |
| `No such command 'build'` | Im `frappe-bench` Verzeichnis ausführen |
| `uv` nicht gefunden | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| `Permission denied` in Nginx | `sudo chmod -R 755 /home/frappe/frappe-bench/sites` |
| Scheduler disabled | `bench --site SITE enable-scheduler` |

---

## ⌨️ Wichtige Befehle

```bash
# Als frappe-User ausführen:
bench restart              # Alle Prozesse neustarten
bench migrate              # Nach Updates
bench build                # Assets neu bauen
bench --site SITE backup   # Backup erstellen
bench update               # ERPNext updaten
```

### Logs einsehen

```bash
sudo tail -f /var/log/nginx/error.log
sudo tail -f /home/frappe/frappe-bench/logs/web.log
```

---

*Erstellt für ERPNext v15 auf Debian 12 (Bookworm)*
