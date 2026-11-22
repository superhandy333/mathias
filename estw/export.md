<button>ESTW</button>

<small>[Entwicklung](../develop.md) / [ESTW](main.md) / [Files](files.md) / export.sql</small>

# export.sql

Die Tabelle `projekte` muss vor dem Export die 24 Ausschnittspalten enthalten.
Für vorhandene Datenbanken die [additive Migration](create_database.md#bestehende-datenbank-erweitern)
ausführen; das Schema für neue Datenbanken enthält die Felder bereits.

`projekte.csv` enthält 27 Felder: `id`, `bezeichnung`, `beschreibung`, danach
`x1,y1,zoom1` bis `x8,y8,zoom8`. Der Trenner bleibt `[[SEP]]`. Alte Projektdateien
mit nur drei Feldern bleiben in estw3 lesbar; die acht Ausschnitte beginnen dort
mit `0/0/1`. `elemente.csv` enthält 19 Felder, ohne `hauptelement_id`.

Alle sechs CSV-Dateien werden mit `CHARACTER SET utf8mb4` als UTF-8 geschrieben,
auch wenn die Quelltabellen Latin-1 verwenden. Ohne diese Klausel findet beim
Export keine Zeichenkonvertierung statt; siehe
[MariaDB: SELECT INTO OUTFILE](https://mariadb.com/docs/server/reference/sql-statements/data-manipulation/selecting-data/select-into-outfile).
Vor einer erneuten Ausführung müssen die bisherigen Exportdateien entfernt oder
verschoben werden, da `INTO OUTFILE` vorhandene Dateien nicht überschreibt.

~~~sql
-- estw3
-- export.sql
-- Author: Mathias Rentsch
-- Februar 2026
-- äöüÄÖÜß

use estw;

-- projekte.csv: 27 Felder, acht X/Y/Zoom-Tripel nach den Projekttexten.
-- Alle CSV-Dateien als UTF-8 exportieren, auch bei latin1-Quelltabellen.
SELECT 
   id,            -- 0
   bezeichnung,   -- 1
   beschreibung,  -- 2
   x1,            -- 3
   y1,            -- 4
   zoom1,         -- 5
   x2,            -- 6
   y2,            -- 7
   zoom2,         -- 8
   x3,            -- 9
   y3,            -- 10
   zoom3,         -- 11
   x4,            -- 12
   y4,            -- 13
   zoom4,         -- 14
   x5,            -- 15
   y5,            -- 16
   zoom5,         -- 17
   x6,            -- 18
   y6,            -- 19
   zoom6,         -- 20
   x7,            -- 21
   y7,            -- 22
   zoom7,         -- 23
   x8,            -- 24
   y8,            -- 25
   zoom8          -- 26
FROM projekte
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/projekte.csv'
CHARACTER SET utf8mb4
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT 
   id,           -- 0
   bezeichnung,  -- 1
   beschreibung  -- 2
FROM elementtypen
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/elementtypen.csv'
CHARACTER SET utf8mb4
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT 
   id,               -- 0
   projekt_id,      -- 1
   typ_id,          -- 2
   unterelementart, -- 3
   bezeichnung,     -- 4
   beschreibung,    -- 5
   PZSP,            -- 6
   PR,              -- 7
   PRS,             -- 8
   PRZ,             -- 9
   PSSL,            -- 10
   PSSR,            -- 11
   PZ,              -- 12
   PZS,             -- 13
   PZZ,             -- 14
   lupe1x,          -- 15
   lupe1y,          -- 16
   rotation,        -- 17
   mirror           -- 18
FROM elemente
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/elemente.csv'
CHARACTER SET utf8mb4
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
CHARACTER SET utf8mb4
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';

SELECT
   id,               -- 0
   bezeichnung,     -- 1
   beschreibung     -- 2
FROM fahrstrassenelementtypen
ORDER BY id
INTO OUTFILE '/home/hostfiles/export/fahrstrassenelementtypen.csv'
CHARACTER SET utf8mb4
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
CHARACTER SET utf8mb4
FIELDS TERMINATED BY '[[SEP]]'
LINES TERMINATED BY '\n';
~~~
