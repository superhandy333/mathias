
# Datenbankserver <!-- omit in toc -->

Anleitung zum Aufsetzen des Datenbankservers für estw

- [Installation und Einrichtung](#installation-und-einrichtung)
  - [Begriffe](#begriffe)
  - [Aufsetzen des Datenbank-Containers als Docker](#aufsetzen-des-datenbank-containers-als-docker)
    - [Parameter](#parameter)
    - [Docker-Befehl](#docker-befehl)
  - [Anlegen der Datenstruktur](#anlegen-der-datenstruktur)
  - [Anlegen des DB-Users](#anlegen-des-db-users)
  - [Vergabe Schreibberechtigung des Export-Verzeichnisses](#vergabe-schreibberechtigung-des-export-verzeichnisses)
- [Datenexport](#datenexport)
  - [Ausleitung nach csv](#ausleitung-nach-csv)
  - [Transfer der Dateien nach Windows](#transfer-der-dateien-nach-windows)
- [Autor](#autor)


## Installation und Einrichtung

### Begriffe

<!-- |Begriff | Beschreibung| -->

|||
|-|-|
|Windows|Windows-Rechner auf dem das System `estw` läuft und auf dem die csv-Dateien für den Zugriff von `estw` abgelegt werden.|
|Host|Linux-Server auf ein Docker-Dienst läuft. Der Platzhalter 192.168.xxx.xxx innerhalb dieser Dokumentation ist die Adresse des Hostes.|
  |Docker|Docker-Container auf dem Host mit MariaDB-Datenbank. Im Beispiel heißt der Container `franzidb`.|

### Aufsetzen des Datenbank-Containers als Docker

Der Docker-Container wird mit dem Befehl `docker run` erstellt und enthält eine Instanz von `mariadb`.
Eingerichtet werden eine Portweiterleitung und ein Sharing-Verzeichnis zum Zugriff auf die abgelegten csv-Dateien vom Host aus.

#### Parameter

|||
|-|-|
|-p 3300:3306|Portweiterleitung `3300:3360` vom Container (3306) zum Host (3300). Auf die Datenbank kann vom Host auf Port 3300 zugegriffen werden. Die Änderung des Standardports ist notwendig, da auf dem  Host selber eine andere Instanz einer MySQL-Datenbank läuft.|
|-v ~/docker/export/:/home/hostfiles/export|Das Verzeichnis `~/docker/export/` vom Host-Rechner wird in den Container als Verzeichnis `/home/hostfiles/export` eingebunden. Die csv-Dateien werden vom Datenbank-Server hier abgelegt und können vom Host in dessen Verzeichnis `~/docker/export/` abgeholt werden.|
|-e MARIADB_ROOT_PASSWORD=xyz|Legt das Root-Passwort der Datenbank fest.|
|-d|Der Container wird detached. Das bedeuted, dass der Prompt nach dem Start wieder freigegeben wird. Die Datenbank läuft so lange weiter, bis der Container wieder gestoppt wird.|
|--name franzidb|Name des Docker-Containers.|

#### Docker-Befehl

~~~batch
docker run -it -p 3300:3306 -v ~/docker/export/:/home/hostfiles/export --name franzidb -e MARIADB_ROOT_PASSWORD=xyz -d mariadb
~~~

### Anlegen der Datenstruktur

Verbinde dich von Windows aus mit dem DB-Server als `root` und führe das Skript in der Datei [create_database.sql](https://github.com/superhandy333/csharp/blob/6b0e968f4034b0f4640607b1c1efba44735616f8/CPP/estw/sql/create_database.sql) aus. Du findest die Datei im Projektverzeichnis des Repositories unter 
`sql`.

Benutze dafür ein SQL-Programm deiner Wahl, z.B. `Heidi`. Alternativ kannst du das Script auch mit dem Terminalprogramm `mysqlsh` starten.

~~~batch
mysqlsh --uri root@192.168.xxx.xxx:3300 --sql -f create_database.sql
~~~

### Anlegen des DB-Users

Führe das Skript in der Datei [create_user.sql](https://github.com/superhandy333/csharp/blob/6b0e968f4034b0f4640607b1c1efba44735616f8/CPP/estw/sql/create_user.sql) aus.

~~~batch
mysqlsh --uri root@192.168.xxx.xxx:3300 --sql -f create_user.sql
~~~

### Vergabe Schreibberechtigung des Export-Verzeichnisses

Wechsle auf die Kommandozeile des Docker-Containers der Datenbank.

~~~batch
docker exec -it franzidb /bin/bash
~~~

Navigiere zum Verzeichnis `/home/hostfiles`. Vergebe dort für das Export-Verzeichnis `export` alle verfügbaren Berechtigungen.

~~~batch
chmod 777 export
~~~

## Datenexport

### Ausleitung nach csv

Speichern der Daten in csv-Dateien. Führe dazu das Skript in der Datei [export.sql](https://github.com/superhandy333/csharp/blob/6b0e968f4034b0f4640607b1c1efba44735616f8/CPP/estw/sql/export.sql) aus.

~~~batch
mysqlsh --uri root@192.168.xxx.xxx:3300 --sql -f export.sql
~~~

### Transfer der Dateien nach Windows

Kopieren der csv-Dateien vom export-Verzeichnis des Hostes auf den Windows-Rechner. Führe dazu das Skript in der Datei [transfer.sql](https://github.com/superhandy333/csharp/blob/6b0e968f4034b0f4640607b1c1efba44735616f8/CPP/estw/sql/transfer.bat) aus.

~~~batch
mysqlsh --uri root@192.168.xxx.xxx:3300 --sql -f transfer.sql
~~~

## Autor

Mathias Rentsch<br>
rentsch@online.de<br>
Juli 2025
