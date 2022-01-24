# Installieren von Smartstore Core auf Debian 11.2.0

## Voraussetzungen

 - Smartstore Core für Linux X64
 - Debian 11
 - Nicht root Benutzer mit sudo-Rechten

## Debian Server vorbereiten

 - NGINX Reverse Proxy installieren
 - todo
 - todo

## NGINX installieren

 - Das System aktualisieren und benötigte Pakete installieren:

   ```bash
   sudo apt update -y
   ```
      ```bash
   sudo apt upgrade -y
   ```
      ```bash
   sudo apt install curl gnupg2 ca-certificates lsb-release
   ```


 - NGINX installieren
      ```bash
   sudo apt install nginx -y
   ```

 - NGINX starten, stoppen, neu starten und Konfiguration neu laden:
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
 - Die installierte Version von NGINX prüfen:
      ```bash
	sudo nginx -v
	  ```
 - Die NGINX-Konfiguration auf Fehler prüfen:
      ```bash
	sudo nginx -t
	  ```
- NGINX als Dienst beim Systemstart starten:
     ```bash
	sudo systemctl enable nginx
  ```

## Firewall regeln anpassen

 - Liste der bereits eingerichteten Anwendungsprofile ausgeben:

   ```bash
   sudo ufw app list
   ```
> **Hinweis:** Wird der Befehl mit 
> ```bash
>   sudo: ufw: Befehl nicht gefunden
 >  ```
 > quittiert, dann ist keine Firewall installiert und dieser Punkt kann erst einmal übersprungen werden.
 - In der ausgebenen Liste sind drei NGINX-Profilevorhanden

 - **Nginx Full**: Dieses Prodil öffnet Port 80 und 443 für NGINX
 - **Nginx HTTP**: Dieses Profil öffnet nur Port 80 für NGINX
 - **Nginx HTTPS**: Dieses Profil öffnet nur Port 443 für NGINX
 
- Port 80 und Port 443 für NGINX zulassen:
   ```bash
   sudo ufw allow 'Nginx FULL'
   ```
-
 


 
 



