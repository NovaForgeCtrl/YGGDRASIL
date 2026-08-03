# ⚔️ ASGARD — Die Fundamente {: #asgard-main }

> *Die heilige Halle der Kernsysteme. Hier residieren die Schmieden, die allen anderen Welten Stabilität verleihen.*

---

## 📋 Systeminformationen

| Attribut | Wert |
|----------|------|
| **Status** | 🟢 Aktiv |
| **Erstellt** | 2026 |
| **Zuletzt geändert** | 2026-07 |

---

## 🎯 Ziel / Zweck

ASGARD bildet das Fundament von YGGDRASIL — die physische und virtuelle Infrastruktur, auf der alle anderen Welten aufbauen.

---

## 🛠️ Komponenten

### Proxmox VE
Virtualisierungsplattform für alle VMs und Container.

[Proxmox VE](proxmox){: .world-link }

### Ubuntu Server
Betriebssystembasis für alle Dienste.

[Ubuntu Server](ubuntu-server){: .world-link }

### Docker
Container-Orchestrierung für isolierte Dienste.

[Docker](docker){: .world-link }

### Storage
ZFS-basierte Speicherlösung für VMs, Backups und Daten.

[Storage](storage){: .world-link }

---

## 📊 Status / Monitoring

| Metrik | Wert | Letzte Prüfung |
|--------|------|----------------|
| CPU-Auslastung | 15% | 2026-07-23 |
| RAM-Nutzung | 32GB / 64GB | 2026-07-23 |
| Storage | 2.5TB / 4TB | 2026-07-23 |

---

## 📝 Notizen

!!! note "Wichtig"
    - Proxmox-Cluster auf 3 Nodes erweitert
    - ZFS-Pool mit RAIDZ1 konfiguriert
    - Backup-Script läuft täglich um 02:00 Uhr

---

> *"Von festem Grund aus erhebt sich jeder mächtige Bau."*
