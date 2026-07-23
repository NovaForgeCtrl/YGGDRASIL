# Allgemeines über Proxmox VE (Virtual Environment)

Proxmox VE ist eine weltweit beliebte, kommerziell unterstützte Open-Source-Plattform für Enterprise-Virtualisierung. Sie vereint virtuelle Maschinen, Container, Storage, Software-defined Networking und Hochverfügbarkeits-Cluster in einer einzigen, webbasierten Benutzeroberfläche.



## 1. Was ist Proxmox VE? (Grundprinzip)
Proxmox VE basiert auf einem robusten **Debian GNU/Linux-Betriebssystem** und verwendet einen angepassten Linux-Kernel. Es ermöglicht die zentrale Verwaltung von virtualisierten Rechenzentren, egal ob auf einem einzelnen Heimserver oder in einem großflächigen Unternehmens-Cluster. 

* **Alles-in-Einem-Plattform:** Hypervisor, Firewall, Backup-Lösung, Cluster-Management und Netzwerkverwaltung sind direkt out-of-the-box integriert.
* **Web-Oberfläche:** Die komplette Administration erfolgt über einen modernen HTML5-Webbrowser – es ist keine separate Client-Software notwendig.

---

## 2. Kerntechnologien
* **KVM (Kernel-based Virtualization):** Ermöglicht die vollständige Virtualisierung (VMs) für Windows, Linux, BSD und andere Betriebssysteme mit eigener Hardware-Emulation.
* **LXC (Linux Containers):** Ermöglicht leichtgewichtige Betriebssystem-Virtualisierung (Container), bei der sich mehrere Linux-Instanzen den Kernel des Host-Systems teilen (ähnlich wie Docker, aber als vollwertige System-Container konzipiert).
* **Proxmox Cluster (PVE Cluster):** Mehrere Proxmox-Knoten können zentral über einen Master verwaltet werden. Dank integrierter Hochverfügbarkeit (HA) ziehen VMs bei einem Hardware-Ausfall automatisch auf einen anderen Knoten um.
* **Storage-Flexibilität:** Unterstützung für lokale Festplatten (ZFS, LVM, Directory) sowie Netzwerkspeicher (NFS, SMB/CIFS, iSCSI, Ceph, GlusterFS).
* **Proxmox Backup Server (PBS):** Eine perfekt integrierte Backup-Lösung für inkrementelle, deduplizierte und verschlüsselte Sicherungen.

---

## 3. Vorteile von Proxmox VE
* **100% Open Source & Kostenlos nutzbar:** Die Community-Version ist komplett kostenlos und unbeschränkt verfügbar. Es gibt optionale, kostenpflichtige Enterprise-Subskriptionen für stabilere Updates und professionellen Support.
* **ZFS-Dateisystem-Integration:** Bietet integrierten Schutz vor Datenverlust, Snapshots in Echtzeit und hochentwickeltes Software-RAID direkt im Installer.
* **Keine Lizenzkosten pro CPU:** Im Gegensatz zu kommerziellen Mitbewerbern (wie VMware ESXi) gibt es keine restriktiven Lizenzmodelle basierend auf Sockeln oder RAM-Mengen.
* **Vollwertige API:** Dank der REST-API lässt sich Proxmox hervorragend in Automatisierungsskripte, Terraform oder Monitoring-Tools (wie Prometheus/Grafana) einbinden.

---

## 4. Nachteile & Herausforderungen
* **Lernkurve für Einsteiger:** Die Vielfalt an Netzwerkoptionen (Linux Bridges, OVS, SDN), Storage-Backend-Typen und Cluster-Konfigurationen kann Neulinge anfangs überfordern.
* **Enterprise-Features erfordern Planung:** Wer ZFS, Ceph oder Hochverfügbarkeit produktiv nutzen möchte, benötigt performante Hardware (ECC-RAM, schnelle SSDs, dedizierte Netzwerke).
* **Kein nativer Desktop-Client:** Die Administration läuft ausschließlich über das Web-Interface oder die Konsole (SSH/CLI).

---

## 5. Proxmox VE vs. Andere Hypervisoren

| Eigenschaft | Proxmox VE | VMware ESXi / vSphere |
| :--- | :--- | :--- |
| **Lizenzmodell** | Open Source (Kostenlose Community / Bezahl-Support) | Kommerziell (Sehr teuer, oft lizenzbasiert) |
| **Basis-OS** | Debian Linux | Proprietäres ESXi-Betriebssystem |
| **Container-Support** | Nativ (LXC) & KVM | Über Zusatzprodukte / Kubernetes |
| **Dateisysteme** | ZFS, Ext4, XFS, LVM u.v.m. | VMFS, vSAN |

---

## 6. Fazit
Proxmox VE hat sich als eine der mächtigsten und flexibelsten Virtualisierungslösungen am Markt etabliert. Sie schließt die Lücke zwischen teuren Enterprise-Lösungen (wie VMware) und reinen DIY-Systemen. Ob für den ambitionierten Homelab-Betreiber, KMUs oder große Rechenzentren – Proxmox bietet maximale Kontrolle, hohe Stabilität und volle Datensouveränität.