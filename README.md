
# Installieren von Smartstore Core auf Debian 11.2.0

## Voraussetzungen

 - Smartstore Core für Linux X64
 - Debian 11
 - Nicht root Benutzer mit sudo-Rechten

## Debian Server vorbereiten

 - NGINX Reverse Proxy installieren
 - FTP-Server installieren
 - Firewall anpassen
 - MySQL installieren
 - Smartstore installieren

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


## MySQL installieren

 ### Das Paket ```mysql-server``` installieren:

MySQL Repository konfigurieren:
Lokalen Paket-Index aktualisieren:
   ```bash
   sudo apt update
   ```
   ```gnupg``` installieren:
   ```bash
sudo apt install gnupg
   ```
Als nächstes das MySQL ```.deb``` Paket mit ```wget``` herunterladen und installieren:
Die [MySQL Downloadseite](https://dev.mysql.com/downloads/repo/apt/) im Browser öffnen und unten rechts den **Download** Button drücken und. Die Aufforderung zum Login bzw. Anmelden ignorieren und mit der rechten Maustaste die Adresse des Links "**Nein Danke...**" kopieren.

Die Datei auf den Server herunterladen (statt der URL zur ```.deb```-Datei, die oben kopierte URL einfügen):
   ```bash
   cd /tmp
   wget https://dev.mysql.com/get/mysql-apt-config_0.8.22-1_all.deb
   ```

Die Datei wurde in das aktuelle Verzeichnis heruntergeladen und kann installiert werden:
   ```bash
   sudo dpkg -i mysql-apt-config*
   ```
```dpkg``` wird zum Installieren, Entfernen und Überprüfen von ```.deb```-Softwarepaketen verwendet. Der Schalter ```-i``` zeigt an, dass die angegebene Datei installiert werden soll.
Während der Installation wird ein Konfigurationsbildschirm angezeigt, im dem die gewünschte Version von MySQL angegeben werden kann.

Paket-Cache aktualisieren:
   ```bash
   sudo apt update
   ```

**MySQL-Installation** aufrufen:
   ```bash
   sudo apt install mysql-server
   ```
   Sicherheitsskript ausführen:
   ```bash
   sudo mysql_secure_installation
   ```
>Die Einstellungen hier können je nach individuellen Sicherheitsanforderungen variieren.


   Um die Benutzerauthentifizierung- und Berechtigungen anzupassen MySQL-Eingabeaufforderung öffnen:
   ```bash
   mysql -u root -p
   ```
Authentifizierungsverfahren prüfen:
```bash
   SELECT user,authentication_string,plugin,host FROM mysql.user;
   ```
Wird der ```root``` Benutzer über das ```auth-socket```-Plugin authentifiziert, muss das ```root```-Konto umkonfiguriert werden. Mit diesem Befehl wird das vorherige ```root```-Passwort geändert. Es sollte ein starkes Passwort gewählt werden (```password``` ersetzen durch eigenes).
```bash
   ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'password';
   ```
Berechtigungstabellen neu laden:
```bash
   FLUSH PRIVILEGES;
   ```
MySQL-Shell verlassen:
```bash
   exit
   ```
Einen dedizierten MySQL-Benutzer für die Nutzung mit Smartstore erstellen:
```bash
   mysql -u root -p
   ```
```bash
   CREATE USER 'smartstore'@'localhost' IDENTIFIED BY 'password';
   ```
> ```smartstore``` und ```password``` nach belieben ändern

Benutzerberechtigungen erteilen:
```bash
   GRANT ALL PRIVILEGES ON *.* TO 'smartstore'@'localhost' WITH GRANT OPTION;
   ```
MySQL-Shell verlassen:
```bash
   exit
   ```

## Smartstore installieren
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
      	
### App als Dienst einrichten
Erstellen einer Dienstdefinitionsdatei für ```systemd```:
```bash
sudo nano /etc/systemd/system/kestrel-smartstore.service
``` 
Folgenden Code-Ausschnitt einfügen und speichern:
> Bitte die Hinweise unter dem Codeblock beachten!
```bash
[Unit]
Description=Smartstore Core Web App running on Linux

[Service]
WorkingDirectory=/var/www/html
ExecStart=/usr/bin/dotnet /var/www/html/Smartstore.Web.dll
Restart=always
#Restart service after 10 seconds if dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=smartstore-core
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
``` 
> **Hinweis**: Pfade in ```WorkingDirectory``` und ```ExecStart``` ggf. anpassen.

> **Wichtig**: 
> Code bei **frameworkabhängiger Bereitstellung**:
>```bash
>ExecStart=/usr/bin/dotnet /var/www/html/Smartstore.Web.dll
>```
> Code bei **eigenständiger Bereitstellung**:
>```bash
>ExecStart=/var/www/html/Smartstore.Web
>```

### Dienst aktivieren und starten
Dienst aktivieren:
```bash
sudo systemctl enable kestrel-smartstore.service
``` 
Dienst starten:
```bash
sudo systemctl start kestrel-smartstore.service
``` 
Dienst soppen:
```bash
sudo systemctl stop kestrel-smartstore.service
```

### ```wkhtmltopdf``` installieren
```bash
sudo apt-get update
```
```bash
sudo apt-get -y install wkhtmltopdf
```

### Festlegen der Ordnerberechtigungen
Eigenen Benutzer als Besitzer des Website-Ordners mit vollen Lese-, Schreib- und Ausführungsrechten setzen:

> **Hinweis:** ```smartstore``` ist beispielhaft der eigene Benutzer
```bash
chown -R smartstore /var/www/html/
```
Webserver als Gruppen-Besitzer setzen:
```bash
chgrp -R www-data /var/www/html/
```
Rekursiv für alle Dateien und Ordner Lese-, Schreib- und Ausführungsrechte für den Besitzer, Lese- und Ausführungsrechte für den Gruppen-Besitzer und keine Rechte für andere:

```bash
chmod -R 750 /var/www/html/
```



### Smartstore installieren
Die Website per IP-Adresse oder Domainname aufrufen und die erforderlichen Daten eingeben.

![Startseite der Installation](https://www.smartstore.com/news/images/smartstore_installation_de_640px.png)

> Der MySQL-Server ist per localhost erreichbar. Als Anmeldename für die Datenbank ist der für diese Installation eigens angelegte MySQL-Benutzer zu verwenden
to be continued...


 
 



