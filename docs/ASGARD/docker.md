# Allgemeines über Docker

Docker ist eine weltweit führende Open-Source-Plattform zur Automatisierung der Bereitstellung, Skalierung und Verwaltung von Anwendungen mithilfe von Containern. 



## 1. Was ist Docker? (Grundprinzip)
Docker ermöglicht es, eine Anwendung mitsamt ihrer gesamten Laufzeitumgebung (Bibliotheken, Abhängigkeiten, Konfigurationen) in ein einziges, in sich geschlossenes Paket zu packen – einen sogenannten **Container**. 

* **Isolierung:** Container laufen isoliert voneinander auf demselben Betriebssystemkern, teilen sich aber die Ressourcen des Host-Systems.
* **Portabilität („Build once, run anywhere“):** Ein einmal erstellter Docker-Container läuft auf jedem System, das Docker unterstützt – egal ob auf dem Entwickler-Laptop, einem Testserver oder in der Cloud (AWS, Azure, Google Cloud).

---

## 2. Kernkomponenten
* **Dockerfile:** Eine Textdatei mit Befehlen, die Schritt für Schritt beschreiben, wie ein Docker-Image erstellt wird.
* **Docker Image:** Ein unveränderliches (read-only) Template, das als Bauplan für die Container dient.
* **Docker Container:** Die laufende, instanziierte Version eines Docker-Images. Hier läuft die eigentliche Anwendung.
* **Docker Hub / Registry:** Ein zentraler Speicherort für Docker-Images (vergleichbar mit GitHub für Quellcode), von dem aus Images heruntergeladen oder geteilt werden können.
* **Docker Compose:** Ein Tool, um Multi-Container-Anwendungen (z. B. eine Web-App plus Datenbank) mithilfe einer einzigen YAML-Konfigurationsdatei zu definieren und zu starten.

---

## 3. Vorteile von Docker
* **Geringer Ressourcenverbrauch:** Da Container keinen eigenen Betriebssystemkern (Guest-OS) benötigen, sind sie extrem leichtgewichtig und starten in Sekundenschnelle.
* **Konsistenz in allen Umgebungen:** Das berüchtigte Problem „*Bei mir auf dem Laptop hat es aber funktioniert!*“ gehört der Vergangenheit an, da Entwicklungs-, Test- und Produktionsumgebung absolut identisch sind.
* **Skalierbarkeit & Microservices:** Anwendungen lassen sich problemlos in viele kleine, unabhängige Dienste (Microservices) aufteilen, die einzeln skaliert oder aktualisiert werden können.
* **Einfache Versionskontrolle:** Docker-Images können versioniert werden. Ein schnelles Rollback auf eine vorherige Version ist bei Problemen problemlos möglich.

---

## 4. Nachteile & Herausforderungen
* **Lernkurve:** Das Verständnis von Containern, Netzwerken (Volumes, Bridges) und Orchestrierung erfordert Einarbeitungszeit.
* **Sicherheitsrisiken (Host-Abhängigkeit):** Da sich alle Container den Kernel des Host-Systems teilen, kann eine Sicherheitslücke im Kernel unter Umständen das gesamte System gefährden (schlechtere Isolation als bei klassischen VMs).
* **Datenverwaltung (Persistenz):** Standardmäßig sind Container zustandslos (stateless). Daten, die dauerhaft gespeichert werden müssen (z. B. Datenbanken), erfordern den Einsatz von sogenannten *Volumes* oder *Bind Mounts*.
* **Eingeschränkte GUI-Unterstützung:** Docker ist primär für kommandozeilenbasierte Server-Anwendungen optimiert; grafische Benutzeroberflächen (Desktop-Apps) erfordern zusätzlichen Konfigurationsaufwand.

---

## 5. Docker vs. Klassische Virtualisierung (VMs)

| Eigenschaft | Virtuelle Maschinen (VMs) | Docker-Container |
| :--- | :--- | :--- |
| **Architektur** | Jede VM hat ein komplettes Gast-Betriebssystem | Teilen sich den Betriebssystemkern des Hosts |
| **Größe** | Meist mehrere Gigabyte (GB) | Meist nur einige Megabyte (MB) |
| **Startzeit** | Minuten | Bruchteile von Sekunden |
| **Ressourceneffizienz** | Geringer (hoher Overhead) | Sehr hoch |

---

## 6. Fazit
Docker hat die Softwareentwicklung und den Systembetrieb revolutioniert. Es sorgt für saubere, reproduzierbare Umgebungen und ist aus der modernen Cloud-Infrastruktur, dem DevOps-Bereich sowie dem Betrieb von Heimservern (z. B. mit Portainer, Plex oder Home Assistant) kaum noch wegzudenken.