# Aufwandsanalyse: Edit-Modus LupeForm + MariaDB-Persistierung

Status: Analyse abgeschlossen, keine Code-Änderungen vorgenommen (wie beauftragt).

## Nutzerentscheidungen (per Rückfrage geklärt)
- Verschieben: Zwei-Klick-Workflow (Element markieren, Zielzelle klicken) - KEINE CadForm-Erweiterung nötig.
- Löschen: weiches Löschen (Node ausblenden, aus Liste entfernen) - KEINE CadForm-Erweiterung nötig.
- Speicherzeitpunkt: sofort je Einzelaktion (Add/Edit/Move/Delete -> direkt INSERT/UPDATE/DELETE), kein Diff/Batch-Mechanismus nötig.
- Edit-Modus nur nutzbar/aktivierbar wenn DataSource=mariadb (Buttons/Menüpunkte sonst deaktiviert).
- Fahrstrassen-Modus grob mit einbezogen (analoge, aber geringer detaillierte Betrachtung).

## Wichtige Codebase-Fakten
- `LupeForm` (lupeform.h/.cpp) hat aktuell NUR den Simulationsmodus; Konstruktor nimmt `FullProjekt const&` (read-only). Kein Mode-Enum, keine Edit-UI, keine Fahrstrassen-UI vorhanden.
- Aufruf-Kette: main.cpp (WinMain, hält `EstwData data` nicht-const) -> `showProjektForm(..., EstwData const& data)` -> `ProjektForm` -> `OnSimulation` erzeugt `LupeForm(*fullProjekt)` mit `fullProjekt` = Iterator auf `data.FullProjekte` (const wegen Parametertyp `EstwData const&`). Für Persistenz/Rückschreiben muss dieser Pfad nicht-const gemacht werden bzw. es braucht Callbacks von LupeForm nach außen.
- `structs.h`: `FullProjekt` erlaubt keine Kopie (nur Move) - Rückgabe/Kopieren von FullProjekt an Aufrufer ist eingeschränkt; Änderungen müssen eher über Callbacks pro Element/Fahrstrasse zurückgemeldet werden statt über Rückgabe eines ganzen FullProjekt.
- `aktionen.cpp` hat nur `load_*_from_mariadb` Funktionen (SELECT). Keine INSERT/UPDATE/DELETE-Funktionen für irgendeine Tabelle vorhanden. `sql.h` enthält nur SELECT-Statements.
- `/cpp/tools/maria.h` (außerhalb Workspace, nur lesend/mit Freigabe änderbar) bietet bereits: `MariaDbConnector::Insert/Update(std::string sql, Args...)`, `Insert(InsertStatement)`, `Update(UpdateStatement)`, `Execute(sql)` (für DELETE geeignet). ABER:
  - `Insert()` gibt die Anzahl betroffener Zeilen zurück (executeUpdate), NICHT die generierte Auto-Increment-ID. Für die neue Element-ID: zusätzlicher Aufruf `conn.Select("SELECT LAST_INSERT_ID() AS id")` auf derselben Connection (funktioniert ohne Framework-Änderung).
  - `InsertStatement::set()`/`UpdateStatement::set()` haben nur Overloads für `std::string` und `int`, KEIN `bool`-Overload (obwohl `tools::Field` intern `Bool` kennt). Workaround ohne Framework-Änderung: bool-Felder (PR, PRS, PRZ, PSSL, PSSR, PZ, PZS, PZZ, PZSP, Mirror) beim Schreiben nach `int(wert ? 1 : 0)` casten.
- `CadForm` (/cpp/tools/cadform/cadform.h) bietet nur `setCadMouseDown` (kein MouseMove/MouseUp/Drag), `CadPoint` bietet kein `remove_node`. Durch die getroffenen Entscheidungen (Zwei-Klick-Move, weiches Löschen) ist hierfür KEINE Änderung am externen Framework nötig.
- `Estw` (estw/estw.h) hat `addElement`/`addFahrstrasse`, aber KEIN `removeElement`. Für weiches Löschen reicht es, das Element in der UI unsichtbar/inaktiv zu machen; das verwaiste Estw-Element bleibt harmlos im Speicher (keine Interaktion mehr möglich) - alternativ kleine Erweiterung `Estw::removeElement(id)`.
- `LupeElement` (lupe/lupeelement.h/.cpp) hat bereits `GetMenuItemList()/ExecuteCommand()` (Kontextmenü-Mechanismus, aktuell für Simulationsaktionen wie BFS/BES/Reset) - erweiterbar für Edit-Aktionen. Kein Setter, um `EP` nach dem Anlegen zu ändern (nötig fürs Verschieben) - kleine Erweiterung nötig.
- `checkLupeElementData()` prüft nur Namensleere + Positions-Duplikat unter den aktuell geladenen `lupeElemente` - nicht die vollständigen `isvalid_estwdata`-Regeln (z. B. Eindeutigkeit Bezeichnung+ProjektId+TypId, Rotation-Domäne, UnterElementArt-Domäne). Für sicheres Sofort-Schreiben in die DB sollten die relevanten Einzel-Regeln zusätzlich geprüft werden.
- Keine bestehenden Dialog-Formulare für Element-Erfassung/-Bearbeitung (`elementform.*` existiert nicht). `configform.cpp` dient als Referenzmuster für ein neues Win32Form-Dialogformular.

## Aufwandsblöcke (grobe Einordnung: Klein / Mittel / Groß)

1. **Mode-Infrastruktur LupeForm** (Mittel)
   - `enum class LupeMode { Simulation, Edit, Fahrstrasse }`, Konstruktorparameter, moduspezifischer Aufbau von Menü/Buttons, moduspezifisches Dispatching in `cadMouseDown`.
   - Neue Einstiegspunkte in `ProjektForm` (Buttons "Edit", "Fahrstrasse") und `main.cpp`-Wiring; Deaktivierung wenn `DataSource != MariaDB`.

2. **Nicht-konstanter Datenfluss / Callback-Wiring** (Mittel)
   - `FullProjekt`/`EstwData` liegen aktuell "read-only" tief in der Aufrufkette. Da `FullProjekt` nicht kopierbar ist, empfiehlt sich: LupeForm bekommt eine änderbare Arbeitskopie der relevanten `elemente_t`/`fahrstrassen_t` plus Callbacks (`OnElementAdded/Updated/Deleted`, analog für Fahrstrasse) an den Aufrufer, der `data.Elemente`/`data.Fahrstrassen`/`data.FullProjekte` inkrementell nachführt (kein kompletter Reload nötig, passt zur "sofort speichern"-Entscheidung).

3. **Persistenzschicht (aktionen.cpp / sql.h)** (Mittel-Groß)
   - Neue Funktionen: `insert_element_to_mariadb`, `update_element_to_mariadb`, `delete_element_from_mariadb` (+ analog `fahrstrassen`, `fahrstrassenelemente` für Fahrstrassen-Modus).
   - ID-Ermittlung nach Insert via `SELECT LAST_INSERT_ID()`.
   - Bool->int-Konvertierung an den Aufrufstellen.
   - Referenzielle Prüfung vor DELETE: Element darf nicht gelöscht werden, wenn es in `fahrstrassenelemente_t` referenziert ist (sonst FK-Fehler in DB + Verstoß gegen AGENTS.md-Konsistenzregeln).

4. **Editier-Dialoge (neue UI)** (Mittel-Groß)
   - Neues Dialogformular (`elementform.h/.cpp`, Muster: `configform.cpp`) für Anlegen/Bearbeiten eines Elements: Bezeichnung, TypId, abhängige Zusatzfelder (Signal: PRS/PRZ/PZS/PZZ/PZSP; Blind: PRZ/PZZ), UnterElementArt, Rotation, Mirror, optional HauptElementId.
   - Wiederverwendung/Erweiterung der Validierung aus `checkLupeElementData` um die relevanten `isvalid_estwdata`-Einzelregeln (Eindeutigkeit, Domänenwerte, FK) vor dem eigentlichen DB-Schreiben.

5. **Grid-Interaktionen Add/Move/Delete** (Mittel)
   - "Neu": Klick auf leere Zelle im Edit-Modus -> Dialog -> Insert in DB -> Estw-/Lupe-Objekt zur Laufzeit erzeugen (bestehende `create*`-Methoden sind darauf vorbereitet, `estwInit` müsste in wiederverwendbare Einzel-Erzeugungsfunktionen aufgeteilt werden, da aktuell nur beim initialen Laden inline ausgeführt).
   - "Verschieben": bestehendes Markieren wiederverwenden (aktuell auf Signal-Typ beschränkt, müsste für Edit-Modus auf alle Typen erweitert werden), Klick auf Zielzelle -> Prüfung frei -> UPDATE lupe1x/lupe1y -> neue `LupeElement`-Methode zum Aktualisieren von `EP`/`cadNode`-Position (aktuell nicht vorhanden, aber ohne Framework-Änderung machbar über vorhandenes `cadNode->set_location()`).
   - "Löschen": Kontextmenü-Erweiterung (`GetMenuItemList`/`ExecuteCommand` moduspezifisch), Sicherheitsabfrage, FK-Check, DELETE, Node ausblenden, aus `lupeElemente` entfernen.

6. **Vorab-Validierung je Aktion** (Klein-Mittel)
   - Da sofort statt gesammelt gespeichert wird, sollten pro Aktion nur die jeweils betroffenen Regeln aus `isvalid_estwdata` geprüft werden (kein Volllauf nötig); optional zusätzlich defensiv ein Gesamt-`isvalid_estwdata`-Lauf nach jeder Aktion (Klein, da Routine schon existiert).

7. **Fahrstrassen-Modus (grobe Betrachtung)** (Groß)
   - Analoge CRUD-Funktionalität für `Fahrstrasse`/`FahrstrassenElement` (eigene Dialoge: Fahrstrassen-Stammdaten, Zuordnung von Elementen mit Rolle TypId/SollStellung).
   - Zusätzliche Konsistenzregeln aus AGENTS.md (genau 1x Start/Ziel/Aufloese, Zieltyp S/B, max. 1 Vorweg-Element mit Typ S) müssten synchron im Dialog geprüft werden, bevor gespeichert wird.
   - Persistenzfunktionen analog Punkt 3 für `fahrstrassen`/`fahrstrassenelemente`-Tabellen.

8. **Aktualisierung des In-Memory-Caches in main.cpp** (Klein)
   - Nach jeder erfolgreichen DB-Aktion inkrementelles Nachführen von `data.Elemente`/`data.Fahrstrassen`/`data.FullProjekte` über die Callbacks aus Punkt 2 (kein Neuladen/`isDataLoaded`-Reset nötig).

## Kein Eingriff nötig in
- `/cpp/tools/cadform/*` (dank Zwei-Klick-Move und weichem Löschen).
- `/cpp/tools/maria.h` grundsätzlich nicht zwingend nötig (Workarounds: `SELECT LAST_INSERT_ID()`, bool->int-Cast); optionale kleine Komfort-Erweiterungen (bool-Overload, `getLastInsertId()`) wären "nice to have", aber nur mit ausdrücklicher Freigabe umzusetzen.

## Offene Restrisiken / To-Dos vor Umsetzung
- `Estw::removeElement` fehlt: entweder kleine Ergänzung oder bewusst verwaistes Objekt in Kauf nehmen (empfohlen: kleine Ergänzung für Sauberkeit).
- `estwInit` müsste refaktoriert werden in einzeln aufrufbare Erzeugungsfunktionen pro Elementtyp, um "Neu"-Elemente zur Laufzeit ohne Formular-Neustart hinzuzufügen.
- Prüfen, ob `elemente.id`/`fahrstrassen.id`/`fahrstrassenelemente.id` in der MariaDB tatsächlich AUTO_INCREMENT sind (Annahme, nicht verifiziert - DB-Schema nicht Teil des Workspace).
