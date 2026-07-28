
# Lokaler Mailserver mit Postfix + Dovecot {: #mailserver-main}

## 1. Projektziel

Ziel war die Einrichtung eines lokal erreichbaren Mailservers innerhalb des eigenen Labornetzwerks.

### Anforderungen

- Interne E-Mails zwischen Benutzern ermöglichen
- Über Thunderbird erreichbar sein
- Keine öffentliche Internet-Mailzustellung benötigen
- Ausschließlich im lokalen Netzwerk funktionieren

### Umgebung

| Komponente | Wert |
|------------|------|
| Betriebssystem | Ubuntu Server 26.04 |
| Servername | mailserver |
| Lokale Domain | example.local |
| Mailserver | Postfix |
| IMAP-Server | Dovecot |
| Mailclient | Thunderbird |

## 2. Installation der benötigten Pakete

sudo apt install postfix dovecot-imapd mailutils

Während der Installation wurde Postfix konfiguriert:

| Auswahl | Wert |
|---------|------|
| Konfigurationstyp | Internet Site |
| Mailname | example.local |

## 3. Erste Stolpersteine bei Postfix

### Problem: Falscher Mailname

Am Anfang wurde nur example eingetragen. Postfix meldete:

Warnung: mailname is not a fully qualified domain name

### Lösung

echo "example.local" | sudo tee /etc/mailname

## 4. Postfix Grundkonfiguration

Die wichtigen Werte in /etc/postfix/main.cf:

myhostname = mailserver.example.local
mydomain = example.local
myorigin = /etc/mailname

mydestination =
  $myhostname,
  localhost.$mydomain,
  localhost,
  $mydomain

mynetworks =
  xxx.xxx.xxx.xxx/8,
  xxx.xxx.xxx.xxx/24

Damit akzeptiert Postfix Mails aus dem lokalen Netzwerk.

## 5. Hostname-Anpassung

Anfangs war myhostname = mailserver gesetzt. Für eine saubere lokale Mailumgebung wurde daraus:

mailserver.example.local

### Kontrolle

postconf | grep -E "myhostname|mydomain|myorigin"

Ergebnis:

mydomain = example.local
myhostname = mailserver.example.local
myorigin = /etc/mailname

## 6. Lokaler Mailtest mit Postfix

Testmail senden:

echo "Hallo aus dem Mailserver" | mail -s "Test" benutzer

Kontrolle:

ls -la ~/Maildir/new

Die Mail wurde erfolgreich gespeichert.

## 7. Dovecot Einrichtung

Dovecot wurde installiert und gestartet:

sudo systemctl status dovecot

Ergebnis:

active (running)

## 8. Problem: Dovecot nutzte falsches Mailformat

Standardmäßig war Ubuntu auf mbox eingestellt:

mail_driver = mbox
mail_path = %{home}/mail
mail_inbox_path = /var/mail/%{user}

Wichtig: Postfix speicherte Mails aber in /home/benutzer/Maildir. Dovecot suchte deshalb am falschen Ort.

## 9. Umstellung auf Maildir

Datei bearbeiten:

sudo nano /etc/dovecot/conf.d/10-mail.conf

### Änderung

Alt:

mail_driver = mbox

Neu:

mail_driver = maildir
mail_home = /home/%{user | username}
mail_path = ~/Maildir

Zusätzlich wurde entfernt:

mail_inbox_path = /var/mail/%{user}

## 10. Kontrolle Maildir

Prüfung:

doveconf | grep mail_

Ergebnis:

mail_driver = maildir
mail_home = /home/%{user | username}
mail_path = ~/Maildir

## 11. Dovecot IMAP aktivieren

Kontrolle:

doveconf protocols

Ergebnis:

imap = yes

Ports prüfen:

ss -tulpn | grep -E ":143|:993"

Ergebnis:

0.0.0.0:993
0.0.0.0:143

IMAP SSL war somit erreichbar.

## 12. Authentifizierung testen

Test:

sudo doveadm auth test benutzer PASSWORT

Ergebnis:

passdb: benutzer auth succeeded

Damit funktioniert die Anmeldung.

## 13. Problem: Zertifikatsdatei falsch getestet

Versuch:

/etc/ssl/certs/ssl-cert-snakeoil.pem

führte zu:

Warnung: Permission denied

Hinweis: Die Datei ist kein Programm, sondern ein Zertifikat.

Richtig:

cat /etc/ssl/certs/ssl-cert-snakeoil.pem

oder nur von Diensten verwenden lassen.

## 14. Benutzer anlegen

Beispiel:

sudo adduser anna

Maildir erstellen:

sudo -u anna mkdir -p /home/anna/Maildir/{cur,new,tmp}

Berechtigungen:

sudo chown -R anna:anna /home/anna/Maildir

## 15. Thunderbird Einrichtung

### Eingang (IMAP)

| Feld | Wert |
|------|------|
| Server | xxx.xxx.xxx.xxx |
| Port | 993 |
| Verbindung | SSL/TLS |
| Authentifizierung | Passwort normal |
| Benutzer | benutzer oder anna |

### Ausgang (SMTP)

| Feld | Wert |
|------|------|
| Server | xxx.xxx.xxx.xxx |
| Port | 25 |
| Verbindung | Keine |
| Authentifizierung | Keine |

## 16. Problem: Falscher SMTP-Port

Fehler: Thunderbird wurde mit Port 993 für SMTP eingerichtet.

Problem: 993 = IMAP

SMTP benötigt: Port 25 oder später 587

## 17. Erfolgreicher Mailtest

Mail von Benutzer zu Anna:

echo "Hallo Anna" | mail -s "Test von Benutzer" anna@example.local

Log:

to=<anna@example.local>
status=sent
(delivered to maildir)

Bedeutung:
- Postfix nimmt Mail an
- Benutzer wird gefunden
- Mail wird gespeichert

## 18. Problem: Versand an externe Domain

Test:

anna@example.de

führte zu:

Warnung: Domain example.de does not accept mail (nullMX)

Hinweis: Der Server ist nur für example.local gedacht. Keine öffentliche Internet-Mailzustellung.

## 19. Aktueller Status

### Funktioniert:

[x] Postfix SMTP
[x] Dovecot IMAP
[x] Maildir Speicherung
[x] Lokale Benutzer
[x] Thunderbird Zugriff
[x] Interne Mailzustellung
[x] SSL IMAP Verbindung

## 20. Mögliche Erweiterungen

### SMTP Submission

| Eigenschaft | Wert |
|-------------|------|
| Port | 587 |
| Features | Login, Passwort, STARTTLS |

### Webmail

- Roundcube

### Sicherheit

- Rspamd
- ClamAV
- Fail2Ban

### Monitoring

- Wazuh Agent
- Mailserver Logs überwachen

## Fazit

Der lokale Mailserver wurde erfolgreich aufgebaut.

### Die größten Herausforderungen:

| Nr. | Problem | Lösung |
|-----|---------|--------|
| 1 | Falscher Mailname ohne Domain | example.local als FQDN setzen |
| 2 | Hostname-Konfiguration | mailserver.example.local verwenden |
| 3 | Dovecot nutzte standardmäßig mbox | Umstellung auf Maildir |
| 4 | Umstellung auf Maildir notwendig | mail_driver = maildir |
| 5 | Thunderbird SMTP/IMAP-Port-Verwechslung | SMTP = 25, IMAP = 993 |
| 6 | Externe Domains funktionieren absichtlich nicht | Nur example.local zulässig |

---

Ergebnis: Ein funktionierender interner Linux-Mailserver für ein Homelab bzw. kleines Firmennetzwerk.
