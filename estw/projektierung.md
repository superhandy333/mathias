<button>ESTW</button>

<small>[Entwicklung](../develop.md) / [ESTW](estw.md) / Projektierung</small>

# Projektierung

Dokumentation des Projektierungsprozesses mit dem Programmsystem `estw` mit Festlegungen der funktionalen Anforderungen zu Weichen, Signalen und Fahrstrassen.

<!-- TOC -->

- [Projekte](#projekte)
- [Elementtypen](#elementtypen)
- [Elemente](#elemente)
    - [Datenfelder](#datenfelder)
    - [Programmfälle](#programmf%C3%A4lle)
    - [Lupenbilder](#lupenbilder)
- [Fahrstrassen](#fahrstrassen)
- [Fahrstrassenelementtypen](#fahrstrassenelementtypen)
- [Fahrstrassenelemente](#fahrstrassenelemente)

<!-- /TOC -->

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
|bezeichnung|Name des Elementtyps (max. 50 Zeichen)|x|
|beschreibung|Erläuternder Text als Beschreibung zum Elementtyp (max. 254 Zeichen)||

|ID|Bezeichnung|Beschreibung|
|-|-|-|
|1| G| Gleis|
|2| W| Weiche|
|3| S| Signal|
|4| B| Blindziel|
|5| A| Auflöseelement|
|7| Z| Zusatzelement|
|99| T| Test|

## Elemente

### Datenfelder

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|id|Eindeutige Identifikationsnummer des Elements. Diese ID wird in der Tabelle [Fahrstrassenelemente](#fahrstrassenelemente) referenziert.|x|
|projekt_id|Identifikationsnummer des [Projektes](#projekte), dem das Element angehört|x|
|typ_id|Identifikationsnummer des Elementtyps. Wert aus dem Feld `id` der Tabelle [Elementtypen](#elementtypen).|x|
|unterelementart|Bildliche Differenzierung eines Elements innerhalb eines Elementtyps. Werte siehe [Projektierungstabelle](#projektierungstabelle).|x|
|bezeichnung|Name des Elements (max. 50 Zeichen)|x|
|beschreibung|Erläuternder Text als Beschreibung zum Element (max. 254 Zeichen)||
|lupe1x|Horizontale Position im des Elements im Bildschirmraster|x|
|lupe1y|Vertikale Position im des Elements im Bildschirmraster|x|
|rotation|Drehwinkel des Elements: 0: 0 Grad, 1: 90 Grad, 2: 180 Grad, 3: 270 Grad|x|
|mirror|Angabe, ob Element gespiegelt dargestellt werden soll|x|


### Programmfälle

|Fall|Beschreibung|G|W|S|B|A|
|-|-|:-:|:-:|:-:|:-:|:-:|
|PR|Rangierstraße zulässig<br>Gleis darf Element einer Rangierstrasse sein.|x|||||
|PZ|Zugstraße zulässig<br>Gleis darf Element einer Zugstrasse sein.|x|||||
|PRS|Rangierstraßenstart zulässig<br>Signal darf Startelement einer Rangierstrasse sein.|||x|||
|PRZ|Rangierstraßenziel zulässig<br>Signal darf Zielelement einer Rangierstrasse sein.|||x|||
|PZS|Zugstraßenstart zulässig<br>Signal darf Startelement einer Zugstrasse sein.|||x|||
|PZZ|Zugstraßenziel zulässig<br>Signal darf Zielelement einer Zugstrasse sein.|||x|||
|PSSL|Umstellsperre links<br>Weiche ist in Stellung Links verriegelt ||x||||
|PSSR|Umstellsperre rechts<br>Weiche ist in Stellung Rechts verriegelt||x||||
|PZSP|Zielsperre<br>Signal kann während es als Flankenschutzelement dient, nicht gleichzeitig Ziel einer Rangierstrasse dienen. Signal kann während es Ziel einer Rangierstrasse, nicht gleichzeitig als Flankenschutzelement dienen.|||x|||

### Lupenbilder

|Nr.|Typ-ID|Typ|Grafik|Rotation|Mirror|Unterelementart|
|-|-|-|-|-|-|-|
|1|1|Gleis|![svg01](images/svg01.svg)|0|N|1|
|2|1|Gleis|![svg02](images/svg02.svg)|1|N|1|
|3|1|Gleis|![svg03](images/svg03.svg)|0|N|2|
|4|1|Gleis|![svg04](images/svg04.svg)|1|N|2|
|5|1|Gleis|![svg05](images/svg05.svg)|2|N|2|
|6|1|Gleis|![svg06](images/svg06.svg)|3|N|2|
|7|1|Gleis|![svg07](images/svg07.svg)|0|J|2|
|8|1|Gleis|![svg08](images/svg08.svg)|1|J|2|
|9|1|Gleis|![svg09](images/svg09.svg)|2|J|2|
|10|1|Gleis|![svg10](images/svg10.svg)|3|J|2|
|xx|1|Gleis|![svgxx](images/svg20.svg)|0|N|3|
|xx|1|Gleis|![svgxx](images/svg21.svg)|1|N|3|
|xx|1|Gleis|![svgxx](images/svg22.svg)|2|N|3|
|xx|1|Gleis|![svgxx](images/svg23.svg)|3|N|3|
|11|2|Weiche|![svg11](images/svg11.svg)|0|N||
|12|2|Weiche|![svg12](images/svg12.svg)|1|N||
|13|2|Weiche|![svg13](images/svg13.svg)|2|N||
|14|2|Weiche|![svg14](images/svg14.svg)|3|N||
|15|2|Weiche|![svg15](images/svg15.svg)|0|J||
|16|2|Weiche|![svg16](images/svg16.svg)|1|J||
|17|2|Weiche|![svg17](images/svg17.svg)|2|J||
|18|2|Weiche|![svg18](images/svg18.svg)|3|J||
|xx|1|Blindziel|![svg30](images/svg30.svg)|0|0|1|
|xx|1|Blindziel|![svg31](images/svg31.svg)|0|0|1|
|xx|1|Blindziel|![svg32](images/svg32.svg)|0|0|1|
|xx|1|Blindziel|![svg33](images/svg33.svg)|0|0|1|

## Fahrstrassen


## Fahrstrassenelementtypen

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|id|Eindeutige Identifikationsnummer des Fahrstrassenelementtyps. Diese ID wird in der Tabelle `Fahrstrassenelemente` referenziert.|x|
|bezeichnung|Name des Fahrstrassenelementtyps (max. 50 Zeichen)|x|
|beschreibung|Erläuternder Text als Beschreibung zum Fahrstrassenelementtyp (max. 254 Zeichen)||

## Fahrstrassenelemente

<hr>
Autor<br>

Mathias Rentsch<br>
rentsch@online.de<br>
Februar 2026
