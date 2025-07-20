
# ESTW 2 <!-- omit in toc -->

Anleitung zum Aufsetzen eines Datenbankservers zur Entwicklung von `estw2`.

- [Aufsetzen des Docker-Containers](#aufsetzen-des-docker-containers)
  - [Parameter](#parameter)
  - [Docker-Befehl](#docker-befehl)
- [Anlegen des DB-Users](#anlegen-des-db-users)
- [Anlegen der Datenstruktur](#anlegen-der-datenstruktur)
- [Skript zum Generieren der csv-Dateien](#skript-zum-generieren-der-csv-dateien)
- [Skript zum Transfer der csv-Dateien](#skript-zum-transfer-der-csv-dateien)
- [Autor](#autor)


## Aufsetzen des Docker-Containers

Der Docker-Container wird mit dem Befehl `docker run` erstellt und enthält eine Instanz von `mariadb`.
Eingerichtet werden eine Portweiterleitung und ein Sharing-Verzeichnis zum Zugriff der abgelegten csv-Dateien vom Host aus.

### Parameter

|Parameter|Erläuterung|
|-|-|
|-p 3300:3306|Portweiterleitung `3300:3360` vom Container (3306) zum Host (3300). Auf die Datenbank kann vom Host auf Port 3300 zugegriffen werden. Änderung des Standardports, weil auf dem  Host selber eine Instanz von MySQL-Datenbank läuft.|
|-v ~/docker/export/:/home/hostfiles|Das Verzeichnis `~/docker/export/` vom Host-Rechner wird in den Container als Verzeichnis `/home/hostfiles` eingebunden. Die csv-Dateien werden vom Datenbank-Server hier abgelegt und können vom Host im Verzeichnis `~/docker/export/` abgeholt werden.|
|-e MARIADB_ROOT_PASSWORD=xyz|Legt das Root-Passwort der Datenbank fest.|
|-d|Der Container wird detached. Das bedeuted, dass der Prompt nach dem Start wieder freigegeben wird. Die Datenbank läuft so lange weiter, bis der Container wieder gestoppt wird.|

### Docker-Befehl

~~~batch
docker run -it -p 3306:3306 -v ~/docker/export/:/home/hostfiles --name franzidb -e MARIADB_ROOT_PASSWORD=xyz -d mariadb
~~~

## Anlegen des DB-Users

## Anlegen der Datenstruktur

## Skript zum Generieren der csv-Dateien

## Skript zum Transfer der csv-Dateien

## Autor

Mathias Rentsch<br>
rentsch@online.de
