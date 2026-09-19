# Datenstrukturen

Die folgenden Abschnitte beschreiben die grundlegenden Datenstrukturen, die im System verwendet werden. Sie umfassen Projekte, Elementtypen, Elemente und deren Eigenschaften.

## Projekte

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|id|Eindeutige Identifikationsnummer des Projektes. Diese ID wird in den Tabellen [Elemente](#elemente) und [Fahrstrassen](#fahrstrassen) referenziert.|x|
|bezeichnung|Name des Projektes (max. 50 Zeichen)|x|
|beschreibung|Erläuternder Text als Beschreibung zum Projektes (max. 254 Zeichen)||

## Elementtypen

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|id|Eindeutige Identifikationsnummer des Elementtyps. Diese ID wird in der Tabelle [Elemente](#elemente) referenziert.|x|
|bezeichnung|Ausgeschriebener Name des Elementtyps, beispielsweise `Weiche` (max. 50 Zeichen)|x|
|beschreibung|Erläuternder Text als Beschreibung zum Elementtyp (max. 254 Zeichen)||

Elementtypen im System sind fest definierte Kategorien von Elementen. Ihre Katalogdaten werden als Datenbankeinträge verwaltet; die stabilen IDs sind zusätzlich zentral durch `ElementTypId` in [structs.h](../structs.h) benannt. Die fachliche und technische Identifikation erfolgt ausschließlich über diese ID; die `bezeichnung` dient der Anzeige und darf nicht zur Steuerung der Programmlogik verwendet werden.

|ElementTypId|Bezeichnung|
|-|-|
|1|Gleis|
|2|Weiche|
|3|Signal|
|4|Blindziel|
|5|Auflöseelement|
|99|Test|

## Elemente

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|id|Eindeutige Identifikationsnummer des Elements. Diese ID wird in der Tabelle [Fahrstrassenelemente](#fahrstrassenelemente) referenziert.|x|
|projekt_id|Identifikationsnummer des [Projektes](#projekte), dem das Element angehört. Wert aus dem Feld `id` der Tabelle [Projekte](#projekte).|x|
|typ_id|Identifikationsnummer des Elementtyps. Wert aus dem Feld `id` der Tabelle [Elementtypen](#elementtypen).|x|
|unterelementart|Bildliche und fachliche Differenzierung eines Elements innerhalb eines Elementtyps. Für Signale sind die Werte in [signale.md](signale.md#signalarten-nach-unterelementart) verbindlich festgelegt.|x|
|bezeichnung|Name des Elements (max. 50 Zeichen)|x|
|beschreibung|Erläuternder Text als Beschreibung zum Element (max. 254 Zeichen)||
|lupe1x|Horizontale Position im des Elements im Bildschirmraster|x|
|lupe1y|Vertikale Position im des Elements im Bildschirmraster|x|
|rotation|Drehwinkel des Elements: 0: 0 Grad, 1: 90 Grad, 2: 180 Grad, 3: 270 Grad|x|
|mirror|Angabe, ob Element gespiegelt dargestellt werden soll|x|


### Programmfälle

|Fall|Beschreibung|Gleis|Weiche|Signal|Blindziel|Auflöseelement|
|-|-|:-:|:-:|:-:|:-:|:-:|
|PR|Rangierstraße zulässig<br>Gleis darf Element einer Rangierstrasse sein.|x|||||
|PZ|Zugstraße zulässig<br>Gleis darf Element einer Zugstrasse sein.|x|||||
|PRS|Rangierstraßenstart zulässig<br>Signal darf Startelement einer Rangierstrasse sein.|||x|||
|PRZ|Rangierstraßenziel zulässig<br>Signal darf Zielelement einer Rangierstrasse sein.|||x|x||
|PZS|Zugstraßenstart zulässig<br>Signal darf Startelement einer Zugstrasse sein.|||x|||
|PZZ|Zugstraßenziel zulässig<br>Signal darf Zielelement einer Zugstrasse sein.|||x|x||
|PSSL|Umstellsperre links<br>Weiche ist in Stellung Links verriegelt ||x||||
|PSSR|Umstellsperre rechts<br>Weiche ist in Stellung Rechts verriegelt||x||||
|PZSP|Zielsperre<br>Signal kann während es als Flankenschutzelement dient, nicht gleichzeitig Ziel einer Rangierstrasse dienen. Signal kann während es Ziel einer Rangierstrasse, nicht gleichzeitig als Flankenschutzelement dienen.|||x|||

### Mögliche Projektierungen

|Nr.|Typ|Typ|U-Art|Rotation|Mirror|
|-|-|-|-|-|-|
|1|Gleis|1|1|0|N|
|2|Gleis|1|1|1|N|
|3|Gleis|1|2|0|N|
|4|Gleis|1|2|1|N|
|5|Gleis|1|2|2|N|
|6|Gleis|1|2|3|N|
|7|Gleis|1|2|0|J|
|8|Gleis|1|2|1|J|
|9|Gleis|1|2|2|J|
|10|Gleis|1|2|3|J|
|11|Gleis|1|3|0|N|
|12|Gleis|1|3|1|N|
|13|Gleis|1|3|2|N|
|14|Gleis|1|3|3|N|
|15|Weiche|2|1|0|N|
|16|Weiche|2|1|1|N|
|17|Weiche|2|1|2|N|
|18|Weiche|2|1|3|N|
|19|Weiche|2|1|0|J|
|20|Weiche|2|1|1|J|
|21|Weiche|2|1|2|J|
|22|Weiche|2|1|3|J|
|23|Signal|3|1|0|N|
|24|Signal|3|1|1|N|
|25|Signal|3|1|2|N|
|26|Signal|3|1|3|N|
|27|LS-Signal|3|2|0|N|
|28|LS-Signal|3|2|1|N|
|29|LS-Signal|3|2|2|N|
|30|LS-Signal|3|2|3|N|
|31|Blindziel|4|1|0|N|
|32|Blindziel|4|1|1|N|
|33|Blindziel|4|1|2|N|
|34|Blindziel|4|1|3|N|
|35|Auflöse|5|1|0|N|
|36|Auflöse|5|1|1|N|
|37|Auflöse|5|1|2|N|
|38|Auflöse|5|1|3|N|

Die Projektierungen des Elementtyps **Signal** mit den Unterelementarten `1` und `2` entsprechen den in [signale.md](signale.md#signalarten-nach-unterelementart) festgelegten Signalarten.

## Fahrstrassen

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|id|Eindeutige Identifikationsnummer der Fahrstrasse. Diese ID wird in der Tabelle `Fahrstrassenelemente` referenziert.|x|
|projekt_id|Eindeutige Identifikationsnummer des Projekts, zu dem die Fahrstrasse gehört. Wert aus dem Feld `id` der Tabelle [Projekte](#projekte).|x|
|typ_id|Typ der Fahrstrasse. 1 = Rangierstrasse, 2 = Zugstrasse.|x|
|bezeichnung|Name des Fahrstrassenelementtyps (max. 50 Zeichen)|x|
|beschreibung|Erläuternder Text als Beschreibung zum Fahrstrassenelementtyp (max. 254 Zeichen)||


## Fahrstrassenelementtypen

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|id|Eindeutige Identifikationsnummer des Fahrstrassenelementtyps. Diese ID wird in der Tabelle `Fahrstrassenelemente` referenziert.|x|
|bezeichnung|Name des Fahrstrassenelementtyps (max. 50 Zeichen)|x|
|beschreibung|Erläuternder Text als Beschreibung zum Fahrstrassenelementtyp (max. 254 Zeichen)||

Fahrstrassenelementtypen im System sind fest definierte Kategorien von Fahrstrassenelementen, die allerdings nicht im Code, sondern als Datenbankeinträge verwaltet werden. Es ist davon auszugehen, dass sich diese nicht ändern.

### Zulässige Zuordnungen zu Elementtypen

Die folgende Zuordnung ist verbindlich. Ein Fahrstrassenelementtyp darf einem Element nur zugewiesen werden, wenn er in der Zeile des betreffenden Elementtyps aufgeführt ist. Für andere Elementtypen sind keine Fahrstrassenelementtypen zugelassen.

|Elementtyp|Zulässige Fahrstrassenelementtypen|
|-|-|
|Gleis|Fahrwegelement, Flankenschutztransportelement|
|Weiche|Fahrwegelement, Flankenschutzelement, Flankenschutztransportelement|
|Signal|Fahrwegelement, Fahrstrassenstart, Fahrstrassenziel, Flankenschutzelement, Flankenschutztransportelement, Vorwegelement, Unterwegselement|
|Blindziel|Fahrwegelement, Fahrstrassenziel, Flankenschutztransportelement|
|Auflöseelement|Fahrwegelement, Auflöseelement, Flankenschutztransportelement|

## Fahrstrassenelemente

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|id|Eindeutige Identifikationsnummer des Fahrstrassenelements.|x|
|fahrstrasse_id|Identifikationsnummer der Fahrstrasse, zu der das Element gehört. Wert aus dem Feld `id` der Tabelle [Fahrstrassen](#fahrstrassen).|x|
|element_id|Identifikationsnummer des Elements. Wert aus dem Feld `id` der Tabelle [Elemente](#elemente).|x|
|fahrstrassenelementtyp_id|Identifikationsnummer des Fahrstrassenelementtyps. Wert aus dem Feld `id` der Tabelle [Fahrstrassenelementtypen](#fahrstrassenelementtypen).|x|
|sollstellung|Sollstellung, die das Element einnehmen muss, damit die Fahrstrasse eingestellt werden kann. Bei einem Startsignal wird sie als [Hauptsignalgeschwindigkeit](signale.md#hauptsignalgeschwindigkeit) verwendet.|x|

Die Kombination aus `fahrstrasse_id` und `element_id` ist eindeutig (jedes Element kann einer Fahrstrasse nur einmal zugeordnet werden).



