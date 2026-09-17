# Projektanweisungen

## Regeln für Prüfroutine isvalid_estwdata

Diese Regeln gelten für die Prüfroutine `isvalid_estwdata` in validate.cpp.

Die maßgeblichen Datentypen (`Projekt`, `Elementtyp`, `Element`, `FahrstrassenElementtyp`,
`FahrstrassenElement`, `Fahrstrasse`, `FullProjekt`, `EstwData` sowie `ec_t`) sind in structs.h
definiert. structs.h ist die einzige verbindliche Quelle (Master) für diese Structs; bei
Unklarheiten über Feldnamen oder -typen gilt der Stand dort, nicht dieser Text.

Die Prüfroutine soll diese Signatur haben:

```cpp
auto isvalid_estwdata(EstwData const&,ec_t&) -> bool;
```

- Fehler sollen in den logcollector (ec) geschrieben werden.
- Fehler, die in den logcollector geschrieben werden, sollen immer die Typangabe (Projekt, Elementtyp, Element, ...) und die Id enthalten.
- Gibt es am Ende keine Fehler, soll die Prüfroutine `true` zurückgeben, sonst `false`.

Es gelten folgende Prüfbedingungen:

- Ein string gilt als leer, wenn er die Zeichenlänge 0 hat.
- `b_ws` = string besteht nur aus Leerzeichen (Whitespaces)
- `b_null` = string darf leer sein, falls es nicht leer ist, darf er nicht `b_ws` sein
- `b_notnull` = string darf nicht leer sein und er darf nicht `b_ws` sein

### In Projekt

- id darf nicht 0 sein
- id muss innerhalb projekte_t eindeutig sein
- bezeichnung ist b_notnull
- bezeichnung muss innerhalb projekte_t eindeutig sein
- beschreibung ist b_null

### In Elementtyp

- id darf nicht 0 sein
- id muss innerhalb elementtypen_t eindeutig sein
- bezeichnung ist b_notnull
- bezeichnung muss innerhalb elementtypen_t eindeutig sein
- beschreibung ist b_null

### In Element

- id darf nicht 0 sein
- id muss innerhalb elemente_t eindeutig sein
- bezeichnung ist b_notnull
- beschreibung ist b_null
- ProjektId darf nicht 0 sein
- ProjektId muss einen Wert aus projekte_t.id enthalten
- TypId darf nicht 0 sein
- TypId muss einen Wert aus elementtypen_t.id enthalten
- rotation darf nur die Werte 0, 1, 2 oder 3 annehmen
- UnterElementArt darf nur die Werte 1, 2 oder 3 annehmen
- Die Kombination von bezeichnung, ProjektId und TypId muss innerhalb elemente eindeutig sein
- Die Kombination von ProjektId, lupe1x und lupe1y muss innerhalb elemente eindeutig sein

### In FahrstrassenElementtyp

- id darf nicht 0 sein
- id muss innerhalb fahrstrassenelementtypen_t eindeutig sein
- bezeichnung ist b_notnull
- bezeichnung muss innerhalb fahrstrassenelementtypen_t eindeutig sein
- beschreibung ist b_null

### In Fahrstrassen

- id darf nicht 0 sein
- id muss innerhalb fahrstrassen_t eindeutig sein
- ProjektId darf nicht 0 sein
- ProjektId muss einen Wert aus projekte_t.id enthalten
- TypId darf nur die Werte 1 oder 2 annehmen
- bezeichnung ist b_notnull
- beschreibung ist b_null
- Die Kombination aus ProjektId und bezeichnung muss innerhalb fahrstrassen_t eindeutig sein

### In Fahrstrassenelemente

- id darf nicht 0 sein
- id muss innerhalb fahrstrassenelemente_t eindeutig sein
- FahrstrassenId darf nicht 0 sein
- FahrstrassenId muss einen Wert aus fahrstrassen_t.id enthalten
- ElementId darf nicht 0 sein
- ElementId muss einen Wert aus elemente_t.id enthalten
- TypId darf nicht 0 sein
- TypId muss einen Wert aus fahrstrassenelementtypen_t.id enthalten
- Die Kombination von FahrstrasseId und ElementId muss innerhalb von fahrstrassenelemente_t eindeutig sein
- Die ProjektId des Elementes, welches über ElementId referenziert ist, muss mit der ProjektId der Fahrstrasse, die über FahrstrassenId referenziert ist, übereinstimmen.

### Zusätzliche Konsistenzprüfungen je Fahrstrasse (bezogen auf ihre Fahrstrassenelemente)

Die Prüfungen müssen für jede Fahrstrasse aus fahrstrassen_t durchgeführt werden, auch wenn ihr kein Fahrstrassenelement zugeordnet ist.

Die zulässigen Kombinationen aus Elementtyp und Fahrstrassenelementtyp sind in der Zuordnungstabelle in [datenstrukturen.md](datenstrukturen.md#zulässige-zuordnungen-zu-elementtypen) verbindlich festgelegt und gelten zusätzlich zu den folgenden fahrstrassentypabhängigen Regeln.

#### Für jede Rangierstrasse (Fahrstrasse.TypId = 1) gilt

- Eine Rangierstrasse darf eine beliebige Anzahl von Fahrstrassenelementen mit TypId 1 (Fahrwegelement) besitzen. Das referenzierte Element muss `Element.TypId` 1 (Gleis), 2 (Weiche), 3 (Signal), 4 (Blindziel) oder 5 (Aufloeseelement) haben.
- Eine Rangierstrasse muss genau ein Fahrstrassenelement mit TypId 2 (Fahrstrassenstart) besitzen.
  - Das über das Fahrstrassenstartelement (TypId 2) referenzierte Element muss `Element.TypId` 3 (Signal) haben.
- Eine Rangierstrasse muss genau ein Fahrstrassenelement mit TypId 3 (Fahrstrassenziel) besitzen.
  - Das über das Fahrstrassenzielelement (TypId 3) referenzierte Element muss `Element.TypId` 3 (Signal) oder 4 (Blindziel) haben.
- Eine Rangierstrasse muss genau ein Fahrstrassenelement mit TypId 4 (Aufloeseelement) besitzen.
  - Das über das Aufloeseelement (TypId 4) referenzierte Element muss `Element.TypId` 5 (Aufloeseelement) haben.
- Eine Rangierstrasse darf kein Fahrstrassenelement mit TypId 5 (Flankenschutzelement) besitzen.
- Eine Rangierstrasse darf kein Fahrstrassenelement mit TypId 6 (VorwegElement) besitzen.
- Eine Rangierstrasse darf kein Fahrstrassenelement mit TypId 7 (Unterwegselement) besitzen.
- Eine Rangierstrasse darf kein Fahrstrassenelement mit TypId 8 (Flankenschutztransportelement) besitzen.

#### Für jede Zugstrasse (Fahrstrasse.TypId = 2) gilt

- Eine Zugstrasse darf eine beliebige Anzahl von Fahrstrassenelementen mit TypId 1 (Fahrwegelement) besitzen. Das referenzierte Element muss `Element.TypId` 1 (Gleis), 2 (Weiche), 3 (Signal), 4 (Blindziel) oder 5 (Aufloeseelement) haben.
- Eine Zugstrasse muss genau ein Fahrstrassenelement mit TypId 2 (Fahrstrassenstart) besitzen.
  - Das über das Fahrstrassenstartelement (TypId 2) referenzierte Element muss `Element.TypId` 3 (Signal) haben.
- Eine Zugstrasse muss genau ein Fahrstrassenelement mit TypId 3 (Fahrstrassenziel) besitzen.
  - Das über das Fahrstrassenzielelement (TypId 3) referenzierte Element muss `Element.TypId` 3 (Signal) oder 4 (Blindziel) haben.
- Eine Zugstrasse muss genau ein Fahrstrassenelement mit TypId 4 (Aufloeseelement) besitzen.
  - Das über das Aufloeseelement (TypId 4) referenzierte Element muss `Element.TypId` 5 (Aufloeseelement) haben.
- Eine Zugstrasse darf eine beliebige Anzahl von Fahrstrassenelementen mit TypId 5 (Flankenschutzelement) besitzen.
  - Das über ein Flankenschutzelement (TypId 5) referenzierte Element muss `Element.TypId` 2 (Weiche) oder 3 (Signal) haben.
- Eine Zugstrasse darf höchstens ein Fahrstrassenelement mit TypId 6 (VorwegElement) besitzen.
  - Falls ein VorwegElement (TypId 6) vorhanden ist, muss das darüber referenzierte Element `Element.TypId` 3 (Signal) haben.
- Eine Zugstrasse darf eine beliebige Anzahl von Fahrstrassenelementen mit TypId 7 (Unterwegselement) besitzen.
  - Das über ein Unterwegselement (TypId 7) referenzierte Element muss `Element.TypId` 3 (Signal) haben.
- Eine Zugstrasse darf eine beliebige Anzahl von Fahrstrassenelementen mit TypId 8 (Flankenschutztransportelement) besitzen.
  - Das über ein Flankenschutztransportelement (TypId 8) referenzierte Element muss `Element.TypId` 1 (Gleis), 2 (Weiche), 3 (Signal), 4 (Blindziel) oder 5 (Aufloeseelement) haben.


