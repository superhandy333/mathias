# Analyse der Datenstruktur

## Zweck der Datenstruktur
Die analysierte Struktur bildet eine kleine MariaDB-Zugriffsschicht ab. `MariaDbConnector` stellt Verbindungen her und fuehrt SQL-Befehle aus. `Table` und `Row` modellieren ein Ergebnis-Set, `Field` kapselt Werte mit Typinformation, und `InsertStatement` sowie `UpdateStatement` sammeln Werte fuer vorbereitete Schreiboperationen.

## Beteiligte Klassen, Strukturen und Beziehungen
- `maria::MariaDbConnector` verwaltet Konfiguration, Verbindung und Abfragen.
- `maria::ConnectionConfig` speichert Verbindungsparameter.
- `maria::Table` enthaelt `ColumnNames` und `Rows`.
- `maria::Row` referenziert die Spaltennamen und speichert Feldwerte.
- `tools::Field` kapselt einen einzelnen Wert mit Typ und Nullzustand.
- `maria::InsertStatement` und `maria::UpdateStatement` sammeln Zieltabellen, Spaltennamen, Werte und bei `UpdateStatement` zusaetzlich eine WHERE-Bedingung.

Die Beziehungen sind stark gekuppelt: `MariaDbConnector::getTable()` baut `Table` und `Row` aus einem `sql::ResultSet`, `Row` liest ueber `m_columnNames` auf Felder zu, und die Statement-Typen werden spaeter direkt in SQL-Strings und Parameterbindungen ueberfuehrt.

## Einzelanalyse der Klassen und Strukturen

### Kritisch: Inkonsistente Rueckgabetypen bei `Row::getRowID`
Betroffene Stelle:
`maria::Row::getRowID`

Problem:
Die Deklaration in der Klasse verspricht `std::string const &`, die Definition liefert jedoch `std::string` by value. Das ist ein echter Widerspruch zwischen Schnittstelle und Implementierung.

Auswirkung:
Der Code ist strukturell inkonsistent und vom Compiler bereits als Fehler erkennbar. Zusaetzlich ist die Semantik unklar: Der Member `m_rowID` wirkt wie ein langlebiger Identifikator, die Implementierung kopiert aber nur einen Wert heraus.

Empfehlung:
Deklaration und Definition muessen denselben Typ haben. Wenn der Member intern bleiben soll, ist eine Rueckgabe als `const &` naheliegend; wenn eine Kopie gewollt ist, muss die Deklaration angepasst werden.

### Kritisch: Metadaten werden vor der Nullpruefung verwendet
Betroffene Stelle:
`maria::MariaDbConnector::getTable`

Problem:
`rs->getMetaData()` wird direkt benutzt, bevor `meta` auf Null geprueft wird. Erst spaeter folgt `if (!meta)`, also nach bereits erfolgten Zugriffen.

Auswirkung:
Falls keine Metadaten geliefert werden, kann dies zu einem Absturz oder undefiniertem Verhalten fuehren, bevor die Fehlerbehandlung greifen kann.

Empfehlung:
Den Nullcheck unmittelbar nach `getMetaData()` platzieren und erst danach alle Zugriffe auf `meta` ausfuehren.

### Hoch: Nullzustand und Typzustand von `tools::Field` widersprechen sich
Betroffene Stelle:
`tools::Field::Field()`, `tools::Field::m_type`, `tools::Field::m_isNull`, `tools::Field::getType()`

Problem:
Der Default-Konstruktor soll laut Kommentar ein Null-Field erzeugen, aber `m_type` ist trotzdem auf `DataType::String` voreingestellt. Ein Null-Field meldet damit weiterhin den Typ `String`.

Auswirkung:
Code kann einen echten Nullzustand als String missverstehen. Das fuehrt zu inkonsistenten Entscheidungen, wenn zuerst der Typ und erst danach der Nullzustand geprueft wird.

Empfehlung:
Ein expliziter Null-Typ oder ein konsistentes Variant-/Optional-Modell waere robuster. Der Nullzustand und der Typzustand sollten sich nicht gegenseitig widersprechen.

### Hoch: Schema wird doppelt und mutabel gehalten
Betroffene Stelle:
`maria::Table::ColumnNames`, `maria::Row::m_columnNames`, `maria::MariaDbConnector::getTable`

Problem:
Die Spaltennamen werden sowohl in `Table::ColumnNames` als auch in jeder `Row` ueber denselben `shared_ptr` gespeichert. Damit existiert Schema-Information doppelt und zusaetzlich mutabel.

Auswirkung:
Wenn die Spaltennamen nach dem Aufbau veraendert werden, koennen Feldzugriffe und Schema auseinanderlaufen. Die Struktur ist dadurch leicht inkonsistent zu machen.

Empfehlung:
Das Schema sollte immutable gemacht werden, zum Beispiel als `shared_ptr<const std::vector<std::string>>`, oder zentral nur an einer Stelle gehalten werden.

### Hoch: `Row` haengt an einem nicht abgesicherten Pointer
Betroffene Stelle:
`maria::Row::Row(PColumnNames)`, `maria::Row::getColumnIndexByName`, `maria::Row::getField`, `maria::Row::get*`

Problem:
`Row` nimmt einen `shared_ptr` auf Spaltennamen entgegen, prueft aber nicht auf Null. Alle Zugriffe dereferenzieren `m_columnNames` direkt.

Auswirkung:
Ein leerer oder ungueltiger Pointer fuehrt zu Absturzrisiken. Die Klasse dokumentiert den benoetigten Besitz- und Lebenszeit-Zustand nicht selbst.

Empfehlung:
Den Konstruktor auf eine nicht-null, lesbare Schema-Referenz umstellen oder mindestens eine harte Vorbedingung im Konstruktor erzwingen.

### Hoch: SQL-Identifier und WHERE-Bedingungen sind nicht sauber modelliert
Betroffene Stelle:
`maria::InsertStatement::into`, `maria::InsertStatement::set`, `maria::UpdateStatement::table`, `maria::UpdateStatement::set`, `maria::UpdateStatement::where`

Problem:
Identifier werden nur auf Leerheit und Laenge geprueft, nicht auf erlaubte Zeichen oder korrektes Quoting. Besonders `where()` uebernimmt freien SQL-Text ohne Struktur oder Absicherung.

Auswirkung:
Je nach Quelle der Eingaben entstehen malformed SQL oder Injektionsrisiken. Selbst bei internem Gebrauch bleibt die Schnittstelle missverstaendlich, weil Identifikatoren und rohe SQL-Fragmente gleich behandelt werden.

Empfehlung:
Identifier separat validieren oder quoten. Fuer `where()` sollte klar sein, ob bewusst roher SQL-Text erlaubt ist oder ob eine strukturierte Bedingung vorgesehen ist.

### Hoch: Typmodell der Statement-Typen ist enger als das Wertmodell
Betroffene Stelle:
`maria::InsertStatement`, `maria::UpdateStatement`, `MariaDbConnector::Insert(...)`, `MariaDbConnector::Update(...)`, `tools::Field`

Problem:
`tools::Field` unterstuetzt mehrere Typen, aber `InsertStatement` und `UpdateStatement` koennen nur `std::string` und `int` speichern. Der Connector behandelt spaeter ebenfalls nur diese beiden Typen.

Auswirkung:
Die API sieht allgemeiner aus, als sie ist. Erweiterungen wie bool, float, double, unsigned oder DateTime sind im Wertmodell schon angelegt, werden im Schreibpfad aber nicht abgebildet.

Empfehlung:
Entweder das Modell bewusst auf die wirklich unterstuetzten Typen reduzieren oder die Statement- und Binding-Seite vollstaendig an `tools::Field` angleichen.

### Mittel: `m_rowID` ist vorhanden, aber ohne klare Verantwortlichkeit
Betroffene Stelle:
`maria::Row::m_rowID`, `maria::Row::getRowID`, `maria::MariaDbConnector::getTable`

Problem:
`Row` besitzt einen Member `m_rowID` und einen Getter, aber beim Aufbau aus dem ResultSet wird dieser Wert nie befuellt.

Auswirkung:
Der Member suggeriert eine fachliche Identitaet, die tatsaechlich nicht genutzt wird. Das ist missverstaendlich und kann spaeter zu falschen Annahmen fuehren.

Empfehlung:
Entweder den Member entfernen oder seine Quelle und Semantik eindeutig definieren und beim Aufbau konsequent fuellen.

### Mittel: Doppelte Spaltennamen werden technisch geloest, fachlich aber nur teilweise
Betroffene Stelle:
`maria::MariaDbConnector::getTable` mit `addColumnName`

Problem:
Bei doppelten Spaltennamen werden Suffixe wie `_1` angehaengt. Dadurch werden Kollisionen vermieden, aber die Beziehung zum Originalnamen geht verloren.

Auswirkung:
Die resultierenden Namen sind eindeutig, entsprechen aber nicht mehr unmittelbar dem SQL-Resultset. Bei Joins kann das missverstaendlich werden.

Empfehlung:
Die Umbenennung dokumentieren oder zusaetzlich eine Abbildung von Original- zu Anzeigename speichern.

### Mittel: `Table` ist sehr offen und dadurch leicht missbrauchbar
Betroffene Stelle:
`maria::Table::ColumnNames`, `maria::Table::Rows`

Problem:
`Table` ist als `struct` mit oeffentlichen Membern ausgelegt. Es gibt keine Kapselung der Invarianten.

Auswirkung:
Externer Code kann die Struktur leicht in ungueltige oder inkonsistente Zustaende versetzen.

Empfehlung:
Wenn die Struktur stabil bleiben soll, sollten die Daten zumindest teilweise gekapselt oder klarer geregelt werden.

### Niedrig: Benennung und Rollen sind nicht durchgaengig eindeutig
Betroffene Stelle:
`InsertStatement::m_Values`, `UpdateStatement::m_Values`, `ConnectionConfig`, `Table::ColumnNames`

Problem:
Einige Membernamen sind fachlich brauchbar, aber nicht durchgaengig praezise. Besonders `m_Values` sagt wenig darueber aus, dass es sich um Paare aus Spaltennamen und `Field` handelt.

Auswirkung:
Die Lesbarkeit leidet, und die Verantwortlichkeit des Members wird erst durch Kontext klar.

Empfehlung:
Praezisere Namen wie `m_columnsAndValues` oder eine eigene Tupel-/Strukturart wuerden die Rolle klarer machen.

## Zusammenspiel aller Bestandteile
Das Zusammenspiel ist grundsaetzlich nachvollziehbar, aber an mehreren Stellen zu locker gekuppelt. `MariaDbConnector` erzeugt `Table` und `Row`, waehrend `Row` von einem gemeinsam genutzten Schema abhängt. Das erschwert die Sicherung von Invarianten. Gleichzeitig sind die Schreibpfade weniger flexibel als das vorhandene Wertmodell. Dadurch entsteht eine Struktur, die nach aussen allgemein wirkt, intern aber nur einen Teil der moeglichen Faelle tatsaechlich korrekt abbildet.

Die groessten Abhaengigkeiten mit Inkonsistenzrisiko sind:
- `Field`-Nullzustand versus Typzustand
- `Table::ColumnNames` versus `Row::m_columnNames`
- SQL-Identifier versus freie SQL-Fragmente
- Statement-Builder versus unterstuetzte Feldtypen

## Zusammenfassung der wichtigsten Probleme
Die schwerwiegendsten Punkte sind die echte Signaturinkonsistenz bei `Row::getRowID`, der zu spaet gesetzte Nullcheck in `getTable()` und das widerspruechliche Zustandsmodell von `tools::Field`. Hinzu kommen strukturelle Risiken durch doppelt gehaltene Spaltennamen, fehlende Absicherung des `Row`-Schemas und eine uneinheitliche Behandlung von SQL-Identifiern und Parametern.

## Empfohlene Reihenfolge der Korrekturen
1. Signatur von `Row::getRowID` bereinigen.
2. Nullpruefung fuer `meta` in `getTable()` vorziehen.
3. Null- und Typmodell von `tools::Field` konsistent machen.
4. Besitz und Unveraenderlichkeit der Spaltennamen festlegen.
5. `InsertStatement` und `UpdateStatement` fachlich sauberer von rohem SQL trennen.
6. Typunterstuetzung der Statement- und Bindungslogik an `tools::Field` angleichen.

## Vorschlag fuer eine konsistentere Gesamtstruktur
Eine schlankere und robustere Variante waere:
- `Table` besitzt ein unveraenderliches Schema.
- `Row` referenziert dieses Schema nur lesend.
- `Field` hat einen expliziten Nullzustand.
- `InsertStatement` und `UpdateStatement` unterscheiden streng zwischen Identifikatoren, Werten und freien SQL-Fragmenten.

Damit waeren die Daten- und Lebenszeitbeziehungen einfacher, und doppelt gespeicherte Information wuerde reduziert.

## Offene Fragen vor einer Ueberarbeitung
- Soll `m_rowID` eine fachliche Bedeutung haben oder nur ein Platzhalter sein?
- Darf `where()` bewusst rohen SQL-Text speichern?
- Muessen doppelte Spaltennamen nur technisch aufgeloest oder fachlich modelliert werden?
- Soll `Field` wirklich alle in `DataType` genannten Typen unterstuetzen?
- Darf `Table::ColumnNames` nach der Erzeugung veraendert werden?

## Bereits sinnvoll und konsistent umgesetzt
- Vor Datenbankoperationen wird mit `test_connected()` geprueft, ob eine Verbindung aktiv ist.
- Daten werden weitgehend ueber Prepared Statements gebunden.
- `Field` kapselt Typkonvertierungen und wirft bei ungueltigen Faellen statt still falsche Werte zu liefern.
- `ConnectionConfig` hat sinnvolle Defaultwerte fuer Host und Port.

## Hinweis zur Bewertung
Ein Teil der Bewertung bleibt ohne weitere fachliche Vorgaben bewusst vorsichtig formuliert. Das betrifft vor allem die Frage, ob freier SQL-Text in `where()` gewollt ist, ob `m_rowID` eine echte Domendenidentitaet darstellen soll und ob die breite Typunterstuetzung in `Field` nur vorbereitet oder bereits verbindlich genutzt werden soll.