

<html><h1 id="ollama-main"> Ollama & Open WebUI – Lokales KI-Setup unter Ubuntu</h1></html>

Komplette Einrichtung von **Ollama** (LLM-Backend) und **Open WebUI** (Chat-Oberfläche) auf Ubuntu Server 26.04 LTS – inklusive aller auftretenden Netzwerkprobleme und deren Lösungen.

---

## 1. Systemvoraussetzungen

| Komponente | Anforderung |
|------------|-------------|
| Betriebssystem | Ubuntu Server 24.04/26.04 LTS (auch als Proxmox-LXC-Container) |
| RAM | Mindestens 8 GB (16+ GB empfohlen) |
| Speicher | Je Modell 4–30 GB (SSD empfohlen) |
| GPU | Optional (NVIDIA mit CUDA beschleunigt stark, sonst CPU) |

**Verwendete Ports:**
- `11434/tcp` – Ollama API
- `8080/tcp` – Open WebUI (bei `--net=host`)

---

## 2. Installation

### 2.1 System aktualisieren

```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 Ollama installieren

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

> Ollama wird als systemd-Dienst eingerichtet und startet automatisch.

**Status prüfen:**
```bash
sudo systemctl status ollama
```

### 2.3 Docker installieren

```bash
sudo apt install -y docker.io docker-compose-v2
sudo usermod -aG docker $USER
```

> **Wichtig:** Nach `usermod` einmal abmelden (`exit`) und neu per SSH einloggen, damit Docker ohne `sudo` funktioniert.

---

## 3. Das zentrale Problem: Docker ↔ Ollama Netzwerk

### 3.1 Symptome

- Open WebUI startet, aber zeigt **keine Modelle** an
- In den Verbindungseinstellungen erscheint: *„Ollama: Network Problem“*
- `curl` aus dem Docker-Container in Ollama läuft in einen **Timeout**
- `docker0` Bridge zeigt Status `DOWN`

### 3.2 Ursachenanalyse

| Problem | Ursache |
|---------|---------|
| Ollama nicht erreichbar | Standardmäßig lauscht Ollama nur auf `127.0.0.1:11434` – Docker-Container können nicht zugreifen |
| `host.docker.internal` funktioniert nicht | Unter Linux wird dieser DNS-Name von Docker nicht zuverlässig aufgelöst |
| `172.17.0.1` nicht erreichbar | Die Docker-Bridge (`docker0`) ist inaktiv (`state DOWN`) |
| `--net=host` + Port 3000 | Funktioniert nicht, da Open WebUI intern auf `8080` lauscht; Port-Mapping entfällt bei Host-Netzwerk |

---

## 4. Lösung: Ollama global freigeben + Open WebUI im Host-Netzwerk

### 4.1 Schritt 1 – Ollama auf alle Schnittstellen binden

```bash
# Systemd-Override-Verzeichnis erstellen
sudo mkdir -p /etc/systemd/system/ollama.service.d

# Umgebungsvariable setzen, damit Ollama auf allen IPs lauscht
echo -e '[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"' |   sudo tee /etc/systemd/system/ollama.service.d/override.conf

# Dienst neu laden und neustarten
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

**Kontrolle:** Es muss `*:11434` (nicht `127.0.0.1:11434`) angezeigt werden:
```bash
sudo ss -tulpn | grep 11434
```

### 4.2 Schritt 2 – Open WebUI im Host-Netzwerk starten

> Die einzige unter Linux zuverlässige Methode, wenn die Docker-Bridge inaktiv ist oder Routing-Regeln blockieren.

```bash
# Alten Container stoppen und entfernen (falls vorhanden)
sudo docker stop open-webui 2>/dev/null
sudo docker rm open-webui 2>/dev/null

# Neu starten mit Host-Netzwerk
sudo docker run -d --net=host   -v open-webui:/app/backend/data   -e OLLAMA_BASE_URL=http://127.0.0.1:11434   --name open-webui   --restart always   ghcr.io/open-webui/open-webui:main
```

**Wichtig bei `--net=host`:**
- Der Container teilt sich das Netzwerk des Hosts
- Open WebUI ist dann unter Port **8080** erreichbar (nicht mehr 3000!)
- Kein Port-Mapping (`-p`) mehr nötig und auch nicht möglich

### 4.3 Schritt 3 – Firewall freigeben

```bash
sudo ufw allow 8080/tcp
sudo ufw reload
```

### 4.4 Schritt 4 – Aufruf im Browser

```
http://<SERVER-IP>:8080
```

> Beim ersten Start Admin-Konto anlegen. Die heruntergeladenen Ollama-Modelle erscheinen automatisch im Dropdown-Menü.

---

## 5. Modelle verwalten

### 5.1 Modell herunterladen (Terminal)

```bash
ollama run <modellname>
```

Beispiele:
```bash
ollama run llama3.1:8b        # Allrounder, ca. 4.9 GB
ollama run qwen2.5-coder:7b   # Code-Modell, ca. 4.7 GB
ollama run deepseek-r1:8b     # Reasoning/Logik, ca. 4.9 GB
```

> Nach dem Download erscheint ein interaktiver Chat im Terminal. Mit `/exit` beenden.

### 5.2 Installierte Modelle anzeigen

```bash
ollama list
```

### 5.3 Modell in der WebUI wechseln

Oben im Chat-Fenster über das Dropdown-Menü „Wählen Sie ein Modell“ auswählen.

---

## 6. Empfohlene kostenlose Coding-Modelle

| Modell | Größe | Stärken | Befehl |
|--------|-------|---------|--------|
| **qwen2.5-coder:7b** | ~4.7 GB | Code-Generierung, Bugfixing, 40+ Sprachen | `ollama run qwen2.5-coder:7b` |
| **qwen2.5-coder:14b** | ~9 GB | Besseres Verständnis, komplexere Aufgaben | `ollama run qwen2.5-coder:14b` |
| **deepseek-r1:8b** | ~4.9 GB | Reasoning – "denkt" vor der Antwort nach, ideal für Algorithmen | `ollama run deepseek-r1:8b` |
| **codestral** | ~22 GB | Fill-in-the-Middle, sehr gute Vervollständigung | `ollama run codestral` |
| **qwen2.5-coder:32b** | ~20 GB | Spitzenleistung im Open-Source-Bereich | `ollama run qwen2.5-coder:32b` |

> **Hinweis:** Modelle mit `:8b` oder `:7b` laufen auf 8–16 GB RAM/VRAM. Größere Varianten (`:14b`, `:32b`) benötigen 24+ GB.

---

## 7. Wichtige Betriebshinweise

### 7.1 Speicherplatz
Ollama speichert Modelle standardmäßig unter `/usr/share/ollama/.ollama` oder im Home-Verzeichnis des Installationsbenutzers. Große Modelle füllen die Partition schnell – vorab prüfen:
```bash
df -h
```

### 7.2 CPU vs. GPU
- **Ohne GPU:** Modelle laufen rein über CPU. Bei 7B/8B-Parametern noch akzeptabel, bei größeren Modellen sehr langsam.
- **Mit NVIDIA GPU:** NVIDIA-Treiber + `nvidia-container-toolkit` installieren, dann Docker-Container mit `--gpus all` starten.

### 7.3 Port-Konflikte bei `--net=host`
Da der Container das Host-Netzwerk direkt nutzt, darf kein anderer Dienst auf dem Server bereits Port 8080 belegen.

### 7.4 Proxmox-Snapshots
Bei Betrieb in einem Proxmox-LXC-Container empfiehlt sich ein Snapshot vor größeren Änderungen oder Modell-Downloads.

---

## 8. Schnell-Checkliste (Troubleshooting)

| Check | Befehl |
|-------|--------|
| Ollama lauscht global? | `sudo ss -tulpn \| grep 11434` → muss `*:11434` zeigen |
| Ollama antwortet? | `curl http://127.0.0.1:11434/api/tags` → JSON mit Modellen |
| Container läuft? | `sudo docker ps` → Status `Up` |
| Firewall offen? | `sudo ufw status` → Port 8080 erlaubt |
| Richtige URL? | `http://<SERVER-IP>:8080` (nicht 3000!) |

---

*Erstellt aus einer vollständigen Debug-Session – alle typischen Docker-Netzwerk-Fallstricke sind berücksichtigt.*
