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

#### Programmfall-Validierung

Verbindliche Quelle ist die Tabelle **„Programmfälle“** in
[Projektierung](https://superhandy333.github.io/mathias/estw/projektierung.html#programmf%C3%A4lle)
(abgeglichen am 9. Oktober 2026). Die dortigen Kürzel bedeuten G = Gleis,
W = Weiche, S = Signal, B = Blindziel und A = Auflöseelement.
Die Zuordnungen stimmen mit der Tabelle in [datenstrukturen.md](datenstrukturen.md#programmfälle) überein.

| Programmfall | Zulässige Elementtypen | Stabile `ElementTypId` |
|---|---|---|
| PR | Gleis (G) | 1 |
| PZ | Gleis (G) | 1 |
| PRS | Signal (S) | 3 |
| PRZ | Signal (S), Blindziel (B) | 3, 4 |
| PZS | Signal (S) | 3 |
| PZZ | Signal (S), Blindziel (B) | 3, 4 |
| PZSP | Signal (S) | 3 |
| PSSL | Weiche (W) | 2 |
| PSSR | Weiche (W) | 2 |

- Geprüft werden alle neun Bool-Felder aus `Element` in `structs.h`.
- Ausschließlich `true` gilt als gesetzt. `false` löst unabhängig vom Typ keinen Programmfallfehler aus.
- Ein gesetzter Programmfall an einem nicht in seiner Tabellenzeile zugelassenen Typ ist ein harter Fehler.
  Auflöseelemente, Testelemente und andere Typen ohne Zuordnung dürfen keinen dieser Fälle setzen.
- `PSSL` und `PSSR` sind ausschließlich an Weichen zulässig. An einer Weiche dürfen beide nicht
  gleichzeitig `true` sein. Beide `false` oder genau eine gesetzte Sperre sind zulässig.
  Der Ausschluss wird unabhängig von anderen Programmfällen geprüft.
- Ein Prellbock bleibt ein Gleis (`ElementTypId::Gleis`, `UnterElementArt = 3`);
  für ihn gelten dieselben Programmfall-Zuordnungen wie für andere Gleise.
- Die Typprüfung erfolgt erst nach erfolgreicher Auflösung der Referenz `Element.TypId`.
  Bei fehlender Referenz bleibt der vorhandene Referenzfehler maßgeblich.
- Fachliche Entscheidungen verwenden ausschließlich `ElementTypId`, keine Bezeichnungen.
  Fehler nennen Element-ID, Elementtyp samt stabiler Typ-ID und den betroffenen Programmfall.
- `isvalid_element_data` verwendet dieselben Prüfungen; sie greifen somit auch vor Elementänderungen
  und der Speicherung im Element-Editmodus.

Regressionen stehen in `tests/validate_tests.cpp` (CTest: `estw_data_validation`).
Sie prüfen jeden der neun Fälle gegen alle sechs vorhandenen Elementtypen,
alle vier Kombinationen der Weichensperren, nicht gesetzte Fälle, Prellböcke,
die Prüfung erst nach gültiger Typreferenz und die Fehlerkonventionen.
Die MRDB-Roundtrip-Daten setzen die Fälle an den jeweils zulässigen Elementtypen;
`PSSL` und `PSSR` werden an zwei getrennten Weichen gespeichert.

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

#### Doppelte Fahrstrassenelemente

Die bereits vorhandene Schlüsselprüfung verwendet `(FahrstrasseId, ElementId)`.
Zwei Zuordnungen mit diesem Schlüssel sind unabhängig von ihrer `TypId` ein harter Fehler.
Die Meldung nennt die konkrete Fahrstrassen-ID, doppelte Element-ID, betroffene
Fahrstrassenelement-ID und die konfliktverursachende Fahrstrassenelement-ID;
der Elementtyp wird nur bei gültiger Element- und Typreferenz ergänzt.
Dieselbe Element-ID darf in unterschiedlichen Fahrstrassen vorkommen.

Maßgeblich ist die flache Tabelle `EstwData.Fahrstrassenelemente`. Der
Fahrstrassen-Speicherpfad überführt die verschachtelten Arbeitszuordnungen mit
`replaceFahrstrassenTables` in genau diese Tabelle, bevor `isvalid_estwdata`
aufgerufen wird. `FullProjekte` und die verschachtelten Anzeige-/Arbeitskopien
werden nicht zusätzlich als eigenständige Datensätze gezählt.

Regressionen prüfen doppelte Zuordnungen bei gleicher und unterschiedlicher
Rolle, zulässige Wiederverwendung eines Elements in verschiedenen Fahrstrassen,
die Überführung verschachtelter Arbeitsdaten über `replaceFahrstrassenTables`
sowie fehlende Element-/Typreferenzen ohne unsichere Zugriffe.

Prüfstand am 9. Oktober 2026: Build mit dem vorhandenen Clang-/Ninja-Setup und
`ESTW_BUILD_TESTS=ON` erfolgreich; sämtliche fünf CTest-Prüfreihen bestanden.
Die zuletzt ergänzten Überführungs-/Referenztests in `estw_data_validation`
wurden anschließend erneut erfolgreich ausgeführt. Der Build erfolgte aufgrund
der ausdrücklichen Anforderung in dieser Validierungsaufgabe.

#### Sollstellung nach Elementtyp

Die Zulässigkeit von `SollStellung` richtet sich nach dem Elementtyp des `EstwElement`, das über
`FahrstrassenElement.ElementId` referenziert wird. Der Elementtyp wird über dessen `TypId` bestimmt.
Die Prüfung wird nur durchgeführt, wenn sowohl das referenzierte Element als auch dessen Elementtyp
vorhanden sind; ungültige Referenzen werden durch die vorangehenden Referenzprüfungen gemeldet.

| Elementtyp | Zulässige `SollStellung` |
|-|-|
| Gleis | `0` |
| Weiche | `-1`, `1` |
| Signal | `-3` bis einschließlich `16` |
| Auflöseelement | `0` |
| Blindziel / Blind | `0` |

Für andere Elementtypen führt diese Sollstellungsprüfung keine zusätzliche Einschränkung ein.

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


