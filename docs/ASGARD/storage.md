# Allgemeines über NAS-Systeme als Backup {#nas-storage}

!!! info "💡 Schnellüberblick"
    Ein **NAS (Network Attached Storage)** ist ein dediziertes Datenspeichergerät, das an ein lokales Netzwerk angeschlossen ist und autorisierten Benutzern sowie Clients zentralen Speicherplatz zur Verfügung stellt. Im modernen IT- und Heimserver-Umfeld gehört ein NAS zu den beliebtesten und zuverlässigsten Lösungen für **Datensicherung (Backup).**

---

## 1. Was ist ein NAS? (Grundprinzip)

Im Gegensatz zu einer externen Festplatte, die direkt per USB an einen Computer angeschlossen wird, ist ein NAS ein **eigenständiger Mini-Computer** mit eigenem Betriebssystem *(z. B. Synology DSM, TrueNAS, QNAP QTS)* und Netzwerkschnittstelle.

!!! tip "🎯 Das Besondere"
    * **Zentraler Speicher:** Mehrere Geräte im Heimnetzwerk *(PCs, Laptops, Smartphones, VMs)* können gleichzeitig auf das NAS zugreifen.
    * **RAID-Sicherheit:** Die meisten NAS-Systeme arbeiten mit mehreren Festplatten im RAID-Verbund, wodurch beim Ausfall einer Festplatte die Daten meist geschützt bleiben.

---

## 2. Warum ein NAS als Backup? (Vorteile)

!!! success "✅ Warum ein NAS?"

    * **Automatisierte Sicherungen:** Backups können vollautomatisch im Hintergrund laufen *(z. B. nachts via Time Machine, Windows-Sicherung oder spezialisierter Software wie BorgBackup, UrBackup).*
    * **Zentralisierung:** Alle Daten von verschiedenen Endgeräten landen an einem einzigen, organisierten Ort.
    * **Erweiterte Backup-Features:** Moderne NAS-Betriebssysteme bieten integrierte Funktionen wie **Snapshots** *(Schutz vor Ransomware/Verschlüsselungstrojanern)* und Versionsverwaltung.
    * **Netzwerkintegration:** Dank Protokollen wie **SMB, NFS, AFP** oder **iSCSI** ist die Anbindung an fast jedes Betriebssystem problemlos möglich.

---

## 3. Nachteile & Risiken von NAS-Backups

!!! warning "⚠️ Was man wissen sollte"

    * **Kein vollständiger Schutz vor Desasterszenarien:** Wenn das NAS dauerhaft eingeschaltet im lokalen Netzwerk steht, sind die Backup-Daten bei einem echten Brand, Wasserschaden, Blitzeinschlag oder einer professionellen Ransomware-Attacke *(die netzwerkweite Freigaben verschlüsselt)* ebenfalls gefährdet.
    * **Anschaffungskosten & Wartung:** Ein gutes NAS mit mehreren Festplatten *(plus USV für Stromabsicherung)* ist in der Anschaffung teurer als eine einfache USB-Festplatte. Zudem muss sich der Nutzer um Updates und RAID-Überwachung kümmern.
    * **Geschwindigkeitsabhängigkeit:** Die Backup-Geschwindigkeit hängt stark von der Netzwerkanbindung ab *(Gigabit-LAN limitiert oft; 2.5GbE oder 10GbE bzw. Glasfaser sind für große Datenmengen ratsam).*

---

## 4. Das 3-2-1-Backup-Prinzip mit NAS

!!! abstract "📋 Das Goldene Regelwerk"
    Ein NAS ist ein hervorragender Baustein für das bewährte **3-2-1-Backup-Konzept:**

    | Regel | Bedeutung |
    | :--- | :--- |
    | **3** | 🗂️ Drei Kopien deiner Daten insgesamt *(1 Original + 2 Backups)* |
    | **2** | 💾 Zwei verschiedene Medientypen *(z. B. Computer + NAS)* |
    | **1** | 🌐 Eine externe Kopie — idealerweise an einem anderen Ort oder offline *(z. B. ein Offsite-NAS bei Freunden, Cloud-Backup oder eine manuell gewechselte externe USB-Festplatte, die nach dem Backup abgezogen wird)* |

---

## 5. Fazit

!!! quote "🧭 Das Fazit"
    Ein NAS-System ist die ideale Lösung für **strukturierte, automatisierte** und netzwerkweite Backups im Heimnetzwerk oder in kleinen Unternehmen. Es bietet hohen Komfort und Flexibilität, sollte für eine lückenlose Sicherheitsstrategie jedoch mit **Offsite-Backups** oder einer **Offline-Kopie** ergänzt werden.

!!! tip "🚀 Tipp für Einsteiger"
    Starte mit einem **2-Bay NAS** im **RAID 1** und einer günstigen USV. Das ist der beste Einstieg in die Welt der professionellen Datensicherung — ohne dein Budget zu sprengen!
