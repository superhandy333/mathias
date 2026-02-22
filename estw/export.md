<button>ESTW</button>

<small>[Entwicklung](../develop.md) / [ESTW](estw.md) / [Files](files.md) / export.sql</small>

# export.sql

~~~sql
-- estw3
-- export.sql
-- Author: Mathias Rentsch
-- Februar 2026
-- äöüÄÖÜß

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
   PZSP,            -- 7
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
