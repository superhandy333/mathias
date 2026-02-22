<button>ESTW</button>

<small>[Entwicklung](../develop.md) / [ESTW](estw.md) / Files</small>

# Files

- [create_database.sql](create_databasesql.md)
- [create_user.sql](#create_usersql)
- [export.bat](#exportbat)
- [export.sql](#exportsql)
- [transfer.sql](#transfersql)

## create_user.sql

~~~sql
CREATE USER 'estw'@'192.168.%' IDENTIFIED BY 'estw';
GRANT SELECT  ON *.* TO 'estw'@'192.168.%';
GRANT FILE  ON *.* TO 'estw'@'192.168.%';
FLUSH PRIVILEGES;
~~~

## export.bat

~~~
mysqlsh --uri estw@192.168.xxx.xx:3300/estw --sql -f export.sql
~~~

## export.sql

Auszug aus 07/2025. Das Original befindet sich im [Repository](https://github.com/superhandy333).

~~~sql
use estw;

SELECT 
   id,
   bezeichnung,
   beschreibung
FROM projekte
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/projekte.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT 
   id,           -- 0
   bezeichnung,  -- 1
   beschreibung  -- 2
FROM elementtypen
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/elementtypen.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT 
   id,               -- 0
   projekt_id,      -- 1
   typ_id,          -- 2
   unterelementart, -- 3
   hauptelement_id, -- 4
   bezeichnung,     -- 5
   beschreibung,    -- 6
   PFZS,            -- 7
   PR,             -- 8
   PRS,            -- 9
   PRZ,            -- 10
   PSSL,           -- 11
   PSSR,           -- 12
   PZ,             -- 13
   PZS,            -- 14
   PZZ,            -- 15
   lupe1x,        -- 16
   lupe1y,        -- 17
   rotation,     -- 18
   mirror       -- 19
FROM elemente
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/elemente.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT 
   id,               -- 0
   projekt_id,      -- 1
   typ_id,          -- 2
   bezeichnung,     -- 3
   beschreibung     -- 4
FROM fahrstrassen
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/fahrstrassen.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT
   id,               -- 0
   bezeichnung,     -- 1
   beschreibung     -- 2
FROM fahrstrassenelementtypen
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/fahrstrassenelementtypen.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT 
   id,                       -- 0
   fahrstrasse_id,           -- 1
   element_id,               -- 2
   fahrstrassenelementtyp_id,-- 3
   sollstellung              -- 4
FROM fahrstrassenelemente
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/fahrstrassenelemente.csv'
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

~~~

## transfer.sql

Beispiel. Das Original befindet sich im [Repository](https://github.com/superhandy333).

~~~sql
scp papa@192.168.xxx.xx:/home/papa/docker/export/*.csv d:\rentsch\estwdaten
~~~

<hr>
Autor<br>

Mathias Rentsch<br>
rentsch@online.de<br>
Januar 2026
