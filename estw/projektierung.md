
# Projektierung <!-- omit in toc -->

Privates Stellwerkssystem `estw2` zum Steuern von Modelleisenbahnen.

- [Projekte](#projekte)
- [Elementtypen](#elementtypen)
- [Elemente](#elemente)
- [Fahrstrassen](#fahrstrassen)
- [Fahrstrassenelementtypen](#fahrstrassenelementtypen)
- [Fahrstrassenelemente](#fahrstrassenelemente)
- [Autor](#autor)


## Projekte

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|ID|Eindeutige Identifikationsnummer des Projektes. Diese ID wird in den Tabellen `Elemente` und `Fahrstrassen` referenziert.|x|
|Bezeichnung|Name des Projektes (max. 50 Zeichen)|x|
|Beschreibung|Erläuternder Text als Beschreibung zum Projektes (max. 254 Zeichen)||

## Elementtypen

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|ID|Eindeutige Identifikationsnummer des Elementtyps. Diese ID wird in der Tabelle `Elemente` referenziert.|x|
|Bezeichnung|Name des Elementtyps (max. 50 Zeichen)|x|
|Beschreibung|Erläuternder Text als Beschreibung zum Elementtyp (max. 254 Zeichen)||

|ID|Bezeichnung|Beschreibung|
|-|-|-|
|1| G| Gleis|
|2| W| Weiche|
|3| S| Signal|
|4| B| Blindziel|
|5| A| Auflöseelement|
|6| P| Prellbock|
|7| Z| Zusatzelement|
|99| T| Test|


## Elemente

|Nr.|Typ-ID|Typ|Grafik|Rotation|Mirror|
|-|-|-|-|-|-|
|1|1|Gleis|![svg01](images/svg01.svg)|0|N|
|2|1|Gleis|![svg02](images/svg02.svg)|1|N|
|3|1|Gleis|![svg03](images/svg03.svg)|1|J|
|4|1|Gleis|![svg04](images/svg04.svg)|2|J|
|5|1|Gleis|![svg05](images/svg05.svg)|3|J|
|6|1|Gleis|![svg06](images/svg06.svg)|4|J|
|7|2|Weiche|![svg07](images/svg07.svg)|0|N|
|8|2|Weiche|![svg08](images/svg08.svg)|1|N|
|9|2|Weiche|![svg09](images/svg09.svg)|2|N|
|10|2|Weiche|![svg10](images/svg10.svg)|3|N|
|11|2|Weiche|![svg11](images/svg11.svg)|0|J|
|12|2|Weiche|![svg12](images/svg12.svg)|1|J|
|13|2|Weiche|![svg13](images/svg13.svg)|2|J|
|14|2|Weiche|![svg14](images/svg14.svg)|3|J|


## Fahrstrassen

## Fahrstrassenelementtypen

|Feld|Beschreibung|Pflichtfeld|
|-|-|-|
|ID|Eindeutige Identifikationsnummer des Fahrstrassenelementtyps. Diese ID wird in der Tabelle `Fahrstrassenelemente` referenziert.|x|
|Bezeichnung|Name des Fahrstrassenelementtyps (max. 50 Zeichen)|x|
|Beschreibung|Erläuternder Text als Beschreibung zum Fahrstrassenelementtyp (max. 254 Zeichen)||

## Fahrstrassenelemente





## Autor

Mathias Rentsch<br>
rentsch@online.de<br>
Oktober 2025
