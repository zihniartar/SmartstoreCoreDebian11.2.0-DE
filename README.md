# Installieren von Smartstore Core auf Debian 11.2.0

## Voraussetzungen

 - Smartstore Core für Linux X64
 - Debian 11
 - Nicht root Benutzer mit sudo-Rechten

## Debian Server vorbereiten

 - NGINX Reverse Proxy installieren
 - FTP-Server installieren
 - Firewall anpassen

## NGINX installieren

 ### Das System aktualisieren und benötigte Pakete installieren:

   ```bash
   sudo apt update -y
   ```
   ```bash
   sudo apt upgrade -y
   ```
   ```bash
   sudo apt install curl gnupg2 ca-certificates lsb-release
   ```


 ### NGINX installieren
   ```bash
   sudo apt install nginx -y
   ```

 ### NGINX starten, stoppen, neu starten und Konfiguration neu laden:
   ```bash
sudo systemctl start nginx
```
 ```bash
sudo systemctl stop nginx
  ```
```bash
sudo systemctl restart nginx
 ```
 ```bash
sudo systemctl reload nginx
   ```

 ### Die installierte Version von NGINX prüfen:
   ```bash
	sudo nginx -v
   ```
 ### Die NGINX-Konfiguration auf Fehler prüfen:
 ```bash
	sudo nginx -t
  ```

### NGINX als Dienst beim Systemstart starten:
   ```bash
	sudo systemctl enable nginx
  ```

## FTP-Server installieren und konfigurieren
### Installation
   ```bash
sudo apt update
sudo apt install proftpd-basic
  ```

### Konfiguration
Die Konfigurationsdateien befinden sich im folgeden Ordner:
   ```bash
/etc/proftpd/
  ```
Eigene Konfigurationsdateien sollten am besten im Verzeichnis
   ```bash
conf.d
  ```
  abgelegt werden.
  
Wir erstellen mit diesem Befehl eine neue Konfigurationsdatei:
   ```bash
sudo vi /etc/proftpd/conf.d/custom.confcustom
  ```
Und fügen folgende Zeilen ein:
   ```bash
 # Ftp user doesn't need a valid shell
<Global>
    RequireValidShell off
</Global>
 # If desired turn off IPv6
UseIPv6 off
 # Default directory is ftpusers home
DefaultRoot ~ ftpuser
 # Limit login to the ftpuser group
<Limit LOGIN>
    DenyGroup !ftpuser
</Limit>
  ```

Anschließend wird die Datei gespeichert und der ProFTPD Server neu gestartet:
   ```bash
sudo systemctl restart proftpd.service
  ```

### FTP Benutzer erstellen
Für den FTP Zugriff wird ein eigener BEnutzer ohne gültiger Login-Shell und mit dem Homeverzeichnis 
   ```bash
/var/www/upload
  ```
  erstellt.
   ```bash
sudo adduser ftpuser --shell /bin/false --home /var/www/upload
  ```
  

## Firewall regeln anpassen

 ### Liste der bereits eingerichteten Anwendungsprofile ausgeben:

   ```bash
   sudo ufw app list
   ```
> **Hinweis:** Wird der Befehl mit 
> ```bash
>   sudo: ufw: Befehl nicht gefunden
 >  ```
 > quittiert, dann ist keine Firewall installiert und dieser Punkt kann erst einmal übersprungen werden.

 ### In der ausgebenen Liste sind drei NGINX-Profile vorhanden:
 
- **Nginx Full**: Dieses Profil öffnet Port 80 und 443 für NGINX
- **Nginx HTTP**: Dieses Profil öffnet nur Port 80 für NGINX
- **Nginx HTTPS**: Dieses Profil öffnet nur Port 443 für NGINX
 
### Port 80 und Port 443 für NGINX zulassen:
   ```bash
   sudo ufw allow 'Nginx FULL'
   ```
   
### Das Ergebnis prüfen:
   ```bash
   sudo ufw status
   ```
 
 ## NGINX einrichten
 ### Standard NGINX-Seite aufrufen
 - IP-Adresse herausfinden, wenn unbekannt
    ```bash
   hostname -I
   ```
- Im Browser NGINX-Startseite per IP aufrufen
	 ```bash
	http://ip-adresse
	```
- Es wird die Standard-Landingpage für NGINX angezeigt

	![NGINX-Landingspage](https://www.smartstore.com/news/images/Qs7PlUtvga.png)

### NGINX als Reverse-Proxy konfigurieren
- Folgende Datei mit einem Editor öffnen und den Inhalt durch den Codeausschnitt ersetzen:
	 ```bash
	/etc/nginx/sites-available/default
	```

	 ```bash
	server {
    listen        80;
    server_name   example.com *.example.com;
    location / {
        proxy_pass         http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
	```
	Wenn noch keine Domain vorhanden ist, kann auch eine IP-Adresse eingetragen werden:
	```bash
	server_name 172.16.1.17;
	```
	Hiernach muss die NGINX-Konfiguration neu geladen werden:
	```bash
	sudo systemctl reload nginx
	```

## Dateien übertragen und starten
### Dateien übertragen
Die Dateien aus dem Release per FTP auf den Debian-Server in den Ordner
   ```bash
	/var/www/html
``` 
übertragen.
> **Hinweis:** Bei unserem Beispiel FTP-Benutzer werden die Dateien per FTP nach 
> ```bash
>   /var/www/upload
 >  ```
 > übertragen und müssen von da verschoben werden.
      	
### App starten


 
 



