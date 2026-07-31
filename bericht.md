# Analyse der Datenstruktur

## Zweck der Datenstruktur
Die analysierte Struktur bildet eine kleine MariaDB-Zugriffsschicht ab. `MariaDbConnector` stellt Verbindungen her und führt SQL-Befehle aus. `Table` und `Row` modellieren ein Ergebnis-Set, `Field` kapselt Werte mit Typinformation, und `InsertStatement` sowie `UpdateStatement` sammeln Werte für vorbereitete Schreiboperationen.

## Beteiligte Klassen, Strukturen und Beziehungen
- `maria::MariaDbConnector` verwaltet Konfiguration, Verbindung und Abfragen.
- `maria::ConnectionConfig` speichert Verbindungsparameter.
- `maria::Table` enthält `ColumnNames` und `Rows`.
- `maria::Row` referenziert die Spaltennamen und speichert Feldwerte.
- `tools::Field` kapselt einen einzelnen Wert mit Typ und Nullzustand.
- `maria::InsertStatement` und `maria::UpdateStatement` sammeln Zieltabellen, Spaltennamen, Werte und bei `UpdateStatement` zusätzlich eine WHERE-Bedingung.

Die Beziehungen sind stark gekuppelt: `MariaDbConnector::getTable()` baut `Table` und `Row` aus einem `sql::ResultSet`, `Row` liest über `m_columnNames` auf Felder zu, und die Statement-Typen werden später direkt in SQL-Strings und Parameterbindungen überführt.

## Einzelanalyse der Klassen und Strukturen

### Kritisch: Inkonsistente Rückgabetypen bei `Row::getRowID`
Betroffene Stelle:
`maria::Row::getRowID`

Problem:
Die Deklaration in der Klasse verspricht `std::string const &`, die Definition liefert jedoch `std::string` by value. Das ist ein echter Widerspruch zwischen Schnittstelle und Implementierung.

Auswirkung:
Der Code ist strukturell inkonsistent und vom Compiler bereits als Fehler erkennbar. Zusätzlich ist die Semantik unklar: Der Member `m_rowID` wirkt wie ein langlebiger Identifikator, die Implementierung kopiert aber nur einen Wert heraus.

Empfehlung:
Deklaration und Definition müssen denselben Typ haben. Wenn der Member intern bleiben soll, ist eine Rückgabe als `const &` naheliegend; wenn eine Kopie gewollt ist, muss die Deklaration angepasst werden.

### Kritisch: Metadaten werden vor der Nullprüfung verwendet
Betroffene Stelle:
`maria::MariaDbConnector::getTable`

Problem:
`rs->getMetaData()` wird direkt benutzt, bevor `meta` auf Null geprüft wird. Erst später folgt `if (!meta)`, also nach bereits erfolgten Zugriffen.

Auswirkung:
Falls keine Metadaten geliefert werden, kann dies zu einem Absturz oder undefiniertem Verhalten führen, bevor die Fehlerbehandlung greifen kann.

Empfehlung:
Den Nullcheck unmittelbar nach `getMetaData()` platzieren und erst danach alle Zugriffe auf `meta` ausführen.

### Hoch: Nullzustand und Typzustand von `tools::Field` widersprechen sich
Betroffene Stelle:
`tools::Field::Field()`, `tools::Field::m_type`, `tools::Field::m_isNull`, `tools::Field::getType()`

Problem:
Der Default-Konstruktor soll laut Kommentar ein Null-Field erzeugen, aber `m_type` ist trotzdem auf `DataType::String` voreingestellt. Ein Null-Field meldet damit weiterhin den Typ `String`.

Auswirkung:
Code kann einen echten Nullzustand als String missverstehen. Das führt zu inkonsistenten Entscheidungen, wenn zuerst der Typ und erst danach der Nullzustand geprüft wird.

Empfehlung:
Ein expliziter Null-Typ oder ein konsistentes Variant-/Optional-Modell wäre robuster. Der Nullzustand und der Typzustand sollten sich nicht gegenseitig widersprechen.

### Hoch: Schema wird doppelt und mutabel gehalten
Betroffene Stelle:
`maria::Table::ColumnNames`, `maria::Row::m_columnNames`, `maria::MariaDbConnector::getTable`

Problem:
Die Spaltennamen werden sowohl in `Table::ColumnNames` als auch in jeder `Row` über denselben `shared_ptr` gespeichert. Damit existiert Schema-Information doppelt und zusätzlich mutabel.

Auswirkung:
Wenn die Spaltennamen nach dem Aufbau verändert werden, können Feldzugriffe und Schema auseinanderlaufen. Die Struktur ist dadurch leicht inkonsistent zu machen.

Empfehlung:
Das Schema sollte immutable gemacht werden, zum Beispiel als `shared_ptr<const std::vector<std::string>>`, oder zentral nur an einer Stelle gehalten werden.

### Hoch: `Row` hängt an einem nicht abgesicherten Pointer
Betroffene Stelle:
`maria::Row::Row(PColumnNames)`, `maria::Row::getColumnIndexByName`, `maria::Row::getField`, `maria::Row::get*`

Problem:
`Row` nimmt einen `shared_ptr` auf Spaltennamen entgegen, prüft aber nicht auf Null. Alle Zugriffe dereferenzieren `m_columnNames` direkt.

Auswirkung:
Ein leerer oder ungültiger Pointer führt zu Absturzrisiken. Die Klasse dokumentiert den benötigten Besitz- und Lebenszeit-Zustand nicht selbst.

Empfehlung:
Den Konstruktor auf eine nicht-null, lesbare Schema-Referenz umstellen oder mindestens eine harte Vorbedingung im Konstruktor erzwingen.

### Hoch: SQL-Identifier und WHERE-Bedingungen sind nicht sauber modelliert
Betroffene Stelle:
`maria::InsertStatement::into`, `maria::InsertStatement::set`, `maria::UpdateStatement::table`, `maria::UpdateStatement::set`, `maria::UpdateStatement::where`

Problem:
Identifier werden nur auf Leerheit und Länge geprüft, nicht auf erlaubte Zeichen oder korrektes Quoting. Besonders `where()` übernimmt freien SQL-Text ohne Struktur oder Absicherung.

Auswirkung:
Je nach Quelle der Eingaben entstehen malformed SQL oder Injektionsrisiken. Selbst bei internem Gebrauch bleibt die Schnittstelle missverständlich, weil Identifikatoren und rohe SQL-Fragmente gleich behandelt werden.

Empfehlung:
Identifier separat validieren oder quoten. Für `where()` sollte klar sein, ob bewusst roher SQL-Text erlaubt ist oder ob eine strukturierte Bedingung vorgesehen ist.

### Hoch: Typmodell der Statement-Typen ist enger als das Wertmodell
Betroffene Stelle:
`maria::InsertStatement`, `maria::UpdateStatement`, `MariaDbConnector::Insert(...)`, `MariaDbConnector::Update(...)`, `tools::Field`

Problem:
`tools::Field` unterstützt mehrere Typen, aber `InsertStatement` und `UpdateStatement` können nur `std::string` und `int` speichern. Der Connector behandelt später ebenfalls nur diese beiden Typen.

Auswirkung:
Die API sieht allgemeiner aus, als sie ist. Erweiterungen wie bool, float, double, unsigned oder DateTime sind im Wertmodell schon angelegt, werden im Schreibpfad aber nicht abgebildet.

Empfehlung:
Entweder das Modell bewusst auf die wirklich unterstützten Typen reduzieren oder die Statement- und Binding-Seite vollständig an `tools::Field` angleichen.

### Mittel: `m_rowID` ist vorhanden, aber ohne klare Verantwortlichkeit
Betroffene Stelle:
`maria::Row::m_rowID`, `maria::Row::getRowID`, `maria::MariaDbConnector::getTable`

Problem:
`Row` besitzt einen Member `m_rowID` und einen Getter, aber beim Aufbau aus dem ResultSet wird dieser Wert nie befüllt.

Auswirkung:
Der Member suggeriert eine fachliche Identität, die tatsächlich nicht genutzt wird. Das ist missverständlich und kann später zu falschen Annahmen führen.

Empfehlung:
Entweder den Member entfernen oder seine Quelle und Semantik eindeutig definieren und beim Aufbau konsequent füllen.

### Mittel: Doppelte Spaltennamen werden technisch gelöst, fachlich aber nur teilweise
Betroffene Stelle:
`maria::MariaDbConnector::getTable` mit `addColumnName`

Problem:
Bei doppelten Spaltennamen werden Suffixe wie `_1` angehaengt. Dadurch werden Kollisionen vermieden, aber die Beziehung zum Originalnamen geht verloren.

Auswirkung:
Die resultierenden Namen sind eindeutig, entsprechen aber nicht mehr unmittelbar dem SQL-Resultset. Bei Joins kann das missverständlich werden.

Empfehlung:
Die Umbenennung dokumentieren oder zusätzlich eine Abbildung von Original- zu Anzeigename speichern.

### Mittel: `Table` ist sehr offen und dadurch leicht missbrauchbar
Betroffene Stelle:
`maria::Table::ColumnNames`, `maria::Table::Rows`

Problem:
`Table` ist als `struct` mit öffentlichen Membern ausgelegt. Es gibt keine Kapselung der Invarianten.

Auswirkung:
Externer Code kann die Struktur leicht in ungültige oder inkonsistente Zustände versetzen.

Empfehlung:
Wenn die Struktur stabil bleiben soll, sollten die Daten zumindest teilweise gekapselt oder klarer geregelt werden.

### Niedrig: Benennung und Rollen sind nicht durchgängig eindeutig
Betroffene Stelle:
`InsertStatement::m_Values`, `UpdateStatement::m_Values`, `ConnectionConfig`, `Table::ColumnNames`

Problem:
Einige Membernamen sind fachlich brauchbar, aber nicht durchgängig präzise. Besonders `m_Values` sagt wenig darüber aus, dass es sich um Paare aus Spaltennamen und `Field` handelt.

Auswirkung:
Die Lesbarkeit leidet, und die Verantwortlichkeit des Members wird erst durch Kontext klar.

Empfehlung:
Präzisere Namen wie `m_columnsAndValues` oder eine eigene Tupel-/Strukturart würden die Rolle klarer machen.

## Zusammenspiel aller Bestandteile
Das Zusammenspiel ist grundsätzlich nachvollziehbar, aber an mehreren Stellen zu locker gekuppelt. `MariaDbConnector` erzeugt `Table` und `Row`, während `Row` von einem gemeinsam genutzten Schema abhängt. Das erschwert die Sicherung von Invarianten. Gleichzeitig sind die Schreibpfade weniger flexibel als das vorhandene Wertmodell. Dadurch entsteht eine Struktur, die nach außen allgemein wirkt, intern aber nur einen Teil der möglichen Fälle tatsächlich korrekt abbildet.

Die größten Abhängigkeiten mit Inkonsistenzrisiko sind:
- `Field`-Nullzustand versus Typzustand
- `Table::ColumnNames` versus `Row::m_columnNames`
- SQL-Identifier versus freie SQL-Fragmente
- Statement-Builder versus unterstützte Feldtypen

## Zusammenfassung der wichtigsten Probleme
Die schwerwiegendsten Punkte sind die echte Signaturinkonsistenz bei `Row::getRowID`, der zu spät gesetzte Nullcheck in `getTable()` und das widersprüchliche Zustandsmodell von `tools::Field`. Hinzu kommen strukturelle Risiken durch doppelt gehaltene Spaltennamen, fehlende Absicherung des `Row`-Schemas und eine uneinheitliche Behandlung von SQL-Identifiern und Parametern.

## Empfohlene Reihenfolge der Korrekturen
1. Signatur von `Row::getRowID` bereinigen.
2. Nullprüfung für `meta` in `getTable()` vorziehen.
3. Null- und Typmodell von `tools::Field` konsistent machen.
4. Besitz und Unveränderlichkeit der Spaltennamen festlegen.
5. `InsertStatement` und `UpdateStatement` fachlich sauberer von rohem SQL trennen.
6. Typunterstützung der Statement- und Bindungslogik an `tools::Field` angleichen.

## Vorschlag für eine konsistentere Gesamtstruktur
Eine schlankere und robustere Variante wäre:
- `Table` besitzt ein unveränderliches Schema.
- `Row` referenziert dieses Schema nur lesend.
- `Field` hat einen expliziten Nullzustand.
- `InsertStatement` und `UpdateStatement` unterscheiden streng zwischen Identifikatoren, Werten und freien SQL-Fragmenten.

Damit wären die Daten- und Lebenszeitbeziehungen einfacher, und doppelt gespeicherte Information würde reduziert.

## Offene Fragen vor einer Überarbeitung
- Soll `m_rowID` eine fachliche Bedeutung haben oder nur ein Platzhalter sein?
- Darf `where()` bewusst rohen SQL-Text speichern?
- Müssen doppelte Spaltennamen nur technisch aufgelöst oder fachlich modelliert werden?
- Soll `Field` wirklich alle in `DataType` genannten Typen unterstützen?
- Darf `Table::ColumnNames` nach der Erzeugung verändert werden?

## Bereits sinnvoll und konsistent umgesetzt
- Vor Datenbankoperationen wird mit `test_connected()` geprüft, ob eine Verbindung aktiv ist.
- Daten werden weitgehend über Prepared Statements gebunden.
- `Field` kapselt Typkonvertierungen und wirft bei ungültigen Fällen statt still falsche Werte zu liefern.
- `ConnectionConfig` hat sinnvolle Defaultwerte für Host und Port.

## Hinweis zur Bewertung
Ein Teil der Bewertung bleibt ohne weitere fachliche Vorgaben bewusst vorsichtig formuliert. Das betrifft vor allem die Frage, ob freier SQL-Text in `where()` gewollt ist, ob `m_rowID` eine echte Domendenidentität darstellen soll und ob die breite Typunterstützung in `Field` nur vorbereitet oder bereits verbindlich genutzt werden soll.