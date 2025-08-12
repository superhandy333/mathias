
# ESTW 2 <!-- omit in toc -->

Privates Stellwerkssystem `estw2` zum Steuern von Modelleisenbahnen.

- [Projektierung](#projektierung)
  - [Projekte](#projekte)
  - [Elemente](#elemente)
  - [Files](#files)
  - [create\_database.sql](#create_databasesql)
      - [create\_user.sql](#create_usersql)
      - [export.sql](#exportsql)
      - [transfer.sql](#transfersql)
- [Autor](#autor)

## Datenbankserver

Anleitung zum Aufsetzen eines Datenbankservers

### Installation und Einrichtung

#### Begriffee

<!-- |Begriff | Beschreibung| -->

|||
|-|-|
|Windows|Windows-Rechner auf dem das System `estw` läuft und auf dem die csv-Dateien für den Zugriff von `estw` abgelegt werden.|
|Host|Linux-Server auf ein Docker-Dienst läuft. Der Platzhalter 192.168.xxx.xxx innerhalb dieser Dokumentation ist die Adresse des Hostes.|
  |Docker|Docker-Container auf dem Host mit MariaDB-Datenbank. Im Beispiel heißt der Container `franzidb`.|

#### Aufsetzen des Docker-Containers mit der Datenbank

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

Verbinde dich von Windows aus mit dem DB-Server als `root` und führe das Skript in der Datei `create_database.sql` aus. Du findest die Datei im Projektverzeichnis des Repositories unter 
`sql`.

Benutze dafür ein SQL-Programm deiner Wahl, z.B. `Heidi`. Alternativ kannst du das Script auch mit dem Terminalprogramm `mysqlsh` starten.

~~~batch
mysqlsh --uri root@192.168.xxx.xxx:3300 --sql -f create_database.sql
~~~

### Anlegen des DB-Users

Führe das Skript in der Datei [create_user.sql](#create_usersql) aus.

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

Speichern der Daten in csv-Dateien. Führe dazu das Skript in der Datei [export.sql](#exportsql) aus.

~~~batch
mysqlsh --uri root@192.168.xxx.xxx:3300 --sql -f export.sql
~~~

### Transfer der Dateien nach Windows

Kopieren der csv-Dateien vom export-Verzeichnis des Hostes auf den Windows-Rechner. Führe dazu das Skript in der Datei [transfer.sql](#transfer-der-csv-dateien-nach-windows) aus.

~~~batch
mysqlsh --uri root@192.168.xxx.xxx:3300 --sql -f transfer.sql
~~~

# Projektierung

## Projekte

## Elemente



## Files

## create_database.sql

Auszug aus 07/2025. Das Original befindet sich im Repository.

~~~sql
create database estw;
use estw;

create table projekte (
id int not null,
bezeichnung varchar(50) not null,
beschreibung varchar(254),
primary key (id)
) engine=innodb default charset=latin1 row_format=default;
create unique index index1 on projekte (bezeichnung);

create table elementtypen (
id int not null,
bezeichnung varchar(50) not null,
beschreibung varchar(254),
primary key (id)
) engine=innodb default charset=latin1 row_format=default;
create unique index index2 on elementtypen (bezeichnung);

create table elemente (
id int not null auto_increment,
projekt_id int not null,
elementtyp_id int not null,
hauptelement_id int NULL COMMENT 'Ist ein Zusatzelement, wenn nicht NULL',
bezeichnung varchar(50) not null,
beschreibung varchar(254),
PFZS ENUM('J','N') NULL COMMENT 'Rangierstraßenzielsperre',
PR ENUM('J','N') NULL COMMENT 'Rangierstraßen zulässig',
PRS ENUM('J','N') NULL COMMENT 'Rangierstraßenstart zulässig',
PRZ ENUM('J','N') NULL COMMENT 'Rangierstraßenziel zulässig',
PSSL ENUM('J','N') NULL COMMENT 'Stellsperre Links',
PSSR ENUM('J','N') NULL COMMENT 'Stellsperre Rechts',
PZ ENUM('J','N') NULL COMMENT 'Zugstraßen zulässig',
PZS ENUM('J','N') NULL COMMENT 'Zugstraßenstart zulässig',
PZZ ENUM('J','N') NULL COMMENT 'Zugstraßenziel zulässig',
lupe1x int NULL COMMENT 'Lupe 1: X-Koordinate',
lupe1y int NULL COMMENT 'Lupe 1: Y-Koordinate',
rotation int not null default 0 COMMENT 'Rotation in 90° Schritten (0=0°, 1=90°, 2=180°, 3=270°)',
mirror ENUM('J','N') NOT NULL DEFAULT 'N' COMMENT 'Spiegeln',
primary key (id),
check (rotation IN (0, 1, 2, 3)),
foreign key (projekt_id) references projekte (id) on update cascade,
foreign key (elementtyp_id) references elementtypen (id) on update cascade,
foreign key (hauptelement_id) references elemente (id)  -- nur ids erlaubt, die in elemente.id existieren
) engine=innodb default charset=latin1 row_format=default;

create unique index index3 on elemente (projekt_id,elementtyp_id,bezeichnung);
~~~

#### create_user.sql

~~~sql
CREATE USER 'estw'@'192.168.%' IDENTIFIED BY 'estw';
GRANT SELECT  ON *.* TO 'estw'@'192.168.%';
GRANT FILE  ON *.* TO 'estw'@'192.168.%';
FLUSH PRIVILEGES;
~~~

#### export.sql

Auszug aus 07/2025. Das Original befindet sich im Repository.

~~~sql
SELECT 
   id,
   bezeichnung,
   beschreibung
from projekte
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/projekte.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT 
   id,
   bezeichnung,
   beschreibung
from elementtypen
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/elementtypen.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT 
   id,
   projekt_id,
   elementtyp_id,
   hauptelement_id,
   bezeichnung,
   beschreibung
from elemente
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/elemente.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';
~~~

#### transfer.sql

Beispiel. Das Original befindet sich im Repository.

~~~sql
scp <user>@192.168.xxx.xxx:/home/<user>/docker/export/projekte.csv d:\estwdaten
scp <user>@192.168.xxx.xxx:/home/<user>/docker/export/elementtypen>.csv d:\estwdaten
scp <user>@192.168.xxx.xxx:/home/<user>/docker/export/elemente>.csv d:\estwdaten
~~~


# Autor

Mathias Rentsch<br>
rentsch@online.de<br>
Juli 2025
