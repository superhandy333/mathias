# mrdb – Anwenderhandbuch und API-Referenz

<small>[Entwicklung](develop.md) / [mrdb-Startseite](mrdb.md) / Anwenderhandbuch</small>

Dieses Handbuch erklärt die Einbindung und Verwendung von mrdb in eigenen C++-Programmen. Die API liegt im Namespace `mrdb` und wird über `mrdb.h` eingebunden.

<!-- TOC -->

- [Funktionsverzeichnis](#funktionsverzeichnis)
- [Überblick](#ueberblick)
- [Einbindung in ein eigenes Projekt](#lifecycle)
- [Schnelleinstieg](#schnelleinstieg)
- [Grundbegriffe](#datenmodell)
- [Referenz der öffentlichen API](#api)
  - [Hinweise zu den Beispielen](#api-regeln)
  - [Öffentliche Typen](#typen)
  - [Initialisierung und Tabellen](#tabellen)
  - [Schreiben und Lesen von Datensätzen](#datensaetze)
  - [Record](#record)
  - [Filter und WhereVector](#filter)
- [Typische Anwendungsfälle](#praxisbeispiele)
- [Fehlerbehandlung](#fehler)

<!-- /TOC -->

<a id="funktionsverzeichnis"></a>

## Funktionsverzeichnis der öffentlichen API

Die Links führen direkt zu den Detailbeschreibungen. Überladungen sind einzeln aufgeführt; ihre Parametertypen unterscheiden die Signaturen. Das Verzeichnis umfasst die aufrufbaren öffentlichen Funktionen einschließlich Konstruktoren, Destruktoren und Operatoren. Nicht aufrufbare Kopier- und Verschiebeoperationen werden bei den betreffenden Typen erläutert.

### Initialisierung und Tabellen

- [`Init(data)`](#funktion-init)
- [`CreateTable(table)`](#funktion-createtable)
- [`GetTable(tablename)`](#funktion-gettable)
- [`GetTableNames()`](#funktion-gettablenames)
- [`GetTables()`](#funktion-gettables)
- [`IsTableExist(tablename)`](#funktion-istableexist)
- [`GetRecordIDs(tablename)`](#funktion-getrecordids)
- [`SetTableReadOnly(tablename, readonly)`](#funktion-settablereadonly)

### Datensätze

- [`Insert(tablename, values)`](#funktion-insert)
- [`Update(std::string const & tablename, unsigned int recordID, std::map<std::string, std::string> const & values)`](#funktion-update-werte)
- [`Update(Record const & record)`](#funktion-update-record)
- [`Delete(tablename, recordID)`](#funktion-delete)
- [`SelectSingle(tablename, recordID)`](#funktion-selectsingle)
- [`SelectAll(tablename)`](#funktion-selectall)
- [`Select(tablename, whereVector)`](#funktion-select)
- [`isValidRecord(record)`](#funktion-isvalidrecord)

### KeyValue

- [`KeyValue::KeyValue(k, v)`](#keyvalue-konstruktor)

### Methoden von Record

- [`Record::Record(unsigned int rowID, std::vector<std::string> const & values, TablePtr tablePtr)`](#record-konstruktor)
- [`Record::Record(Record&&)`](#record-move-konstruktor)
- [`Record::~Record()`](#record-destruktor)
- [`Record::operator=(Record&&)`](#record-move-zuweisung)
- [`Record::GetRowID()`](#record-getrowid)
- [`Record::GetRowIDAsString()`](#record-getrowidasstring)
- [`Record::GetValue(int index) const`](#record-getvalue-index)
- [`Record::GetValue(std::string const & columnName) const`](#record-getvalue-name)
- [`Record::GetTable()`](#record-gettable)
- [`Record::GetValueSize()`](#record-getvaluesize)
- [`Record::GetValuesAsMap()`](#record-getvaluesasmap)
- [`Record::GetValuesAsKeyValues()`](#record-getvaluesaskeyvalues)
- [`Record::SetValue(columnName, value)`](#record-setvalue)
- [`Record::begin()`](#record-begin)
- [`Record::end()`](#record-end)

### WhereItem

- [`WhereItem::operator==(rhs)`](#whereitem-gleichheit)

### WhereVector

- [`WhereVector::WhereVector()`](#wherevector-konstruktor)
- [`WhereVector::~WhereVector()`](#wherevector-destruktor)
- [`WhereVector::Add(std::string const & fieldname, WhereOperator const whereOperator, std::string const & value)`](#wherevector-add-operator)
- [`WhereVector::Add(std::string const & fieldname, std::string const & value)`](#wherevector-add-gleichheit)
- [`WhereVector::begin()`](#wherevector-begin)
- [`WhereVector::end()`](#wherevector-end)

<a id="ueberblick"></a>

## Überblick

mrdb (Mathias-Rentsch-Daten-Bank) speichert Tabellen mit benannten Spalten und Datensätzen in Textdateien. Es eignet sich beispielsweise für kleine Projektlisten, Inventare oder Werkzeugverwaltungen. Alle Feldwerte sind `std::string`; Zahlen und andere fachliche Datentypen wandelt das eigene Programm bei Bedarf um.

Die Bibliothek bietet Funktionen zum Anlegen und Auflisten von Tabellen sowie zum Einfügen, Ändern, Löschen und Abfragen von Datensätzen. Filter unterstützen Gleichheit, Ungleichheit und die Suche nach einem Teiltext. Ein externer Datenbankserver ist nicht erforderlich.

<a id="lifecycle"></a>

## Einbindung in ein eigenes Projekt

Binde den öffentlichen Header ein:

```cpp
#include "mrdb.h"
```

`mrdb.h` ist Header-only: Eine zusätzliche `mrdb.cpp` muss weder kompiliert noch gelinkt werden. Die mitgelieferten Dateien `wherevector.h` und `tools.h` müssen ebenfalls verfügbar sein; sie werden vom Header eingebunden. Zusätzlich muss **`tools.cpp`** mit dem eigenen Programm kompiliert und gelinkt werden, da es die benötigten Hilfsfunktionen bereitstellt. Behalte die mitgelieferte Verzeichnisanordnung der Header bei und ergänze das Verzeichnis von `mrdb.h` im Include-Suchpfad.

Die Beispiele verwenden C++20 und die C++-Standardbibliothek einschließlich `std::filesystem`. Die API verwendet unter anderem Sprachmittel aus C++17. Weitere externe Bibliotheken sind für die hier beschriebene Nutzung nicht erforderlich.

Lege vor der Verwendung ein eigenes Datenverzeichnis an und rufe `mrdb::Init` auf. Der Pfad muss nicht leer sein und mit `/` enden. `Init` legt kein Verzeichnis an und prüft den Pfad nicht:

```cpp
std::filesystem::create_directories("./projektdaten");
mrdb::Init({"./projektdaten/"});
```

Die Initialisierung gilt für alle folgenden Datenzugriffe im Programm. Ein erneuter Aufruf von `Init` wechselt die verwendete Datenbasis. Speichere zuvor gelesene Records deshalb vor einem solchen Wechsel: `Update(record)` verwendet die dann aktive Datenbasis.

<a id="schnelleinstieg"></a>

## Schnelleinstieg: Projekte schreiben und lesen

Dieses vollständige Programm legt bei Bedarf eine Projekttabelle an, schreibt ein Projekt, ändert dessen Status und liest die Daten wieder. Bei wiederholtem Start wird ein weiteres Projekt eingefügt. Eine bereits vorhandene Tabelle muss das gezeigte Schema besitzen und schreibbar sein.

```cpp
#include "mrdb.h"

#include <exception>
#include <filesystem>
#include <iostream>

auto main() -> int
{
    try {
        std::filesystem::create_directories("./projektdaten");
        mrdb::Init({"./projektdaten/"});

        if (!mrdb::IsTableExist("PROJEKTE")) {
            auto table = mrdb::Table{};
            table.Name = "PROJEKTE";
            table.Columns = {{"01", "NAME", false}, {"02", "STATUS", true}};
            mrdb::CreateTable(table);
        }

        mrdb::Insert("PROJEKTE", {{"NAME", "Werkstattumbau"}, {"STATUS", "Geplant"}});

        auto filter = mrdb::WhereVector{};
        filter.Add("NAME", "Werkstattumbau");
        filter.Add("STATUS", "Geplant");
        auto projects = mrdb::Select("PROJEKTE", filter);
        for (auto & project : projects) {
            project.SetValue("STATUS", "In Arbeit");
            mrdb::Update(project);
        }

        auto const allProjects = mrdb::SelectAll("PROJEKTE");
        for (auto const & project : allProjects) {
            std::cout << project.GetRowID() << ": "
                      << project.GetValue("NAME") << " – "
                      << project.GetValue("STATUS") << '\n';
        }
        return 0;
    }
    catch (std::exception const & error) {
        std::cerr << error.what() << '\n';
        return 1;
    }
}
```

`Insert` gibt keine neue ID zurück. Mit `Select` erhältst du passende Datensätze und kannst deren IDs über `GetRowID()` lesen. Ein Name wie `Werkstattumbau` ist kein eindeutiger Schlüssel: Der Filter kann mehrere Projekte liefern und das Beispiel ändert alle passenden Projekte.

<a id="datenmodell"></a>

## Grundbegriffe

| Begriff | Bedeutung für die Verwendung |
| --- | --- |
| Datenbasis | Das mit `Init` gewählte Datenverzeichnis. |
| Tabelle (`Table`) | Beschreibung eines Tabellennamens, der Spalten und des Schreibschutzes. |
| Spalte (`Column`) | Ein benanntes Feld mit Sortierindex und Angabe, ob leere Werte erlaubt sind. |
| Datensatz (`Record`) | Eine Zeile mit numerischer ID, Feldwerten und Tabellenbeschreibung. |
| Feldwert | Ein Textwert vom Typ `std::string`. Es gibt keinen eigenen NULL-Typ. |
| Zeilen-ID | Die von mrdb verwendete `unsigned int`-Kennung einer Zeile; unabhängig von `Table::ID`. |
| Filter (`WhereVector`) | Eine Liste von Bedingungen, die eine ausgewählte Zeile alle erfüllen muss. |

### Namen und Werte

Verwende eindeutige Tabellen- und Spaltennamen wie `PROJEKTE`, `NAME` und `STATUS`. Namen dürfen keine Pfade sein. Unzulässig sind insbesondere leere Namen, `.` und `..`, reservierte Windows-Namen wie `CON`, Steuerzeichen und die Zeichen `<>:"/\|?*`. Führende Leerzeichen, abschließende Punkte oder Leerzeichen und Namen über 255 Bytes werden ebenfalls abgewiesen; das Dateisystem kann weitere Grenzen setzen.

Beim Anlegen werden Spaltennamen in Großbuchstaben gespeichert. `Insert` und `Update(tablename, recordID, values)` akzeptieren Spaltenschlüssel unabhängig von ihrer Groß-/Kleinschreibung. Verwende in einer Map dennoch nur einen Schlüssel pro Spalte. `Record::GetValue`, `Record::SetValue` und Filter benötigen dagegen den **exakten** Spaltennamen aus der gelesenen Tabellenbeschreibung.

Für `Nullable == false` muss `Insert` einen Wert angeben, der weder leer ist noch ausschließlich aus Leerzeichen besteht. Nullable-Spalten dürfen bei `Insert` fehlen; sie sind dann leer. Leere oder nur aus Leerzeichen bestehende Feldinhalte werden als leerer String gelesen. Tabs und Zeilenumbrüche zählen bei dieser Leerprüfung nicht als Leerzeichen.

### Spaltenreihenfolge und IDs

`SortIndex` ist ein String und bestimmt die Spaltenreihenfolge lexikographisch: `"10"` steht vor `"2"`. Verwende beispielsweise `"01"`, `"02"`, `"10"` für eine numerisch gedachte Reihenfolge. Bei gleichen Sortierindizes solltest du dich nicht auf eine bestimmte Reihenfolge verlassen. `GetValue(int)` und die Wertiteration folgen der Spaltenreihenfolge der gelesenen Tabelle.

Tabellen- und Datensatzlisten sind nicht nach Namen oder IDs sortiert. Eine neue Zeilen-ID beginnt bei eins beziehungsweise folgt auf die höchste vorhandene ID. Nach dem Löschen können IDs wiederverwendet werden; verwende sie daher nicht als dauerhaft eindeutige Kennungen über die Lebensdauer gelöschter Zeilen hinaus.

### Gelesene Werte und Änderungen

`Table` und `Record` sind gelesene Werte. Spätere Datenbankänderungen aktualisieren sie nicht automatisch. `Record::SetValue` ändert nur den lokalen Datensatz; erst `Update(record)` speichert ihn. Die Zerstörung eines Records speichert keine Änderungen und löscht keine Zeile.

Strings, Maps und Vektoren werden als eigene Werte zurückgegeben und automatisch freigegeben. `Record::GetTable()` gibt einen `std::shared_ptr<Table>` zurück: Die Beschreibung bleibt erhalten, solange mindestens ein solcher Besitzer existiert. Verändere die Spalten dieser Beschreibung nicht, da sonst ihre Zuordnung zu den Record-Werten verloren geht.

<a id="api"></a>

## Referenz der öffentlichen API

<a id="api-regeln"></a>

### Hinweise zu den Beispielen

Alle Signaturen stehen im Namespace `mrdb`; Member-Signaturen stehen im Abschnitt ihres Typs. Bei `begin()` und `end()` wird der deduzierte Rückgabetyp zur besseren Lesbarkeit ausgeschrieben. C++-Referenzparameter benötigen ein gültiges Objekt und können nicht `nullptr` sein. Die einzige Zeigereingabe ist `TablePtr` im Record-Konstruktor; ihre Verwendung wird dort erklärt.

Die kurzen Beispiele sind eigenständige Ausschnitte innerhalb einer Funktion. Für Datenzugriffe setzen sie eine initialisierte Datenbasis sowie die schreibbare Tabelle `PROJEKTE` mit den Spalten `NAME` und `STATUS` aus dem Schnelleinstieg voraus. Für Zugriffe auf eine einzelne Zeile dient folgende Vorbereitung:

```cpp
auto const tableName = std::string{"PROJEKTE"};
auto const ids = mrdb::GetRecordIDs(tableName);
if (ids.empty()) {
    throw std::runtime_error("Das Beispiel benötigt ein vorhandenes Projekt");
}
auto const recordId = ids.front();
auto record = mrdb::SelectSingle(tableName, recordId);
```

Zusätzlich zu `mrdb.h` verwenden die Ausschnitte je nach Beispiel `<iostream>`, `<memory>`, `<stdexcept>`, `<string>`, `<utility>` und `<vector>`. Die vollständigen Programme nennen ihre Includes selbst.

Bei Datenzugriffen werden fehlende Tabellen, fehlende Zeilen, unzulässige Werte und Schreibschutz durch Exceptions gemeldet. Fehler beim Dateizugriff können ebenfalls Exceptions auslösen. Ein `void`-Aufruf liefert keinen Statuscode; normale Rückkehr bedeutet, dass kein Fehler gemeldet wurde. `IsTableExist` fasst auch Fehler als `false` zusammen. Weitere Beispiele stehen unter [Fehlerbehandlung](#fehler).

<a id="typen"></a>

### Öffentliche Typen

<a id="typ-initdata"></a>

#### InitData

```cpp
struct InitData
{
    std::string NAME_DATABASE_DIRECTORY{};
};
```

Konfiguration für `Init`. Der String enthält den eigenen, vorhandenen Datenpfad mit abschließendem `/`; standardmäßig ist er leer.

```cpp
auto data = mrdb::InitData{};
data.NAME_DATABASE_DIRECTORY = "./projektdaten/";
mrdb::Init(data);
```

<a id="typ-column"></a>

#### Column

```cpp
struct Column
{
    std::string SortIndex{};
    std::string Name{};
    bool Nullable{true};
};
```

`SortIndex` bestimmt die Reihenfolge als Text, `Name` benennt die Spalte, `Nullable` erlaubt leere Werte. Die Standardwerte sind leere Strings und `true`.

```cpp
auto const nameColumn = mrdb::Column{"01", "NAME", false};
auto const statusColumn = mrdb::Column{"02", "STATUS", true};
```

<a id="typ-table"></a>

#### Table und TablePtr

```cpp
struct Table
{
    unsigned int ID{0};
    std::string Name{};
    bool ReadOnly{false};
    std::vector<Column> Columns{};
};
using TablePtr = std::shared_ptr<Table>;
```

`Name` und `Columns` beschreiben die Tabelle. `ReadOnly` sperrt das Schreiben von Datensätzen über `Insert`, `Update` und `Delete`; mit `SetTableReadOnly` lässt sich die Sperre setzen und aufheben. `ID` ist nicht die Zeilen-ID und wird von `CreateTable` nicht aus dem Eingabeobjekt übernommen. Änderungen an einem `Table`-Wert werden nicht automatisch gespeichert.

```cpp
auto table = mrdb::Table{};
table.Name = "PROJEKTE";
table.Columns = {{"01", "NAME", false}, {"02", "STATUS", true}};
auto tablePtr = std::make_shared<mrdb::Table>(table);
```

<a id="typ-keyvalue"></a>

#### KeyValue und KeyValues

```cpp
struct KeyValue
{
    std::string Key{};
    std::string Value{};
    KeyValue(std::string const & k, std::string const & v);
};
using KeyValues = std::vector<KeyValue>;
```

Ein Paar aus Spaltenname (`Key`) und Feldwert (`Value`). `KeyValues` ist eine Liste solcher Paare, etwa das Ergebnis von `Record::GetValuesAsKeyValues()`.

<a id="keyvalue-konstruktor"></a>

#### KeyValue::KeyValue(k, v)

```cpp
KeyValue(std::string const & k, std::string const & v);
```

Erzeugt ein Schlüssel/Wert-Paar mit eigenen Kopien der beiden Strings.

- **Parameter:** `k`: Schlüssel; `v`: Wert. Beide dürfen leer sein.
- **Rückgabe:** Kein Rückgabewert (Konstruktor). Die Felder `Key` und `Value` enthalten die übergebenen Texte.
- **Voraussetzungen:** Keine Initialisierung oder vorhandene Tabelle erforderlich.

```cpp
auto const pair = mrdb::KeyValue{"STATUS", "Geplant"};
std::cout << pair.Key << ": " << pair.Value << '\n';
```

<a id="typ-records"></a>

#### Records

```cpp
using Records = std::vector<Record>;
```

Ergebnisliste von `SelectAll` und `Select`. Da `Record` nicht kopierbar ist, ist auch das Kopieren einer solchen Liste nicht möglich. Iteriere über Referenzen oder verschiebe die Liste.

```cpp
auto records = mrdb::SelectAll("PROJEKTE");
auto transferred = std::move(records);
for (auto const & project : transferred) {
    std::cout << project.GetRowID() << '\n';
}
```

`InitData`, `Column`, `Table`, `KeyValue` und `WhereItem` können als Werte kopiert und verschoben werden. Für sie ist keine besondere Speicherfreigabe nötig. `Record` und die Filtertypen werden mit ihren Methoden weiter unten beschrieben.

<a id="tabellen"></a>

### Initialisierung und Tabellen

<a id="funktion-init"></a>

#### Init(data)

```cpp
void Init(InitData const & data);
```

Wählt die Datenbasis für folgende Datenzugriffe.

- **Parameter:** `data`: Konfiguration mit einem nicht leeren Datenpfad und abschließendem `/`.
- **Rückgabe:** Keiner. Die Konfiguration wird kopiert; das Eingabeobjekt muss danach nicht weiterleben.
- **Voraussetzungen:** Für folgende Datenzugriffe muss das Verzeichnis vorhanden und zugänglich sein. `Init` prüft dies nicht und legt kein Verzeichnis an.

```cpp
mrdb::Init({"./projektdaten/"});
```

<a id="funktion-createtable"></a>

#### CreateTable(table)

```cpp
void CreateTable(Table const & table);
```

Legt eine neue Tabelle mit den angegebenen Spalten und dem Schreibschutz an.

- **Parameter:** `table`: gültiger `Name`, mindestens eine Spalte, gültige und auch nach Großschreibung eindeutige Spaltennamen. `SortIndex` bestimmt die Spaltenordnung, `Nullable` die Zulässigkeit leerer Werte. `ReadOnly` setzt den Schreibschutz. `ID` wird nicht übernommen.
- **Rückgabe:** Keiner. Das Eingabeobjekt bleibt unverändert. Bei Erfolg wird außerdem eine Meldung ausgegeben.
- **Voraussetzungen:** Initialisierte Datenbasis mit Schreibzugriff; der Tabellenname darf noch nicht vorhanden sein.
- **Fehler:** `std::runtime_error` beispielsweise bei vorhandenem Namen, fehlenden Spalten oder unzulässigen Namen; außerdem Dateizugriffsfehler.

```cpp
auto table = mrdb::Table{};
table.Name = "ARCHIVPROJEKTE";
table.Columns = {{"01", "NAME", false}, {"02", "STATUS", true}};
mrdb::CreateTable(table);
```

<a id="funktion-gettable"></a>

#### GetTable(tablename)

```cpp
auto GetTable(std::string const & tablename) -> Table;
```

Liest die Beschreibung einer Tabelle.

- **Parameter:** `tablename`: gültiger Name einer vorhandenen Tabelle.
- **Rückgabe:** Ein eigener `Table`-Wert mit Spalten, ID und Schreibschutz. Die Spalten sind nach `SortIndex` geordnet. Änderungen am Ergebnis speichern nichts.
- **Voraussetzungen:** Initialisierte Datenbasis und lesbare Tabelle.
- **Fehler:** Eine fehlende oder nicht lesbare Tabelle löst eine Exception aus; es gibt keinen leeren Ersatzwert.

```cpp
auto const table = mrdb::GetTable(tableName);
std::cout << table.Name << ": " << table.Columns.size() << '\n';
```

<a id="funktion-gettablenames"></a>

#### GetTableNames()

```cpp
auto GetTableNames() -> std::vector<std::string>;
```

Listet die Tabellennamen in der Datenbasis auf.

- **Parameter:** Keine.
- **Rückgabe:** Eigener Vektor von Namen, ohne zugesicherte Sortierung; leer, wenn keine vorhanden sind.
- **Voraussetzungen:** Initialisierte, zugängliche Datenbasis. Verwende ein eigenes Datenverzeichnis: Fremde Unterverzeichnisse können ebenfalls als Namen erscheinen; die Auflistung prüft kein Schema.

```cpp
auto const names = mrdb::GetTableNames();
for (auto const & name : names) {
    std::cout << name << '\n';
}
```

<a id="funktion-gettables"></a>

#### GetTables()

```cpp
auto GetTables() -> std::vector<Table>;
```

Liest die Beschreibungen aller aufgelisteten Tabellen.

- **Parameter:** Keine.
- **Rückgabe:** Eigener Vektor von `Table`-Werten; leer ohne Tabellen und ohne zugesicherte Reihenfolge.
- **Voraussetzungen:** Initialisierte Datenbasis; alle aufgelisteten Tabellen müssen lesbar sein. Fremde Unterverzeichnisse können den Aufruf scheitern lassen.
- **Fehler:** Ist eine Tabelle nicht lesbar, wird eine Exception ausgelöst und kein Teilergebnis zurückgegeben.

```cpp
auto const tables = mrdb::GetTables();
for (auto const & table : tables) {
    std::cout << table.Name << ": " << table.Columns.size() << '\n';
}
```

<a id="funktion-istableexist"></a>

#### IsTableExist(tablename)

```cpp
auto IsTableExist(std::string const & tablename) -> bool;
```

Prüft, ob eine Tabelle unter dem angegebenen Namen vorhanden ist.

- **Parameter:** `tablename`: der gesuchte Tabellenname.
- **Rückgabe:** `true`, wenn ein entsprechendes Verzeichnis vorhanden ist; `false` bei Nichtvorhandensein, ungültigem Namen, fehlender Initialisierung oder Zugriffsfehlern. `true` bestätigt nicht die Lesbarkeit oder Vollständigkeit einer Tabelle.
- **Voraussetzungen:** Für eine aussagekräftige Prüfung zuerst die Datenbasis initialisieren.

```cpp
if (mrdb::IsTableExist("PROJEKTE")) {
    auto const table = mrdb::GetTable("PROJEKTE");
    std::cout << table.Name << '\n';
}
```

<a id="funktion-getrecordids"></a>

#### GetRecordIDs(tablename)

```cpp
auto GetRecordIDs(std::string const & tablename) -> std::vector<unsigned int>;
```

Liefert die IDs der vorhandenen Datensätze einer Tabelle.

- **Parameter:** `tablename`: Name einer vorhandenen Tabelle.
- **Rückgabe:** Eigener Vektor von IDs; leer ohne Datensätze. Die IDs sind nicht zwingend sortiert.
- **Voraussetzungen:** Initialisierte Datenbasis und zugängliche Tabelle.

```cpp
auto const recordIds = mrdb::GetRecordIDs(tableName);
for (auto const id : recordIds) {
    std::cout << id << '\n';
}
```

<a id="funktion-settablereadonly"></a>

#### SetTableReadOnly(tablename, readonly)

```cpp
void SetTableReadOnly(std::string const & tablename, bool readonly);
```

Setzt oder entfernt den Schreibschutz für Datensatzänderungen.

- **Parameter:** `tablename`: vorhandene Tabelle; `readonly`: `true` sperrt `Insert`, `Update` und `Delete`, `false` erlaubt sie wieder.
- **Rückgabe:** Keiner. Bereits gelesene Tabellenbeschreibungen werden nicht automatisch aktualisiert.
- **Voraussetzungen:** Initialisierte Datenbasis und Schreibzugriff auf die Tabellenbeschreibung. Der Aufruf ist auch bei bereits gesetztem Schreibschutz zulässig.

```cpp
mrdb::SetTableReadOnly(tableName, true);
auto const projects = mrdb::SelectAll(tableName); // Lesen bleibt möglich.
mrdb::SetTableReadOnly(tableName, false);
```

<a id="datensaetze"></a>

### Schreiben und Lesen von Datensätzen

<a id="funktion-insert"></a>

#### Insert(tablename, values)

```cpp
void Insert(std::string const & tablename,
            std::map<std::string, std::string> const & values);
```

Fügt einen Datensatz ein.

- **Parameter:** `tablename`: vorhandene Tabelle. `values`: nicht leere Zuordnung bekannter Spaltennamen zu Textwerten. Alle Pflichtspalten müssen angegeben und weder leer noch ausschließlich Leerzeichen sein. Nullable-Spalten dürfen fehlen.
- **Rückgabe:** Keiner; insbesondere keine ID und kein Zeilenzähler. Ausgelassene Nullable-Spalten sind leer.
- **Voraussetzungen:** Initialisierte Datenbasis, vorhandene schreibbare Tabelle und Schreibzugriff.
- **Fehler:** `std::runtime_error` bei unbekannten Spalten, leeren Eingaben, fehlenden Pflichtwerten oder Schreibschutz; außerdem Dateizugriffsfehler.

```cpp
mrdb::Insert(tableName, {{"NAME", "Dachsanierung"}, {"STATUS", "Geplant"}});
```

<a id="funktion-update-werte"></a>

#### Update(std::string const & tablename, unsigned int recordID, std::map<std::string, std::string> const & values)

```cpp
void Update(std::string const & tablename, unsigned int recordID,
            std::map<std::string, std::string> const & values);
```

Ändert ausgewählte Felder eines vorhandenen Datensatzes.

- **Parameter:** `tablename`: Tabelle; `recordID`: vorhandene positive Zeilen-ID; `values`: nicht leere Map mit bekannten Spalten und zulässigen Werten. Pflichtspalten dürfen ausgelassen werden, angegebene Pflichtwerte dürfen jedoch nicht leer oder nur Leerzeichen sein.
- **Rückgabe:** Keiner. Nur angegebene Felder werden ersetzt, alle anderen bleiben erhalten.
- **Voraussetzungen:** Initialisierte Datenbasis, vorhandene Zeile und schreibbare Tabelle.
- **Fehler:** Fehlende Zeile, unzulässige Werte oder Schreibschutz führen zu `std::runtime_error`; außerdem Dateizugriffsfehler.

```cpp
mrdb::Update(tableName, recordId, {{"STATUS", "Abgeschlossen"}});
```

<a id="funktion-update-record"></a>

#### Update(Record const & record)

```cpp
void Update(Record const & record);
```

Speichert die Werte eines Records in seiner Tabelle und Zeile.

- **Parameter:** `record`: gültiger Datensatz mit Tabellenname, positiver ID und passender Anzahl von Spalten und Werten.
- **Rückgabe:** Keiner. Der Record selbst bleibt unverändert.
- **Voraussetzungen:** Initialisierte Datenbasis, vorhandene schreibbare Zielzeile und gültige Tabellenbeschreibung. Ziel sind Name und ID des Records in der aktuell gewählten Datenbasis.
- **Fehler:** Ungültiger Record, fehlende Zielzeile, unzulässige Feldwerte oder Schreibschutz lösen Exceptions aus.

```cpp
record.SetValue("STATUS", "In Arbeit");
mrdb::Update(record);
```

<a id="funktion-delete"></a>

#### Delete(tablename, recordID)

```cpp
void Delete(std::string const & tablename, unsigned int recordID);
```

Löscht einen vorhandenen Datensatz.

- **Parameter:** `tablename`: Tabelle; `recordID`: positive ID der zu löschenden Zeile.
- **Rückgabe:** Keiner. Bereits gelesene Records bleiben als lokale Werte erhalten.
- **Voraussetzungen:** Initialisierte Datenbasis, vorhandene Zeile, schreibbare Tabelle und Schreibzugriff.
- **Fehler:** Eine fehlende Zeile oder Schreibschutz wird durch `std::runtime_error` gemeldet; außerdem Dateizugriffsfehler.

```cpp
mrdb::Delete(tableName, recordId); // Eigenständiges Beispiel: entfernt diese Zeile.
```

<a id="funktion-selectsingle"></a>

#### SelectSingle(tablename, recordID)

```cpp
auto SelectSingle(std::string const & tablename, unsigned int recordID) -> Record;
```

Liest einen Datensatz anhand seiner ID.

- **Parameter:** `tablename`: Tabelle; `recordID`: vorhandene positive Zeilen-ID.
- **Rückgabe:** Eigener, bewegbarer `Record` mit allen Feldwerten und Tabellenbeschreibung. Ein fehlender Datensatz wird nicht als `nullptr` oder leerer Record zurückgegeben.
- **Voraussetzungen:** Initialisierte Datenbasis und lesbare Zielzeile. Die Tabelle darf schreibgeschützt sein.
- **Fehler:** Fehlende Tabelle, fehlende Zeile oder nicht lesbare Inhalte lösen Exceptions aus.

```cpp
auto project = mrdb::SelectSingle(tableName, recordId);
std::cout << project.GetValue("NAME") << '\n';
```

<a id="funktion-selectall"></a>

#### SelectAll(tablename)

```cpp
auto SelectAll(std::string const & tablename) -> Records;
```

Liest alle Datensätze einer Tabelle.

- **Parameter:** `tablename`: vorhandene Tabelle.
- **Rückgabe:** Eigener `Records`-Vektor, leer bei einer Tabelle ohne Zeilen. Keine zugesicherte Sortierung.
- **Voraussetzungen:** Initialisierte Datenbasis und lesbare Datensätze.
- **Fehler:** Kann eine Zeile nicht gelesen werden, wird eine Exception ausgelöst; kein Teilergebnis.

```cpp
auto const projects = mrdb::SelectAll(tableName);
for (auto const & project : projects) {
    std::cout << project.GetRowID() << ": " << project.GetValue("NAME") << '\n';
}
```

<a id="funktion-select"></a>

#### Select(tablename, whereVector)

```cpp
auto Select(std::string const & tablename, WhereVector const & whereVector) -> Records;
```

Wählt die Datensätze aus, die alle Filterbedingungen erfüllen (logisches UND).

- **Parameter:** `tablename`: vorhandene Tabelle. `whereVector`: Bedingungen mit exakten vorhandenen Spaltennamen, den Operatoren `Equal`, `NonEqual` oder `Like` und Textwerten. Ein leerer Filter wählt alle Zeilen aus.
- **Rückgabe:** Eigener `Records`-Vektor, leer ohne Treffer. Keine zugesicherte Sortierung.
- **Voraussetzungen:** Initialisierte Datenbasis, lesbare Datensätze und passende Filterspalten. Vergleiche beachten Groß-/Kleinschreibung. `Like` sucht einen Teiltext, ohne Platzhalter; bei leerem Suchtext oder leerem Feld gibt es keinen Treffer.
- **Fehler:** Ein unbekannter Filterfeldname kann beim Prüfen einer Zeile `std::runtime_error` auslösen. Bei einer leeren Tabelle werden Filterfelder nicht geprüft. Lesefehler lösen ebenfalls Exceptions aus.

```cpp
auto filter = mrdb::WhereVector{};
filter.Add("STATUS", "Geplant");
filter.Add("NAME", mrdb::WhereOperator::Like, "Dach");
auto const projects = mrdb::Select(tableName, filter);
std::cout << projects.size() << '\n';
```

<a id="funktion-isvalidrecord"></a>

#### isValidRecord(record)

```cpp
void isValidRecord(Record const & record);
```

Prüft, ob ein Record die grundlegenden Voraussetzungen für seine Verwendung erfüllt.

- **Parameter:** `record`: zu prüfender Record; auch ein manuell erzeugter Record mit leerem `TablePtr` darf geprüft werden.
- **Rückgabe:** Keiner und kein `bool`. Normale Rückkehr bedeutet: Tabelle vorhanden, Tabellenname nicht leer, mindestens eine Spalte, ID ungleich null, mindestens ein Wert und gleiche Anzahl von Werten und Spalten.
- **Voraussetzungen:** Keine Initialisierung erforderlich. Die Prüfung bestätigt weder eine gespeicherte Zielzeile noch die Zulässigkeit der Feldwerte für einen Schreibaufruf.
- **Fehler:** Eine verletzte Voraussetzung löst `std::runtime_error` aus.

```cpp
mrdb::isValidRecord(record);
std::cout << "Record kann weiterverwendet werden" << '\n';
```

<a id="record"></a>

### Record: Datensätze bearbeiten

`Record` ist ein nicht kopierbarer, bewegbarer Datensatz. Es gibt keinen Standardkonstruktor. Verwende zum Lesen normalerweise `SelectSingle`, `SelectAll` oder `Select`. Für eine zweite unabhängige Kopie gespeicherter Daten kannst du erneut lesen.

Nach dem Verschieben darf die Quelle zerstört oder mit einem gültigen Record neu belegt werden. Verwende ihre tabellenbezogenen Methoden erst wieder nach einer solchen Zuweisung. Beschaffe Iteratoren nach einer Verschiebung oder Ersetzung eines Records neu.

<a id="record-konstruktor"></a>

#### Record::Record(unsigned int rowID, std::vector<std::string> const & values, TablePtr tablePtr)

```cpp
Record(unsigned int rowID, std::vector<std::string> const & values, TablePtr tablePtr);
```

Erzeugt einen lokalen Record aus einer ID, einer Wertliste und einer Tabellenbeschreibung.

- **Parameter:** `rowID`: für einen gültigen Record positive Zeilen-ID. `values`: Texte in Spaltenreihenfolge, mit genau einem Wert je Spalte. `tablePtr`: geteilter Besitzer einer Tabelle mit Namen und Spalten. Der Konstruktor akzeptiert `nullptr`; vor tabellenbezogenen Zugriffen muss jedoch eine gültige Beschreibung vorhanden sein. `isValidRecord` kann einen Record mit leerem Zeiger prüfen.
- **Rückgabe:** Kein Rückgabewert (Konstruktor). Der Record besitzt eigene Wertkopien und teilt die Tabellenbeschreibung.
- **Voraussetzungen:** Keine Initialisierung nötig. Der Konstruktor prüft die Eingaben nicht und speichert keine Zeile. Für tabellenbezogene Methoden muss die Beschreibung gültig sein und zu den Werten passen.

```cpp
auto table = std::make_shared<mrdb::Table>();
table->Name = "PROJEKTE";
table->Columns = {{"01", "NAME", false}, {"02", "STATUS", true}};
auto const values = std::vector<std::string>{"Werkstattumbau", "Geplant"};
auto localRecord = mrdb::Record{1, values, table};
mrdb::isValidRecord(localRecord);
```

<a id="record-destruktor"></a>

#### Record::~Record()

```cpp
~Record() = default;
```

Gibt die lokalen Werte und den eigenen Anteil am Tabellenbesitz automatisch frei.

- **Parameter:** Keine.
- **Rückgabe:** Keiner. Die gespeicherte Zeile bleibt bestehen; lokale Änderungen werden nicht gespeichert.
- **Voraussetzungen:** Ein lebender Record; nach seiner Zerstörung dürfen seine Wertiteratoren nicht weiterverwendet werden.

```cpp
{
    auto project = mrdb::SelectSingle(tableName, recordId);
    std::cout << project.GetValue("NAME") << '\n';
} // Der Destruktor wird automatisch aufgerufen.
```

<a id="record-move-konstruktor"></a>

#### Record::Record(Record&&)

```cpp
Record(Record && source) = default;
```

Überträgt einen Record in ein neues Objekt.

- **Parameter:** `source`: zu verschiebender Record, beispielsweise mit `std::move` übergeben.
- **Rückgabe:** Kein Rückgabewert (Konstruktor). Das Ziel enthält die übertragenen Werte und Tabellenbeschreibung.
- **Voraussetzungen:** Keine Datenbankinitialisierung für die Verschiebung. Die Quelle anschließend nur zerstören oder neu zuweisen.

```cpp
auto source = mrdb::SelectSingle(tableName, recordId);
auto destination = mrdb::Record{std::move(source)};
std::cout << destination.GetRowID() << '\n';
```

<a id="record-move-zuweisung"></a>

#### Record::operator=(Record&&)

```cpp
auto operator=(Record && source) -> Record & = default;
```

Ersetzt einen vorhandenen Record durch einen anderen mittels Verschiebung.

- **Parameter:** `source`: ein anderer zu verschiebender Record. Der bisherige Zielinhalt wird ersetzt.
- **Rückgabe:** Referenz auf das Zielobjekt, gültig bis zu dessen Zerstörung.
- **Voraussetzungen:** Lebende Quell- und Zielobjekte; die Quelle anschließend nur zerstören oder neu zuweisen. Alte Zieliteratoren nicht weiterverwenden.

```cpp
auto target = mrdb::SelectSingle(tableName, recordId);
auto source = mrdb::SelectSingle(tableName, recordId);
auto & assigned = (target = std::move(source));
std::cout << assigned.GetRowID() << '\n';
```

<a id="record-getrowid"></a>

#### Record::GetRowID()

```cpp
auto GetRowID() const -> unsigned int;
```

Liefert die Zeilen-ID des Records.

- **Parameter:** Keine.
- **Rückgabe:** Die numerische ID; sie ist unabhängig von einer benannten Spalte `ID`. Bei manuell erzeugten Records kann sie null sein.
- **Voraussetzungen:** Lebender Record. Keine Initialisierung oder Tabellenbeschreibung nötig.

```cpp
auto const id = record.GetRowID();
std::cout << id << '\n';
```

<a id="record-getrowidasstring"></a>

#### Record::GetRowIDAsString()

```cpp
auto GetRowIDAsString() const -> std::string;
```

Liefert die Zeilen-ID als Text.

- **Parameter:** Keine.
- **Rückgabe:** Eigener String mit der dezimalen ID.
- **Voraussetzungen:** Lebender Record; keine Tabellenbeschreibung nötig.

```cpp
auto const idText = record.GetRowIDAsString();
std::cout << idText << '\n';
```

<a id="record-getvalue-index"></a>

#### Record::GetValue(int index) const

```cpp
auto GetValue(int index) const -> std::string;
```

Liest einen Feldwert anhand seiner Position.

- **Parameter:** `index`: ab null gezählte Position, kleiner als `GetValueSize()` und als `int` darstellbar.
- **Rückgabe:** Eigene Kopie des Feldtexts, auch leer möglich; sie bleibt unabhängig vom Record gültig.
- **Voraussetzungen:** Lebender Record mit einem Wert an dieser Position. Die Position entspricht der Spaltenreihenfolge des Records.
- **Fehler:** Ein negativer oder zu großer Index löst `std::out_of_range` aus.

```cpp
if (record.GetValueSize() > 0) {
    std::cout << record.GetValue(0) << '\n';
}
```

<a id="record-getvalue-name"></a>

#### Record::GetValue(std::string const & columnName) const

```cpp
auto GetValue(std::string const & columnName) const -> std::string;
```

Liest einen Feldwert anhand des Spaltennamens.

- **Parameter:** `columnName`: exakter vorhandener Name, beispielsweise `"NAME"`.
- **Rückgabe:** Eigene Kopie des Textwerts, auch leer möglich; unabhängig von der Lebensdauer des Records.
- **Voraussetzungen:** Gültige, nicht leere Tabellenbeschreibung mit passender Wertliste. Keine Datenbankinitialisierung für den lokalen Zugriff nötig.
- **Fehler:** Unbekannter Name: `std::invalid_argument`; fehlende Wertposition: `std::out_of_range`. Ein leerer Tabellenzeiger ist keine zulässige Eingabe für diese Methode.

```cpp
auto const name = record.GetValue("NAME");
std::cout << name << '\n';
```

<a id="record-gettable"></a>

#### Record::GetTable()

```cpp
auto GetTable() const -> TablePtr;
```

Gibt Zugriff auf die Tabellenbeschreibung des Records.

- **Parameter:** Keine.
- **Rückgabe:** Kopie des `std::shared_ptr<Table>`. Die Tabelle bleibt gültig, solange ein Besitzer existiert. Bei manuell erzeugten oder verschobenen Records kann der Zeiger leer sein; gelesene Records besitzen eine Beschreibung.
- **Voraussetzungen:** Lebender Record. Spalten der Beschreibung nicht verändern oder umordnen, da sonst die Wertzuordnung nicht mehr passt.

```cpp
auto table = record.GetTable();
if (table) {
    std::cout << table->Name << '\n';
}
```

<a id="record-getvaluesize"></a>

#### Record::GetValueSize()

```cpp
auto GetValueSize() const -> std::size_t;
```

Liefert die Anzahl der Feldwerte.

- **Parameter:** Keine.
- **Rückgabe:** Anzahl als Wert; null bei leerer Wertliste.
- **Voraussetzungen:** Lebender Record; keine Tabellenbeschreibung nötig.

```cpp
auto const count = record.GetValueSize();
std::cout << count << '\n';
```

<a id="record-getvaluesasmap"></a>

#### Record::GetValuesAsMap()

```cpp
auto GetValuesAsMap() const -> std::map<std::string, std::string>;
```

Erstellt eine Zuordnung von Spaltennamen zu Feldwerten.

- **Parameter:** Keine.
- **Rückgabe:** Eigene Map mit kopierten Namen und Werten, unabhängig vom Record. Die Map ordnet ihre Einträge nach Schlüsseln.
- **Voraussetzungen:** Nicht leere Tabellenbeschreibung und passende Anzahl von Werten. Bei selbst erzeugten Records zuerst `isValidRecord` aufrufen.

```cpp
mrdb::isValidRecord(record);
auto const values = record.GetValuesAsMap();
std::cout << values.at("NAME") << '\n';
```

<a id="record-getvaluesaskeyvalues"></a>

#### Record::GetValuesAsKeyValues()

```cpp
auto GetValuesAsKeyValues() const -> KeyValues;
```

Erstellt eine Liste aus Spaltenname/Wert-Paaren.

- **Parameter:** Keine.
- **Rückgabe:** Eigener `KeyValues`-Vektor mit kopierten Texten in der Spaltenreihenfolge des Records; unabhängig von dessen Lebensdauer.
- **Voraussetzungen:** Nicht leere Tabellenbeschreibung und passende Anzahl von Werten. Bei selbst erzeugten Records zuerst `isValidRecord` aufrufen.

```cpp
mrdb::isValidRecord(record);
auto const pairs = record.GetValuesAsKeyValues();
for (auto const & pair : pairs) {
    std::cout << pair.Key << ": " << pair.Value << '\n';
}
```

<a id="record-setvalue"></a>

#### Record::SetValue(columnName, value)

```cpp
void SetValue(std::string const & columnName, std::string const & value);
```

Ändert einen Feldwert im lokalen Record. Zum Speichern anschließend `Update(record)` verwenden.

- **Parameter:** `columnName`: exakter vorhandener Spaltenname. `value`: neuer Text, auch leer möglich. Die Zulässigkeit eines Pflichtwerts und der Schreibschutz werden erst beim Speichern geprüft.
- **Rückgabe:** Keiner. Der neue Text wird kopiert; die gespeicherte Zeile bleibt bis zu `Update` unverändert.
- **Voraussetzungen:** Nicht leere Tabellenbeschreibung und vorhandene Wertposition.
- **Fehler:** Unbekannter Name: `std::invalid_argument`; fehlende Wertposition: `std::out_of_range`. Ein leerer Tabellenzeiger ist keine zulässige Eingabe für diese Methode.

```cpp
record.SetValue("STATUS", "Abgeschlossen");
std::cout << record.GetValue("STATUS") << '\n';
mrdb::Update(record);
```

<a id="record-begin"></a>

#### Record::begin()

```cpp
auto begin() const -> std::vector<std::string>::const_iterator;
```

Liefert den Anfang der schreibgeschützten Wertiteration.

- **Parameter:** Keine.
- **Rückgabe:** Konstanter Iterator auf den ersten Wert; bei leerer Liste gleich `end()`. Er besitzt die Werte nicht.
- **Voraussetzungen:** Lebender Record. Nur dereferenzieren, wenn ungleich `end()`. Iteratoren nach Zerstörung oder Ersetzung des Records nicht verwenden.

```cpp
auto const first = record.begin();
if (first != record.end()) {
    std::cout << *first << '\n';
}
```

<a id="record-end"></a>

#### Record::end()

```cpp
auto end() const -> std::vector<std::string>::const_iterator;
```

Liefert das Ende der schreibgeschützten Wertiteration.

- **Parameter:** Keine.
- **Rückgabe:** Konstanter Iterator hinter dem letzten Wert; niemals dereferenzieren. Bei leerer Liste gleich `begin()`.
- **Voraussetzungen:** Lebender Record. Iteratoren nach Zerstörung oder Ersetzung des Records nicht verwenden. Die Iteration folgt der Spaltenreihenfolge.

```cpp
for (auto it = record.begin(); it != record.end(); ++it) {
    std::cout << *it << '\n';
}
```

<a id="filter"></a>

### Filter und WhereVector

Die folgenden Typen stehen durch `mrdb.h` zur Verfügung; ein zusätzlicher Include im Anwenderprogramm ist nicht erforderlich.

<a id="typ-whereoperator"></a>

#### WhereOperator

```cpp
enum class WhereOperator
{
    Equal,
    NonEqual,
    Like
};
```

| Wert | Bedeutung bei `Select` |
| --- | --- |
| `Equal` | Feldtext und Suchtext sind exakt gleich. |
| `NonEqual` | Feldtext und Suchtext sind ungleich. |
| `Like` | Ein nicht leerer Suchtext ist Teil eines nicht leeren Feldtexts. |

Alle Vergleiche beachten Groß-/Kleinschreibung. `Like` verwendet keine SQL-Platzhalter und keine regulären Ausdrücke.

```cpp
auto filter = mrdb::WhereVector{};
filter.Add("STATUS", mrdb::WhereOperator::NonEqual, "Abgeschlossen");
```

<a id="typ-whereitem"></a>

#### WhereItem

```cpp
struct WhereItem
{
    std::string Fieldname;
    WhereOperator Operator{WhereOperator::Equal};
    std::string Value;

    auto operator==(WhereItem const & rhs) const -> bool;
};
```

Beschreibt eine Bedingung durch exakten Spaltennamen, Vergleichsoperator und Suchtext. Die Standardinitialisierung ergibt leere Strings und `Equal`. Verwende `WhereVector::Add`, um Bedingungen für `Select` zusammenzustellen; ein `WhereItem` allein wird nicht an `Select` übergeben.

```cpp
auto const condition = mrdb::WhereItem{"STATUS", mrdb::WhereOperator::Equal, "Geplant"};
std::cout << condition.Fieldname << ": " << condition.Value << '\n';
```

<a id="whereitem-gleichheit"></a>

#### WhereItem::operator==(rhs)

```cpp
auto operator==(WhereItem const & rhs) const -> bool;
```

Vergleicht den Bedingungsschlüssel aus Feldname und Operator. Der Suchwert wird nicht verglichen.

- **Parameter:** `rhs`: andere Bedingung.
- **Rückgabe:** `true` bei gleichem `Fieldname` und gleichem `Operator`, auch wenn die Werte unterschiedlich sind; sonst `false`.
- **Voraussetzungen:** Zwei gültige `WhereItem`-Werte; keine Initialisierung nötig.

```cpp
auto const left = mrdb::WhereItem{"STATUS", mrdb::WhereOperator::Equal, "Geplant"};
auto const right = mrdb::WhereItem{"STATUS", mrdb::WhereOperator::Equal, "In Arbeit"};
std::cout << (left == right) << '\n'; // true: Value wird nicht verglichen.
```

<a id="typ-wherevector"></a>

#### WhereVector

`WhereVector` sammelt Filterbedingungen für `Select`. Die Sammlung ist weder kopierbar noch bewegbar: Erzeuge sie am Verwendungsort, ergänze sie mit `Add` und übergib sie als Referenz an `Select`. Alle Bedingungen werden mit UND verknüpft. Eine leere Sammlung wählt alle Zeilen aus.

<a id="wherevector-konstruktor"></a>

#### WhereVector::WhereVector()

```cpp
WhereVector() = default;
```

Erzeugt eine leere Filtersammlung.

- **Parameter:** Keine.
- **Rückgabe:** Kein Rückgabewert (Konstruktor). Anfang und Ende sind zunächst gleich.
- **Voraussetzungen:** Keine Initialisierung nötig.

```cpp
auto filter = mrdb::WhereVector{};
std::cout << (filter.begin() == filter.end()) << '\n'; // true
```

<a id="wherevector-destruktor"></a>

#### WhereVector::~WhereVector()

```cpp
~WhereVector() = default;
```

Gibt die Filtersammlung beim Verlassen ihres Gültigkeitsbereichs automatisch frei.

- **Parameter:** Keine.
- **Rückgabe:** Keiner.
- **Voraussetzungen:** Lebende Sammlung. Danach sind ihre Iteratoren ungültig.

```cpp
{
    auto filter = mrdb::WhereVector{};
    filter.Add("STATUS", "Geplant");
    auto const projects = mrdb::Select(tableName, filter);
    std::cout << projects.size() << '\n';
} // Automatischer Aufruf des Destruktors.
```

<a id="wherevector-add-operator"></a>

#### WhereVector::Add(std::string const & fieldname, WhereOperator const whereOperator, std::string const & value)

```cpp
void Add(std::string const & fieldname, WhereOperator const whereOperator,
         std::string const & value);
```

Ergänzt eine Bedingung mit einem ausdrücklich gewählten Operator.

- **Parameter:** `fieldname`: nicht leerer und nicht ausschließlich aus Leerzeichen bestehender Spaltenname, für `Select` exakt wie in der Tabelle. `whereOperator`: `Equal`, `NonEqual` oder `Like`. `value`: Suchtext, auch leer möglich.
- **Rückgabe:** Keiner. Feldname und Wert werden in die Sammlung kopiert.
- **Voraussetzungen:** Noch keine Bedingung mit demselben Feldnamen und Operator. Ein anderer Suchwert macht eine solche zweite Bedingung nicht zulässig; ein anderer Operator ist erlaubt. `Add` prüft nicht die Spaltenexistenz. Nach `Add` Iteratoren neu beschaffen.
- **Fehler:** `std::runtime_error` bei leerem Feldnamen oder bereits vorhandenem Feldname/Operator-Paar.

```cpp
auto filter = mrdb::WhereVector{};
filter.Add("NAME", mrdb::WhereOperator::Like, "Werkstatt");
```

<a id="wherevector-add-gleichheit"></a>

#### WhereVector::Add(std::string const & fieldname, std::string const & value)

```cpp
void Add(std::string const & fieldname, std::string const & value);
```

Ergänzt eine Gleichheitsbedingung mit `WhereOperator::Equal`.

- **Parameter:** `fieldname`: exakter Spaltenname, weder leer noch nur Leerzeichen. `value`: gesuchter Text, auch leer möglich.
- **Rückgabe:** Keiner. Die Texte werden kopiert.
- **Voraussetzungen:** Noch keine `Equal`-Bedingung für diesen Feldnamen. Keine Initialisierung für das Erstellen des Filters nötig. Nach `Add` Iteratoren neu beschaffen.
- **Fehler:** `std::runtime_error` bei leerem Feldnamen oder doppelter Gleichheitsbedingung für denselben Feldnamen.

```cpp
auto filter = mrdb::WhereVector{};
filter.Add("STATUS", "Geplant");
```

<a id="wherevector-begin"></a>

#### WhereVector::begin()

```cpp
auto begin() const -> std::vector<WhereItem>::const_iterator;
```

Liefert den Anfang der schreibgeschützten Bedingungsiteration in Einfügereihenfolge.

- **Parameter:** Keine.
- **Rückgabe:** Konstanter Iterator auf die erste Bedingung; bei leerem Filter gleich `end()`. Er besitzt die Sammlung nicht.
- **Voraussetzungen:** Lebende Sammlung. Nur dereferenzieren, wenn ungleich `end()`. Nach `Add` neu beschaffen; nach Zerstörung nicht verwenden.

```cpp
auto filter = mrdb::WhereVector{};
filter.Add("STATUS", "Geplant");
auto const first = filter.begin();
if (first != filter.end()) {
    std::cout << first->Fieldname << '\n';
}
```

<a id="wherevector-end"></a>

#### WhereVector::end()

```cpp
auto end() const -> std::vector<WhereItem>::const_iterator;
```

Liefert das Ende der schreibgeschützten Bedingungsiteration.

- **Parameter:** Keine.
- **Rückgabe:** Konstanter Iterator hinter der letzten Bedingung, nicht dereferenzierbar; bei leerem Filter gleich `begin()`.
- **Voraussetzungen:** Lebende Sammlung. Nach `Add` neu beschaffen; nach Zerstörung nicht verwenden.

```cpp
auto filter = mrdb::WhereVector{};
filter.Add("STATUS", "Geplant");
for (auto it = filter.begin(); it != filter.end(); ++it) {
    std::cout << it->Fieldname << ": " << it->Value << '\n';
}
```

<a id="praxisbeispiele"></a>

## Typische Anwendungsfälle

<a id="beispiel-filtern"></a>

### Projekte mit mehreren Bedingungen suchen

Die Tabelle `PROJEKTE` aus dem Schnelleinstieg enthält `NAME` und `STATUS`. Dieses Beispiel wählt geplante Projekte aus, deren Name den Teiltext `Werkstatt` enthält. Beide Bedingungen müssen erfüllt sein.

```cpp
auto filter = mrdb::WhereVector{};
filter.Add("STATUS", "Geplant");
filter.Add("NAME", mrdb::WhereOperator::Like, "Werkstatt");
auto const projects = mrdb::Select("PROJEKTE", filter);
for (auto const & project : projects) {
    std::cout << project.GetRowIDAsString() << ": "
              << project.GetValue("NAME") << '\n';
}
```

<a id="beispiel-aendern"></a>

### Ein Feld ändern oder einen Record speichern

Bei einer vorhandenen ID kannst du nur das gewünschte Feld ändern:

```cpp
mrdb::Update("PROJEKTE", recordId, {{"STATUS", "Abgeschlossen"}});
```

Soll der Datensatz zuvor gelesen und bearbeitet werden, verwende die Record-Überladung. `SetValue` allein speichert noch nicht:

```cpp
auto project = mrdb::SelectSingle("PROJEKTE", recordId);
project.SetValue("STATUS", "Abgeschlossen");
mrdb::Update(project);
```

<a id="beispiel-export"></a>

### Feldnamen und Werte gemeinsam ausgeben

`GetValuesAsKeyValues` liefert die Feldpaare in Spaltenreihenfolge und eignet sich etwa für eine einfache Textansicht eines Datensatzes:

```cpp
auto const project = mrdb::SelectSingle("PROJEKTE", recordId);
auto const fields = project.GetValuesAsKeyValues();
for (auto const & field : fields) {
    std::cout << field.Key << " = " << field.Value << '\n';
}
```

<a id="beispiel-schutz-loeschen"></a>

### Schreibschutz verwenden und ein Projekt löschen

```cpp
mrdb::SetTableReadOnly("PROJEKTE", true);
auto const projects = mrdb::SelectAll("PROJEKTE"); // Lesen ist weiter möglich.
std::cout << projects.size() << '\n';
mrdb::SetTableReadOnly("PROJEKTE", false);
mrdb::Delete("PROJEKTE", recordId); // Vorhandenes Projekt bewusst entfernen.
```

Der Schreibschutz gilt für mrdb-Datensatzänderungen. Er ersetzt keine Zugriffsrechte auf das Datenverzeichnis.

<a id="fehler"></a>

## Fehlerbehandlung

Die API verwendet Exceptions und keine numerischen Fehlercodes. `Insert`, `Update`, `Delete`, `CreateTable` und andere `void`-Funktionen melden keinen Erfolg als Rückgabewert. Eine Ausnahme ist `IsTableExist`: `false` kann sowohl eine fehlende Tabelle als auch einen Fehler bedeuten. Für eine erfolgreiche Existenzprüfung muss die anschließende Operation trotzdem auf Fehler reagieren.

| Situation | Fehlerbehandlung |
| --- | --- |
| Fehlende Tabelle oder Zeile, ungültige Namen/Werte, Schreibschutz | In der Regel `std::runtime_error` abfangen. |
| Unbekannter Spaltenname bei `Record::GetValue` oder `SetValue` | `std::invalid_argument` abfangen. |
| Ungültiger Index bei `Record::GetValue` | `std::out_of_range` abfangen. |
| Fehler beim Dateisystemzugriff | Unter anderem `std::filesystem::filesystem_error`; über `std::exception` ebenfalls erfassbar. |
| `isValidRecord` schlägt fehl | `std::runtime_error` abfangen; normale Rückkehr bedeutet bestandene Prüfung. |

Die Fehlertexte können dem Anwender zur Diagnose angezeigt werden. Verlasse dich für die Programmsteuerung auf Ausnahmetypen und zulässige Eingaben statt auf den Wortlaut einzelner Meldungen.

<a id="beispiel-fehler"></a>

### Fehler einer Änderung behandeln

Die Datenbasis und Tabelle seien wie im Schnelleinstieg vorbereitet. Ein leerer Name ist für die Pflichtspalte `NAME` nicht zulässig:

```cpp
try {
    mrdb::Update("PROJEKTE", recordId, {{"NAME", ""}});
}
catch (std::runtime_error const & error) {
    std::cerr << "Projekt konnte nicht geändert werden: " << error.what() << '\n';
}
```

Bei einem möglicherweise fehlenden Feld lässt sich ein ungültiger Spaltenname gesondert behandeln:

```cpp
try {
    auto const project = mrdb::SelectSingle("PROJEKTE", recordId);
    std::cout << project.GetValue("BESCHREIBUNG") << '\n';
}
catch (std::invalid_argument const & error) {
    std::cerr << "Spalte nicht verfügbar: " << error.what() << '\n';
}
catch (std::exception const & error) {
    std::cerr << "Projekt konnte nicht gelesen werden: " << error.what() << '\n';
}
```

`SelectAll` und `Select` liefern bei einer lesbaren Tabelle ohne Zeilen beziehungsweise ohne Treffer einen leeren Vektor. Das ist ein reguläres Ergebnis:

```cpp
auto filter = mrdb::WhereVector{};
filter.Add("STATUS", "Geplant");
auto const projects = mrdb::Select("PROJEKTE", filter);
if (projects.empty()) {
    std::cout << "Keine geplanten Projekte vorhanden" << '\n';
}
```

Nach einem Dateizugriffsfehler bei einer Schreiboperation solltest du den betroffenen Bestand prüfen, bevor du den Aufruf wiederholst. Die API bietet keine Transaktions- oder Rücksetzfunktion; Änderungen können teilweise erfolgt sein. Koordiniere gleichzeitige Datenzugriffe in deinem Programm und mit anderen Schreibern, insbesondere während Änderungen.
