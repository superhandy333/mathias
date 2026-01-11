<button>ESTW</button>

<small>[Entwicklung](../develop.md) / [ESTW](estw.md) / Files</small>

# Files

<!-- TOC -->

- [create_database.sql](#create_databasesql)
- [create_user.sql](#create_usersql)
- [export.sql](#exportsql)
- [transfer.sql](#transfersql)

<!-- /TOC -->

## create_database.sql

Auszug aus 01/2026. Das Original befindet sich im [Repository](https://github.com/superhandy333/csharp/blob/6b0e968f4034b0f4640607b1c1efba44735616f8/CPP/estw/sql/create_database.sql).

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
typ_id int not null,
hauptelement_id int comment 'Ist ein Zusatzelement, wenn nicht NULL',
bezeichnung varchar(50) not null,
beschreibung varchar(254),
PFZS ENUM('N','J') comment 'Rangierstraßenzielsperre',
PR ENUM('N','J') comment 'Rangierstraßen zulässig',
PRS ENUM('N','J') comment 'Rangierstraßenstart zulässig',
PRZ ENUM('N','J') comment 'Rangierstraßenziel zulässig',
PSSL ENUM('N','J') comment 'Stellsperre Links',
PSSR ENUM('N','J') comment 'Stellsperre Rechts',
PZ ENUM('N','J') comment 'Zugstraßen zulässig',
PZS ENUM('N','J') comment 'Zugstraßenstart zulässig',
PZZ ENUM('N','J') comment 'Zugstraßenziel zulässig',
lupe1x int not null comment 'Lupe 1: X-Koordinate',
lupe1y int not null comment 'Lupe 1: Y-Koordinate',
rotation int not null default 0 comment 'Rotation in 90° Schritten (0=0°, 1=90°, 2=180°, 3=270°)',
mirror ENUM('N','J') not null default 'N' comment 'Spiegeln',
primary key (id),
check (rotation IN (0, 1, 2, 3)),
foreign key (projekt_id) references projekte (id) on update cascade,
foreign key (typ_id) references elementtypen (id) on update cascade,
foreign key (hauptelement_id) references elemente (id)  -- nur ids erlaubt, die in elemente.id existieren
) engine=innodb default charset=latin1 row_format=default;
create unique index index3 on elemente (projekt_id,typ_id,bezeichnung);
create unique index index4 on elemente (projekt_id,lupe1x,lupe1y);

create table fahrstrassen (
id int not null auto_increment,
projekt_id int not null,
typ_id int not null default 1 comment 'Typ der Fahrstraße, 1 = Rangierstraße, 2 = Zugfahrstraße',
bezeichnung varchar(50) not null,
beschreibung varchar(254),
primary key (id),
check (typ_id IN (1, 2)),
foreign key (projekt_id) references projekte (id) on update cascade
) engine=innodb default charset=latin1 row_format=default;
create unique index index5 on fahrstrassen (projekt_id, bezeichnung);

create table fahrstrassenelementtypen (
id int not null,
bezeichnung varchar(50) not null,
beschreibung varchar(254),
primary key (id)
) engine=innodb default charset=latin1 row_format=default;
create unique index index6 on fahrstrassenelementtypen (bezeichnung);

create table fahrstrassenelemente (
id int not null auto_increment,
fahrstrasse_id int not null,
element_id int not null,
fahrstrassenelementtyp_id int not null,
primary key (id),
foreign key (fahrstrasse_id) references fahrstrassen (id) on update cascade,
foreign key (element_id) references elemente (id) on update cascade,
foreign key (fahrstrassenelementtyp_id) references fahrstrassenelementtypen (id) on update cascade
) engine=innodb default charset=latin1 row_format=default;
create unique index index7 on fahrstrassenelemente (fahrstrasse_id, element_id);

-- Trigger, der prüft, ob die projekt_id von elemente und fahrstrassen übereinstimmt
-- beim Einfügen in fahrstrassenelemente
DROP TRIGGER IF EXISTS check_projekt_id_on_insert;

DELIMITER %%

CREATE TRIGGER check_projekt_id_on_insert
BEFORE INSERT ON fahrstrassenelemente
FOR EACH ROW
BEGIN
    DECLARE element_pid INT DEFAULT NULL;
    DECLARE fahrstrasse_pid INT DEFAULT NULL;

    SELECT e.projekt_id
      INTO element_pid
      FROM elemente e
     WHERE e.id = NEW.element_id
     LIMIT 1;

    SELECT f.projekt_id
      INTO fahrstrasse_pid
      FROM fahrstrassen f
     WHERE f.id = NEW.fahrstrasse_id
     LIMIT 1;

    IF element_pid IS NULL
       OR fahrstrasse_pid IS NULL
       OR element_pid <> fahrstrasse_pid
    THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'projekt_id mismatch (oder element/fahrstrasse nicht gefunden)';
    END IF;
END%%

DELIMITER ;
~~~

## create_user.sql

~~~sql
CREATE USER 'estw'@'192.168.%' IDENTIFIED BY 'estw';
GRANT SELECT  ON *.* TO 'estw'@'192.168.%';
GRANT FILE  ON *.* TO 'estw'@'192.168.%';
FLUSH PRIVILEGES;
~~~

## export.sql

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

## transfer.sql

Beispiel. Das Original befindet sich im Repository.

~~~sql
scp <user>@192.168.xxx.xxx:/home/<user>/docker/export/projekte.csv d:\estwdaten
scp <user>@192.168.xxx.xxx:/home/<user>/docker/export/elementtypen>.csv d:\estwdaten
scp <user>@192.168.xxx.xxx:/home/<user>/docker/export/elemente>.csv d:\estwdaten
~~~

<hr>
Autor<br>

Mathias Rentsch<br>
rentsch@online.de<br>
Januar 2026
