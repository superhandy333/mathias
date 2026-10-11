{% raw %}
# mrdb – ausführliche Dokumentation

<small>[Entwicklung](develop.md) / [mrdb-Startseite](mrdb.md) / Dokumentation</small>

Diese Referenz beschreibt den am 11. Oktober 2026 untersuchten lokalen Stand von `CPP/tools/mrdb.h`, einschließlich der für `Select` benötigten Typen aus `wherevector.h`. Maßgeblich sind die Deklarationen und Funktionskörper dieser Dateien; ältere Beispiele können andere Bibliotheksstände beschreiben.

<!-- TOC -->

- [Überblick](#ueberblick)
- [Schnelleinstieg](#schnelleinstieg)
- [Grundbegriffe und Datenmodell](#datenmodell)
- [Einbindung, Voraussetzungen und Lifecycle](#lifecycle)
- [Vollständige Referenz der öffentlichen API](#api)
  - [Regeln für die API-Einträge und Beispiele](#api-regeln)
  - [Namespace, Typen und Konstanten](#typen)
  - [Initialisierung und Tabellen](#tabellen)
  - [Schreiben und Lesen von Datensätzen](#datensaetze)
  - [Record](#record)
  - [Filtertypen aus wherevector.h](#filter)
- [Fehlerbehandlung und Garantien](#fehler)
- [Thread-Sicherheit und Nebenläufigkeit](#threads)
- [Performance und Ressourcen](#performance)
- [Praxisbeispiele](#praxisbeispiele)
- [Quellen und Prüfstand](#quellen)

<!-- /TOC -->

<a id="ueberblick"></a>

## Überblick

mrdb (Mathias-Rentsch-Daten-Bank) verwaltet Tabellen mit benannten Spalten und Datensätzen. Alle Feldwerte sind `std::string`. Die Daten werden über die freie API im Namespace `mrdb` in Textdateien gespeichert und wieder eingelesen. Es gibt kein Datenbankobjekt und keine Verbindungsklasse: Der gemeinsame Basispfad wird mit `Init` gesetzt.

`mrdb.h` ist Header-only; seine freien Implementierungen sind `inline`. Die gesamte Einbindung ist dennoch nicht allein mit diesem Header abgeschlossen: Die mitgelieferten Hilfsfunktionen aus `tools.cpp` müssen mitgelinkt werden. Externe Datenbankserver oder Datenbankbibliotheken werden von dieser API nicht benötigt.

Ein typischer Einsatz ist eine kleine Werkzeugverwaltung mit den Spalten `NAME` und `ORT`: Eine Tabelle wird einmal angelegt, Werkzeuge werden als Textwerte eingefügt, nach ihrem Standort gefiltert und bei einer Umlagerung aktualisiert. SQL, Transaktionen, Datentypprüfungen und eine Schemaänderungs-API werden nicht angeboten.

> Angaben zur beobachteten Persistenz, Reihenfolge, ID-Vergabe, Prüfung und Laufzeit beschreiben die aktuelle Implementierung. Sie sind nicht ausdrücklich als dauerhafter Vertrag garantiert. Die folgenden Einschränkungen sind bei der Verwendung dieses Standes zu beachten.

<a id="schnelleinstieg"></a>

## Schnelleinstieg

Verwende C++20 entsprechend `CPP/tools/mrdbtest/CMakeLists.txt`, die Standardbibliothek mit `std::filesystem` und die Dateien `mrdb.h`, `wherevector.h`, `tools.h`, `tools.cpp` aus `CPP/tools/`. Eine separate `mrdb.cpp` gehört nicht zur hier dokumentierten Variante. Die in diesem Header sichtbaren Sprachmittel setzen mindestens C++17 voraus (unter anderem Inline-Variablen und `std::filesystem`); daraus folgt keine Prüfung aller Abhängigkeiten für C++17. C++20 ist die Projektvorgabe und die Grundlage der Beispiele.

Das vollständige Programm verwendet ein eigenes Beispielverzeichnis. Bei wiederholtem Start wird ein weiterer Datensatz eingefügt. Bei einer vorhandenen Tabelle setzt es das gezeigte Schema und Schreibbarkeit voraus.

~~~cpp
#include "mrdb.h"

#include <exception>
#include <filesystem>
#include <iostream>

auto main() -> int
{
    try {
        std::filesystem::create_directories("./mrdb-beispiel");
        mrdb::Init({"./mrdb-beispiel/"});

        if (!mrdb::IsTableExist("WERKZEUGE")) {
            auto table = mrdb::Table{};
            table.Name = "WERKZEUGE";
            table.Columns = {{"01", "NAME", false}, {"02", "ORT", true}};
            mrdb::CreateTable(table);
        }

        mrdb::Insert("WERKZEUGE", {{"NAME", "Messschieber"}, {"ORT", "Werkstatt"}});

        auto filter = mrdb::WhereVector{};
        filter.Add("NAME", "Messschieber");
        auto const records = mrdb::Select("WERKZEUGE", filter);
        for (auto const & record : records) {
            std::cout << record.GetRowID() << ": "
                      << record.GetValue("NAME") << " – "
                      << record.GetValue("ORT") << '\n';
        }
        return 0;
    }
    catch (std::exception const & error) {
        std::cerr << error.what() << '\n';
        return 1;
    }
}
~~~

`Insert` liefert keine ID zurück. Das Beispiel liest passende Datensätze deshalb über `Select`; ihre Identifikation erfolgt mit `GetRowID()`. Mehrere Datensätze mit demselben `NAME` sind zulässig.

<a id="datenmodell"></a>

## Grundbegriffe und Datenmodell

| Begriff | Bedeutung im aktuellen Stand |
| --- | --- |
| Datenbank | Basisverzeichnis, dessen Pfad durch `InitData::NAME_DATABASE_DIRECTORY` gesetzt wird; prozessweit gemeinsam. |
| `Table` | Kopierbare Beschreibung mit `ID`, `Name`, `ReadOnly`, `Columns`. Änderungen an diesem Wert werden nicht automatisch gespeichert. |
| `Column` | Spaltenbeschreibung mit `SortIndex`, `Name`, `Nullable`. Es gibt kein Spaltentyp-Enum. |
| `Record` | Beweglicher, nicht kopierbarer Wert mit Zeilen-ID, eigenen Feldwerten und geteilter Tabellenbeschreibung. Ein gelesener Record ist keine offene Datei und kein Live-Zugriff auf gespeicherte Felder. |
| Zeilen-ID | `unsigned int`, unabhängig von `Table::ID` und von einer gegebenenfalls vorhandenen benannten Spalte `ID`. |
| Feldwert | `std::string`; keine eigene NULL-Repräsentation, kein `optional`, keine automatische Zahlen- oder Bool-Konvertierung. |
| `Nullable` | Bei Schreibaufrufen dürfen leere Werte für diese Spalte vorkommen. Pflichtspalten müssen bei `Insert` angegeben werden. |
| `ReadOnly` | Gespeichertes Tabellenflag, das `Insert`, beide `Update`-Varianten und `Delete` prüfen. Es ist kein Dateisystem-Zugriffsschutz. |
| `WhereVector` | Sammlung von Bedingungen, die `Select` mit logischem UND auswertet. |

### Namen, Werte und Reihenfolge

Tabellen- und Spaltennamen werden für Dateizugriffe mit den mitgelieferten `tools`-Funktionen in Großbuchstaben umgesetzt. Bei `CreateTable` sind Spaltennamen auch nach dieser Umsetzung eindeutig erforderlich. Das ist keine allgemeine Unicode- oder sprachabhängige Vergleichsgarantie. ASCII-Namen wie `WERKZEUGE` und `NAME` vermeiden diese Unsicherheit.

Die aktuelle Namensprüfung akzeptiert 1 bis 255 Bytes, untersagt `<>:"/\|?*`, Steuerbytes unter 32, ausschließlich Punkte/Leerzeichen, führende Leerzeichen, abschließende Punkte/Leerzeichen sowie die exakt geprüften reservierten Namen `CON`, `PRN`, `AUX`, `NUL`, `COM1` bis `COM9`, `LPT1` bis `LPT9`. Namen müssen einzelne Pfadkomponenten sein; `.` und `..` sind unzulässig. Diese Prüfung ist keine vollständige Zusage, dass ein Name auf jedem Dateisystem angelegt werden kann. Längenbeschränkungen für den gesamten Pfad und weitere Plattformregeln können zusätzlich wirken.

`Insert` und das tabellenbezogene `Update` normalisieren Schlüssel. `Record::GetValue(columnName)`, `Record::SetValue` und die Filterauswertung vergleichen dagegen exakt mit `Column::Name`. Für gelesene Tabellen deshalb die tatsächlich zurückgegebenen Spaltennamen verwenden, normalerweise Großbuchstaben. `GetTable("werkzeuge").Name` enthält aktuell die übergebene Schreibweise, nicht zwingend eine normalisierte Form.

`SortIndex` wird als String lexikographisch sortiert: `"10"` liegt vor `"2"`. Gleich lange Indizes wie `"01"`, `"02"`, `"10"` sind für eine numerisch gedachte Ordnung geeignet. Bei gleichen Indizes ist keine stabile Reihenfolge zugesagt. Der Index von `Record::GetValue(int)` und die Wertiteration beziehen sich auf diese gelesene Spaltenreihenfolge, nicht zwingend auf die Reihenfolge in `CreateTable`.

„Leer oder Whitespace“ bedeutet in den verwendeten `tools`-Funktionen derzeit ausschließlich: leer oder nur ASCII-Leerzeichen (`' '`). Tabs und Zeilenumbrüche werden damit nicht als leer erkannt. Beim Lesen werden solche leeren bzw. nur aus Leerzeichen bestehenden Inhalte zu `""` normalisiert. Andere Werte werden im Textmodus gelesen/geschrieben; eine plattformübergreifend byteidentische Behandlung von Zeilenenden oder eine festgelegte Zeichenkodierung wird nicht zugesagt.

Tabellen- und Zeilenlisten folgen der Dateisystemiteration. Eine sortierte Reihenfolge ist nicht garantiert. Die aktuelle automatische Zeilen-ID ist die höchste vorhandene ID plus eins, beginnend mit eins. Nach Löschen der höchsten ID ist Wiederverwendung möglich; ein Überlauf wird nicht abgefangen. Die API sichert daher weder dauerhaft eindeutige IDs noch lückenlose oder global monotone Nummerierung zu.

<a id="lifecycle"></a>

## Einbindung, Voraussetzungen und Lifecycle

### Header und CMake

Die Header schließen `wherevector.h` und `tools.h` relativ zu ihrer Lage ein. Behalte die Lage der Dateien unter `tools/` bei und füge dieses Verzeichnis als Include-Verzeichnis hinzu. Der Namespace der Bibliothek ist `mrdb`; `tools` gehört zur mitgelieferten Abhängigkeit und wird hier nicht als mrdb-API dokumentiert.

Beispiel für `CPP/tools/meinprojekt/CMakeLists.txt` mit obigem Programm in `main.cpp`:

~~~cmake
cmake_minimum_required(VERSION 3.28)
project(mrdbBeispiel LANGUAGES CXX)

add_executable(mrdbBeispiel main.cpp ../tools.cpp)
target_compile_features(mrdbBeispiel PRIVATE cxx_std_20)
target_include_directories(mrdbBeispiel PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/..")
set_target_properties(mrdbBeispiel PROPERTIES CXX_EXTENSIONS OFF)
~~~

Der Compiler wird nicht in der `CMakeLists.txt` festgelegt. CMake 3.28 entspricht dem untersuchten Testprojekt; mrdb selbst hat keine CMake-Abhängigkeit zur Laufzeit. Für andere Projektverzeichnisse sind die relativen Quellen- und Include-Pfade entsprechend anzupassen.

### Reihenfolge der Operationen

1. Basisverzeichnis außerhalb der mrdb-API anlegen oder ein vorhandenes verwenden.
2. `Init` mit einem nicht leeren Pfad und abschließendem `/` aufrufen. `Init` prüft weder Pfad noch Verzeichnis und korrigiert keinen fehlenden Separator.
3. `CreateTable` aufrufen oder mit `GetTable` ein bestehendes Schema lesen. `IsTableExist` prüft lediglich ein Verzeichnis und ist kein Integritätstest.
4. `Insert`, `SelectSingle`, `SelectAll`, `Select`, `Update` oder `Delete` aufrufen.
5. Nach `Record::SetValue` explizit `Update(record)` zum Speichern aufrufen. Ohne diesen Aufruf bleibt die Änderung lokal.
6. Ergebnisse durch ihre normalen C++-Lebensdauern freigeben. Es gibt keine öffentliche `Open`-, `Close`-, `Save`-, `Flush`- oder Commit-Funktion und keinen Schreib-/Lesemodus pro Objekt.

`Init` kann erneut aufgerufen werden; nachfolgende Dateizugriffe verwenden dann den neuen gemeinsamen Pfad. Bereits gelesene `Record`- und `Table`-Werte bleiben im Speicher erhalten. Ein späteres `Update(record)` verwendet jedoch den *aktuellen* Pfad und den gespeicherten Tabellennamen: Es kann eine gleichnamige Tabelle in einer anderen Datenbasis treffen. Records binden die Datenbasis nicht an sich.

### Lebensdauer und Ownership

Alle Strings, Maps, Vektoren, `Table`- und `Record`-Ergebnisse werden als Werte zurückgegeben. Der Aufrufer besitzt diese Ergebnisse; die Standardtypen geben ihren Speicher automatisch frei. Kein API-Ergebnis muss mit `delete`, `delete[]` oder `free` freigegeben werden.

`Record` kopiert beim expliziten Konstruktor die übergebene Wertliste und teilt den Besitz der übergebenen Tabelle über `TablePtr` (`std::shared_ptr<Table>`). `GetTable()` liefert eine Kopie dieses Smartpointers. Die Tabelle bleibt bis zum letzten Besitzer erhalten; der Smartpointer kann bei manuell konstruierten oder verschobenen Records leer sein. Ein gelesener Record besitzt zunächst eine nicht leere Tabellenbeschreibung. Bei `SelectAll`/`Select` wird derzeit für jede gelesene Zeile eine eigene Tabellenbeschreibung erzeugt.

Die Tabelle hinter `TablePtr` ist veränderbar, auch wenn der Record `const` ist. Änderung, Umordnung oder Verkleinerung ihrer `Columns` kann die Zuordnung zu den gespeicherten Feldwerten zerstören. `GetValuesAsMap` und `GetValuesAsKeyValues` prüfen diese Konsistenz nicht vollständig; zu wenige Werte können zu undefiniertem Verhalten führen. Die Tabellenbeschreibung eines Records daher nicht strukturell verändern.

`Record::begin/end` und `WhereVector::begin/end` liefern konstante Vektoriteratoren. Sie besitzen keinen Speicher. Details zur Invalidierung stehen bei den betreffenden Einträgen. Die Felditeration eines Records bleibt von späteren Datenbankänderungen unabhängig; gespeicherte Änderungen aktualisieren ältere Ergebnisse nicht automatisch. Move-Ziele können weiterverwendet werden; für Move-Quellen sind weder eine weiterhin gültige Tabelle noch unveränderte Werte zugesagt.

<a id="api"></a>

## Vollständige Referenz der öffentlichen API

<a id="api-regeln"></a>

### Regeln für die API-Einträge und Beispiele

Die Referenz umfasst die freien API-Deklarationen vor dem Implementierungsabschnitt von `mrdb.h`, alle öffentlichen Member von `Record`, die öffentlichen Datentypen und die Filtertypen aus `wherevector.h`. Das ausdrücklich dort deklarierte, als „Temporär“ kommentierte `isValidRecord` gehört dazu. Namespaceweite Hilfsfunktionen, der gemeinsame interne Zustand und Metadaten-Schlüssel des Implementierungsabschnitts sind zwar technisch vom Header sichtbar, werden aber nicht als eigenständige öffentliche Schnittstelle oder stabiles Dateiformat angeboten.

Signaturen sind aus den Deklarationen übernommen; Klassenmember stehen ohne qualifizierenden Klassenpräfix im jeweiligen Klassenabschnitt. Parameterbezeichnungen im Text entsprechen gegebenenfalls der Definition, wenn die Deklaration keinen Namen enthält. `inline` der freien Definitionen ändert die Signaturen nicht. Es wird kein im Header nicht geschriebenes `noexcept` ergänzt.

Für **jeden** Eintrag gelten ergänzend diese gemeinsamen Regeln:

- **Parameter:** Alle referenzierten Eingaben sind `in`, sofern nicht `in/out` angegeben. C++-Referenzen sind nicht nullable; ein ungültig erzeugter Referenzwert ist keine unterstützte Eingabe. Für Funktionen ohne Parameter ist Nullability nicht zutreffend. Nullability von `TablePtr` wird gesondert beschrieben.
- **Ownership:** Eingabereferenzen werden nicht vom Aufrufer übernommen. Wertresultate gehören dem Aufrufer; automatische Freigabe, keine manuelle Speicherfreigabe. Bei `void` sind Rückgabewert und Ergebnis-Ownership nicht zutreffend. Ausnahmen sind die ausdrücklich geteilte Tabelle und geliehene Iteratoren.
- **Zustand:** „Unverändert“ meint den Zustand von mrdb bzw. den betrachteten Objekten, ohne die Zuweisung an eine lokale Ergebnisvariable. Nach `throw` existiert kein reguläres Funktionsergebnis.
- **Dateizugriffe (Regel D):** Gemeinsamer Pfad muss initialisiert sein; Basisverzeichnis muss zugänglich sein. Die ausdrücklich genannten Prüfungen können `std::runtime_error` auslösen. Außerdem können `std::filesystem::filesystem_error`, Allokationsfehler und bei numerischem Einlesen von Metadaten/IDs `std::invalid_argument`/`std::out_of_range` weitergegeben werden. Die API fängt diese Fehler nur bei `IsTableExist` ab. Keine Synchronisation, keine Snapshot-Isolation; siehe [Nebenläufigkeit](#threads).
- **Nur Speicher (Regel S):** Keine Initialisierung und keine Datei erforderlich. Gleichzeitige Änderungen des betreffenden Objekts oder seiner geteilten Tabelle müssen extern ausgeschlossen werden. Lesender Zugriff auf unveränderte Objekte erscheint möglich; nicht ausdrücklich garantiert. Kopierende Ergebnisse können wegen Speicherbedarf werfen; keine allgemeine No-Throw-Zusage.
- **Garantien:** Kategorien beziehen sich auf den beobachteten Zustand bei erfüllten Vorbedingungen. „Stark (abgeleitet)“ bedeutet: Bei einer Exception bleibt der betrachtete Zustand unverändert. „Keine für Persistenz“ bedeutet: Auch eine basale Zusage konsistenter gespeicherter Daten ist bei einem Teilfehler nicht ableitbar. Dies ist keine Aussage über fehlendes RAII der lokalen Standardtypen. [Fehlerbehandlung](#fehler) erläutert die Abgrenzung.

Die kurzen Beispiele sind unabhängige Ausschnitte innerhalb einer Funktion; sie dürfen denselben Setup-Code verwenden. Für Dateiaufrufe sei die Datenbasis initialisiert, die Tabelle vorhanden und schreibbar und `recordId` die ID einer vorhandenen Zeile. Benötigte Includes: `mrdb.h` sowie für die Beispielausgabe/-prüfungen `<iostream>`, `<utility>`, `<type_traits>`, `<exception>`, `<filesystem>`, `<stdexcept>`, `<string>`, `<vector>`.

Gemeinsame Variablen für die Ausschnitte (an das vollständige Praxisbeispiel anschließbar):

~~~cpp
auto const tablename = std::string{"WERKZEUGE"};
auto ids = mrdb::GetRecordIDs(tablename);
if (ids.empty()) {
    throw std::runtime_error("Beispiel benötigt einen Datensatz");
}
auto const recordId = ids.front();
auto table = mrdb::GetTable(tablename);
auto record = mrdb::SelectSingle(tablename, recordId);
~~~

<a id="typen"></a>

### Namespace, Typen und Konstanten

Alle hier genannten mrdb-Typen und freien API-Funktionen liegen in `namespace mrdb`. `Record` ist `final`; `WhereVector` wird über `wherevector.h` eingebunden. Es gibt in `mrdb.h` selbst keine weiteren öffentlichen Enums. Die einzige für Fehlerberichte verwendete Konstante ist `ERROR_TABLE_NOT_FOUND`; numerische Fehlercodes oder Status-Enums sind nicht vorhanden.

#### InitData

~~~cpp
struct InitData
{
    std::string NAME_DATABASE_DIRECTORY{};
};
~~~

Öffentliches Aggregat mit leerem Standardpfad. Der Wert ist kopier- und bewegbar. `NAME_DATABASE_DIRECTORY` ist `in` für `Init`; gültige Nutzereingabe ist ein nicht leerer Basispfad mit abschließendem `/`. Existenz und Zugriffsrechte werden erst bei Dateiaufrufen relevant. Der String gehört dem `InitData`-Objekt. Aggregat-/Standardinitialisierung hat keine mrdb-Fehlermeldung; Stringkopien können allokieren. Regel S. Keine gespeicherte Zustandsänderung durch Änderungen an diesem Wert.

~~~cpp
auto data = mrdb::InitData{};
data.NAME_DATABASE_DIRECTORY = "./mrdb-beispiel/";
mrdb::Init(data);
~~~

#### KeyValue, KeyValues

~~~cpp
struct KeyValue
{
    std::string Key{};
    std::string Value{};
    KeyValue(const std::string& k, const std::string& v) : Key(k), Value(v) {}
};
using KeyValues = std::vector<KeyValue>;
~~~

**Konstruktor `KeyValue`:** Kopiert `k` (`std::string const &`, in: Schlüssel, beliebiger String einschließlich leer) und `v` (`std::string const &`, in: Wert, ebenfalls beliebig) in die öffentlichen Felder `Key` und `Value`. Keine Validierung und keine Initialisierung nötig. Rückgabe: nicht zutreffend (Konstruktor). Ownership: eigene Stringkopien; Eingaben müssen nach dem Aufruf nicht weiterleben. Nach Erfolg sind beide Felder gesetzt. Bei Kopier-/Allokationsfehlern wird kein Objekt fertig konstruiert, Eingaben bleiben unverändert (stark, abgeleitet). Regel S; Kosten proportional zu den kopierten Texten, mögliche Heap-Allokationen.

`KeyValue` hat keinen Default-Konstruktor, da dieser Konstruktor benutzerdefiniert ist. Seine Kopier-/Moveoperationen und Destruktion sind implizite Standardoperationen der zwei Strings. `KeyValues` ist genau der Standardvektor, keine eigene Klasse; dessen API wird nicht neu definiert.

~~~cpp
auto entry = mrdb::KeyValue{std::string{"NAME"}, std::string{"Messschieber"}};
auto entries = mrdb::KeyValues{};
entries.push_back(entry);
std::cout << entries.front().Key << ": " << entries.front().Value << '\n';
~~~

#### Column

~~~cpp
struct Column
{
    std::string SortIndex{};
    std::string Name{};
    bool Nullable{true};
};
~~~

Öffentliches Aggregat mit zwei leeren Strings und `Nullable == true`. `SortIndex` ist ein beliebiger Ordnungsstring; `Name` muss bei `CreateTable` ein gültiger Spaltenname sein; `Nullable` erlaubt bei `true` leere Werte. Die Struktur allein prüft diese Bedingungen nicht. Öffentliche Felder sind `in/out` für den eigenen Wert und werden bei `CreateTable` als `in` gelesen. Kein Ergebniszeiger; Strings gehören dem Wert. Regel S; Kopieren kann allokieren. Keine persistente Schemaänderung durch Feldzuweisung.

~~~cpp
auto column = mrdb::Column{"01", "NAME", false};
column.SortIndex = "02"; // Ändert nur diesen Wert.
~~~

#### Table, TablePtr

~~~cpp
struct Table
{
    unsigned int ID{0};
    std::string Name{};
    bool ReadOnly{false};
    std::vector<Column> Columns{};
};
using TablePtr = std::shared_ptr<Table>;
~~~

Öffentliches Aggregat. `ID` ist zunächst null; `CreateTable` speichert aktuell unabhängig von der übergebenen `ID` null und weist dem Eingabewert keine ID zu. Daraus keine eindeutige Tabellen-ID ableiten. `Name` muss für Dateiaufrufe gültig sein. `ReadOnly` ist das bei Anlage zu speichernde Flag; `Columns` muss für `CreateTable` mindestens eine Spalte mit eindeutigen Namen enthalten. Diese Felder sind am eigenen Objekt `in/out`, für `CreateTable` `in`.

Strings und Spaltenvektor gehören dem `Table`-Wert. `TablePtr` ist ein nullable Smartpointer mit geteiltem Besitz, kein vom Nutzer freizugebender Rohzeiger. Die letzte besitzende Kopie zerstört die Tabelle automatisch. Feldänderungen werden nicht automatisch persistiert; das ReadOnly-Flag wird separat mit `SetTableReadOnly` gesetzt. Regel S, kopierende Operationen proportional zu Beschreibungsdaten und möglicherweise allokierend; keine eigene Fehlervalidierung durch das Aggregat.

~~~cpp
auto description = mrdb::Table{};
description.Name = "WERKZEUGE";
description.Columns = {{"01", "NAME", false}, {"02", "ORT", true}};
auto sharedDescription = std::make_shared<mrdb::Table>(description);
~~~

`InitData`, `Column` und `Table` deklarieren keine eigenen Konstruktoren, Destruktoren oder Operatoren. Aggregatinitialisierung, implizite Default-/Kopier-/Moveoperationen und Destruktion verwenden die Standardoperationen der Member. Defaultinitialisierung erzeugt die oben angegebenen Defaults; Kopieren erzeugt eigene Strings/Container; Move lässt die Quelle in einem gültigen, nicht inhaltlich zugesagten Zustand. Rückgabe bei Konstruktion/Destruktion: nicht zutreffend; implizite Zuweisung liefert eine Referenz auf das Ziel, gültig für dessen Lebensdauer und ohne Ownership-Transfer dieser Referenz. Kein Dateizugriff. Bei Kopierzuweisung mehrerer Member kann ein später Fehler nach früheren Memberänderungen eintreten: keine allgemeine starke Garantie für das gesamte Aggregat; Standardmember bleiben gültig (basal, abgeleitet). Selbstdeklarierte zusätzliche Signaturen existieren dafür im Header nicht.

#### Records

~~~cpp
using Records = std::vector<Record>;
~~~

Besitzender Ergebnisvektor. Leere Auswahl liefert einen leeren Vektor. Wegen der gelöschten Kopieroperationen von `Record` sind seine Elemente durch Referenz oder Move zu verwenden, nicht per Wert zu kopieren. Standardvektor-Referenzen/Iteratoren können bei Reallokation, Löschung oder Zerstörung ungültig werden. Kein eigener mrdb-Lifecycle.

~~~cpp
auto records = mrdb::SelectAll("WERKZEUGE");
for (auto const & row : records) {
    std::cout << row.GetRowID() << '\n';
}
~~~

#### ERROR_TABLE_NOT_FOUND

~~~cpp
inline std::string const ERROR_TABLE_NOT_FOUND = "table not found";
~~~

Konstante Fehlermeldung, als Inhalt einer `std::runtime_error` bei fehlender Tabelle verwendet. Sie ist kein numerischer Fehlercode und deckt nicht alle Dateifehler ab. Keine Parameter, kein Funktionsrückgabewert, keine vom Aufrufer auszuführende Freigabe; der konstante String hat statische Lebensdauer. Seine Initialisierung kann als Stringinitialisierung allokieren; lesende Verwendung verändert ihn nicht. Regeln für statische Initialisierung sind die normalen C++-Regeln.

~~~cpp
std::cerr << mrdb::ERROR_TABLE_NOT_FOUND << '\n';
~~~

<a id="tabellen"></a>

### Initialisierung und Tabellen

#### Init

~~~cpp
auto Init(InitData const &) -> void;
~~~

- **Zweck:** Setzt den prozessweit gemeinsamen Basispfad durch Kopieren der Konfiguration.
- **Parameter:** `InitData const &` (in, in der Definition `data`); für anschließende Dateizugriffe nicht leer, Basisverzeichnis vorhanden, abschließender `/`. Referenz nicht nullable. `Init` selbst akzeptiert auch einen leeren oder ungültigen Pfad.
- **Rückgabe/Ownership:** `void`, nicht zutreffend; interne Kopie des Strings, Eingabe darf danach zerstört werden.
- **Vor-/Nachbedingungen:** Keine vorherige Initialisierung nötig. Nach Erfolg verwenden spätere Dateiaufrufe diesen Pfad; keine Verzeichnisanlage oder Dateiprüfung. Bestehende Records behalten ihre Werte. Leerer Pfad führt später zu Initialisierungsfehlern, teils von `IsTableExist` in `false` umgewandelt.
- **Fehler/Garantie:** Stringzuweisung kann werfen; starke Garantie der Stringzuweisung für den bisherigen Pfad (abgeleitet), kein Statuswert, kein deklariertes `noexcept`.
- **Nebenläufigkeit/Kosten:** Extern mit sämtlichen Dateiaufrufen synchronisieren. Stringkopie proportional zur Pfadlänge, mögliche Allokation, keine I/O.

~~~cpp
mrdb::Init({"./mrdb-beispiel/"});
~~~

#### CreateTable

~~~cpp
auto CreateTable(Table const &) -> void;
~~~

- **Zweck:** Legt eine Tabelle mit Schema und ReadOnly-Flag an; gibt zusätzlich eine Meldung über `std::cout` aus.
- **Parameter:** `Table const &` (in, Definition `table`): gültiger `Name`, nicht leere `Columns`, gültige und nach Großschreibung eindeutige `Column::Name`. `SortIndex` beliebiger String, `Nullable`/`ReadOnly` bool. Übergebene `ID` wird aktuell ignoriert.
- **Rückgabe/Ownership:** `void`, nicht zutreffend. Keine Referenz auf die Eingabe gespeichert; Beschreibungswerte werden geschrieben.
- **Vor-/Nachbedingungen:** Regel D; Tabelle darf noch nicht existieren. Nach Erfolg ist das Schema lesbar, zunächst ohne Datensätze; der Eingabewert bleibt unverändert. Bei Validierungsfehlern vor Anlage keine neue Tabelle; bei I/O- oder Ausgabefehlern kann eine teilweise oder bereits vollständig angelegte Tabelle zurückbleiben.
- **Fehler/Garantie:** `std::runtime_error`, unter anderem `"table already exist"`, `"table has no columns"`, ungültige bzw. doppelte Namen; weitere Fehler nach Regel D, gegebenenfalls Ausgabestream-Ausnahmen. Keine starke oder basale Garantie für persistente Konsistenz nach Beginn der Anlage, kein Rollback.
- **Nebenläufigkeit/Kosten:** Regel D; Existenzprüfung und Anlage sind nicht atomar. Prüfung der Eindeutigkeit über einen Set, Kopien und Allokationen, Dateizugriffe für jede Spaltenbeschreibung und Konsolenausgabe.

~~~cpp
auto description = mrdb::Table{};
description.Name = "NEUE_WERKZEUGE";
description.Columns = {{"01", "NAME", false}, {"02", "ORT", true}};
mrdb::CreateTable(description); // Eigenständiges Beispiel: Name muss noch frei sein.
~~~

#### GetTable

~~~cpp
auto GetTable(std::string const & tablename) -> Table;
~~~

- **Zweck:** Liest die Tabellenbeschreibung einschließlich sortierter Spalten neu ein.
- **Parameter:** `tablename` (`std::string const &`, in): gültiger Name einer vorhandenen, vollständig angelegten Tabelle; Dateizugriff normalisiert Großschreibung.
- **Rückgabe/Ownership:** `Table` als eigener Wert mit ID, Name, ReadOnly und Columns; niemals ein nullable Zeiger. Name entspricht aktuell der Eingabe, Spaltennamen der gespeicherten Schreibweise.
- **Vor-/Nachbedingungen:** Regel D; lesbare Metadaten. Keine Zustandsänderung, auch nicht bei Fehler. Nach Erfolg lexikographisch nach `SortIndex` sortierte Spalten; Gleichstände nicht stabil zugesagt.
- **Fehler/Garantie:** Fehlende Tabelle `std::runtime_error(ERROR_TABLE_NOT_FOUND)`; fehlende/beschädigte Metadaten und Regel D. Stark hinsichtlich unveränderter Datenbasis (abgeleitet), keine Konsistenzgarantie gegenüber parallelen Änderungen.
- **Nebenläufigkeit/Kosten:** Regel D; Metadaten-I/O, String-/Vektorallokationen, Spaltensortierung mit `O(c log c)` Vergleichen für `c` Spalten, jeder Stringvergleich zusätzlich längenabhängig.

~~~cpp
auto description = mrdb::GetTable(tablename);
std::cout << description.Name << ": " << description.Columns.size() << '\n';
~~~

#### GetTableNames

~~~cpp
auto GetTableNames() -> std::vector<std::string>;
~~~

- **Zweck:** Listet die Namen der Unterverzeichnisse im Basisverzeichnis auf.
- **Parameter:** Nicht zutreffend.
- **Rückgabe/Ownership:** Eigener `std::vector<std::string>`, leer bei keinem Unterverzeichnis. Keine nullable Ergebnisse.
- **Vor-/Nachbedingungen:** Regel D. Keine Änderung. Jeder gefundene Verzeichnisname wird aufgenommen, ohne Prüfung eines Tabellenschemas; keine Sortierzusage. Fremde Unterverzeichnisse können also als Namen erscheinen.
- **Fehler/Garantie:** Nicht initialisiert: `std::runtime_error("mrdb is not init")`; Verzeichniszugriff/Allokation nach Regel D. Stark hinsichtlich unveränderter Datenbasis (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel D; ein Durchlauf über die Basiseinträge, Ergebnisallokationen proportional zu den Namen; keine Metadatenprüfung.

~~~cpp
auto const names = mrdb::GetTableNames();
for (auto const & name : names) {
    std::cout << name << '\n';
}
~~~

#### GetTables

~~~cpp
auto GetTables() -> std::vector<Table>;
~~~

- **Zweck:** Liest die Beschreibungen aller von `GetTableNames` gemeldeten Verzeichnisse.
- **Parameter:** Nicht zutreffend.
- **Rückgabe/Ownership:** Eigener `std::vector<Table>`, leer ohne Tabellen; kein Teilergebnis bei `throw`.
- **Vor-/Nachbedingungen:** Regel D; jeder gemeldete Verzeichnisname muss ein lesbares Schema haben. Keine Änderung. Reihenfolge wie Namensauflistung, nicht sortiert zugesagt. Ein fremdes Unterverzeichnis kann den gesamten Aufruf scheitern lassen.
- **Fehler/Garantie:** Wie `GetTableNames` und `GetTable`, Regel D. Stark hinsichtlich Datenbasis (abgeleitet), keine Snapshot-Garantie.
- **Nebenläufigkeit/Kosten:** Regel D; Basisscan plus Metadatenlesung und Sortierung für jede Tabelle, Vektorreservierung und Beschreibungsallokationen.

~~~cpp
auto const descriptions = mrdb::GetTables();
for (auto const & description : descriptions) {
    std::cout << description.Name << '\n';
}
~~~

#### IsTableExist

~~~cpp
auto IsTableExist(std::string const & tablename) -> bool;
~~~

- **Zweck:** Prüft, ob unter dem normalisierten Tabellennamen ein Verzeichnis existiert.
- **Parameter:** `tablename` (`std::string const &`, in), gewünschter Tabellenname. Ungültige Namen werden intern geprüft und führen hier zu `false`.
- **Rückgabe/Ownership:** `bool`; `true` bei einem Verzeichnis, `false` bei fehlendem Verzeichnis **oder jedem intern abgefangenen Fehler**, auch fehlender Initialisierung/Zugriffsproblemen. Kein Ergebnisbesitz, keine Lebensdauerfrage.
- **Vor-/Nachbedingungen:** Für aussagekräftige Verwendung Regel D; technisch wird auch uninitialisierter Zustand abgefangen. Kein Schema-/Lesbarkeitstest, keine Änderung.
- **Fehler/Garantie:** Im Funktionskörper `catch (...)`, daher No-Throw-Verhalten des Körpers aus der Implementierung abgeleitet; **kein deklariertes `noexcept`**. Fehler beim Erzeugen eines Arguments vor Eintritt werden davon nicht erfasst.
- **Nebenläufigkeit/Kosten:** Regel D; Pfad-/Namenskopien und eine Dateisystemprüfung, mögliche Allokation trotz bool-Ergebnis. Nach Rückgabe kann sich der Bestand geändert haben.

~~~cpp
auto const exists = mrdb::IsTableExist(tablename);
std::cout << exists << '\n';
~~~

#### GetRecordIDs

~~~cpp
auto GetRecordIDs(std::string const & tablename) -> std::vector<unsigned int>;
~~~

- **Zweck:** Ermittelt die IDs der Datensatz-Unterverzeichnisse einer Tabelle.
- **Parameter:** `tablename` (`std::string const &`, in), gültige vorhandene Tabelle.
- **Rückgabe/Ownership:** Eigener `std::vector<unsigned int>`, leer ohne Datensätze. Unsortierte Reihenfolge möglich; keine nullable Ergebnisse.
- **Vor-/Nachbedingungen:** Regel D; vom Bestand erwartete ID-Verzeichnisse müssen numerisch lesbar sein. Keine Änderung; vorhandene Zeilen-IDs, kein Integritätstest der Feldwerte.
- **Fehler/Garantie:** Fehlende Tabelle: `ERROR_TABLE_NOT_FOUND` als Runtime-Fehler. Andere Verzeichnisnamen können bei `std::stoul` `std::invalid_argument`/`std::out_of_range` auslösen. Präfixkonvertierung und anschließende Konvertierung auf `unsigned int` sind kein strikter Format-/Bereichstest. Regel D; stark hinsichtlich Datenbasis (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel D; Tabellenscan, ID-Konvertierungen, Ergebnisallokationen proportional zur Zeilenzahl, kein Lesen der Feldinhalte.

~~~cpp
auto const recordIds = mrdb::GetRecordIDs(tablename);
for (auto const id : recordIds) {
    std::cout << id << '\n';
}
~~~

#### SetTableReadOnly

~~~cpp
void SetTableReadOnly(std::string const & tablename, bool readonly);
~~~

- **Zweck:** Schreibt das persistente ReadOnly-Flag einer Tabelle.
- **Parameter:** `tablename` (`std::string const &`, in): gültige vorhandene Tabelle; `readonly` (`bool`, in): `true` sperrt die mrdb-Datensatzschreibaufrufe, `false` hebt das Flag auf.
- **Rückgabe/Ownership:** `void`, nicht zutreffend; keine Übernahme der Eingabe.
- **Vor-/Nachbedingungen:** Regel D, beschreibbare Tabellenmetadaten. Auch bei bereits schreibgeschützter Tabelle zulässig. Nach Erfolg lesen spätere Schreibprüfungen den neuen Wert; ältere `Table`-/Record-Beschreibungen werden nicht aktualisiert. Bei I/O-Fehlern kann das Flag beschädigt oder bereits geändert sein.
- **Fehler/Garantie:** Fehlende Tabelle: Runtime-Fehler mit `ERROR_TABLE_NOT_FOUND`; weitere Fehler nach Regel D. Keine Garantie persistenter Konsistenz bei Teilfehlern.
- **Nebenläufigkeit/Kosten:** Regel D; Pfadbildung, eine Metadatenschreiboperation, kein atomarer Wechsel und keine Betriebssystem-Sperre.

~~~cpp
mrdb::SetTableReadOnly(tablename, true);
mrdb::SetTableReadOnly(tablename, false);
~~~

<a id="datensaetze"></a>

### Schreiben und Lesen von Datensätzen

#### Insert

~~~cpp
void Insert(std::string const & tablename, std::map<std::string, std::string> const &);
~~~

- **Zweck:** Legt einen Datensatz an und schreibt die übergebenen Spaltenwerte.
- **Parameter:** `tablename` (`std::string const &`, in), gültige vorhandene Tabelle. Map (`std::map<std::string, std::string> const &`, in, Definition `values`): nicht leer, Schlüssel vorhandene gültige Spaltennamen, Werte Strings; alle Pflichtspalten angeben, deren Werte weder leer noch nur ASCII-Leerzeichen sein dürfen. Nullable-Spalten dürfen fehlen. Keine zwei Schlüssel verwenden, die auf dieselbe Großschreibung fallen; dies wird nicht eigens zurückgewiesen und kann mehrfaches Überschreiben bewirken.
- **Rückgabe/Ownership:** `void`, insbesondere **keine neue ID** und kein affected-row count. Eingaben werden nicht gehalten, Feldwerte werden gespeichert.
- **Vor-/Nachbedingungen:** Regel D; Tabelle nicht ReadOnly, neue ID muss im Wertebereich verfügbar sein. Nach Erfolg neue Zeile; ausgelassene Nullable-Spalten werden zunächst leer angelegt. Aktuelle ID-Vergabe: vorhandenes Maximum plus eins. Bei I/O-Fehlern kann eine unvollständige Zeile mit bereits geschriebenen Feldern zurückbleiben.
- **Fehler/Garantie:** Runtime-Fehler für fehlende Tabelle, ReadOnly, leere Map, unbekannte Spalte, leeren Pflichtwert, fehlende Pflichtspalte oder ungültigen Schlüssel; Regel D. Validierung vor Anlage, danach kein Rollback. Die anfängliche Anlage leerer Felddateien prüft derzeit nicht jeden Streamfehler; kein zusätzlicher erfolgreicher Vollständigkeits-/Durabilitätstest. Keine basale Garantie konsistenter Persistenz bei Teilfehlern.
- **Nebenläufigkeit/Kosten:** Regel D; ID-Ermittlung nicht atomar. Wiederholte Metadatenlesung, kompletter ID-Scan, Anlage aller Felder und Schreiben der übergebenen Werte; lineare Spaltensuchen je Schlüssel, zusätzliche Suche der Pflichtspalten in der Map, String-/Containerallokationen.

~~~cpp
mrdb::Insert(tablename, {{"NAME", "Zange"}, {"ORT", "Werkstatt"}});
~~~

#### Update mit Tabellenname und ID

~~~cpp
void Update(std::string const & tablename, unsigned int recordID, std::map<std::string, std::string> const & values);
~~~

- **Zweck:** Schreibt die angegebenen Felder einer vorhandenen Zeile; andere Felder bleiben erhalten.
- **Parameter:** `tablename` (`std::string const &`, in): vorhandene Tabelle; `recordID` (`unsigned int`, in): ID einer vorhandenen Zeile, regulär positiv; `values` (Map const-Referenz, in): nicht leer, bekannte Spalten, gültige Schlüssel, Pflichtwerte nicht leer/nur Leerzeichen. Ausgelassene Pflichtspalten sind hier zulässig. Großschreibungskollisionen der Schlüssel vermeiden.
- **Rückgabe/Ownership:** `void`, nicht zutreffend; keine Eingabereferenz gespeichert.
- **Vor-/Nachbedingungen:** Regel D; vorhandene Zeile, Tabelle nicht ReadOnly. Null-ID wird in dieser Variante nicht separat zurückgewiesen, sondern nach Existenz behandelt; nicht als reguläre ID verwenden. Nach Erfolg angegebene Felder ersetzt, ältere gelesene Records bleiben unverändert. Fehler können nach einigen ersetzten Feldern eintreten.
- **Fehler/Garantie:** `"record not found"`, ReadOnly, fehlende Tabelle, leere/ungültige Werte, Regel D. Prüfung vor Schreiben; beim sequenziellen Schreiben kein Rollback, keine Garantie persistenter Konsistenz.
- **Nebenläufigkeit/Kosten:** Regel D; keine Konflikterkennung. Metadaten-I/O, lineare Spaltensuche pro Map-Eintrag und Schreibzugriff je angegebenem Feld; keine Suche über alle Datensätze zur ID-Vergabe.

~~~cpp
mrdb::Update(tablename, recordId, {{"ORT", "Lager"}});
~~~

#### Update mit Record

~~~cpp
void Update(Record const &);
~~~

- **Zweck:** Validiert einen Record und schreibt alle über seine Spaltenbeschreibung abgebildeten Werte mit der vorigen Überladung.
- **Parameter:** `Record const &` (in, Definition `record`): muss die Bedingungen von `isValidRecord` erfüllen; `GetTable()->Name` und `GetRowID()` müssen eine schreibbare vorhandene Zeile in der *aktuell initialisierten* Datenbasis benennen.
- **Rückgabe/Ownership:** `void`, nicht zutreffend; Record und geteilte Tabellenbeschreibung bleiben beim bisherigen Besitzer.
- **Vor-/Nachbedingungen:** Regel D plus gültiger Record. Nach Erfolg gespeicherte Feldwerte entsprechend der erzeugten Map; Record bleibt unverändert. Kein Abgleich mit dem ursprünglich gelesenen Schema oder Versionsstand; spätere Änderungen anderer Schreiber können überschrieben werden.
- **Fehler/Garantie:** `isValidRecord`, Map-Erzeugung und tabellenbezogenes `Update` können werfen; Regel D. Vor Schreibbeginn unveränderte Persistenz, danach keine Garantie persistenter Konsistenz. Keine Transaktion über alle Felder.
- **Nebenläufigkeit/Kosten:** Regel D; erstellt eine eigene Map mit Stringkopien/Allokationen und schreibt deren Felder. `ReadOnly` der geteilten Momentaufnahme allein entscheidet nicht: Das gespeicherte Schema wird erneut gelesen.

~~~cpp
record.SetValue("ORT", "Lager");
mrdb::Update(record);
~~~

#### Delete

~~~cpp
void Delete(std::string const & tablename, unsigned int recordID);
~~~

- **Zweck:** Entfernt eine vorhandene Zeile mitsamt ihrem gespeicherten Inhalt.
- **Parameter:** `tablename` (`std::string const &`, in): gültige vorhandene Tabelle; `recordID` (`unsigned int`, in): vorhandene reguläre positive ID. Null-ID wird hier wie bei `Update` nur über Existenz behandelt.
- **Rückgabe/Ownership:** `void`, kein Löschzähler. Vorhandene Record-Werte des Aufrufers werden nicht freigegeben oder verändert.
- **Vor-/Nachbedingungen:** Regel D, Tabelle nicht ReadOnly. Nach regulärem Abschluss ist die Zeile bei unverändertem Dateisystem entfernt; gelesene Momentaufnahmen bleiben im Speicher. Teilfehler können teilweise entfernte Inhalte hinterlassen. Ein späteres Insert kann die höchste entfernte ID wiederverwenden.
- **Fehler/Garantie:** Fehlende Zeile `"record not found"`, fehlende Tabelle, ReadOnly, Regel D. Kein Rollback, keine Garantie konsistenter Persistenz bei teilweiser Löschung; Rückgabewert der internen Dateisystemlöschung wird nicht als Status ausgegeben.
- **Nebenläufigkeit/Kosten:** Regel D; Metadatenlesung, rekursives Entfernen der Zeileninhalte. Kosten hängen von Dateien/Verzeichnissen und Dateisystem ab.

~~~cpp
mrdb::Delete(tablename, recordId); // Diesen Ausschnitt unabhängig verwenden.
~~~

#### SelectSingle

~~~cpp
auto SelectSingle(std::string const & tablename, unsigned int recordID) -> Record;
~~~

- **Zweck:** Liest eine vorhandene Zeile mit Tabellenbeschreibung und allen Feldwerten ein.
- **Parameter:** `tablename` (`std::string const &`, in): vorhandene gültige Tabelle; `recordID` (`unsigned int`, in): ID vorhandener Zeile. Null-ID nicht separat geprüft, regulär positive IDs verwenden.
- **Rückgabe/Ownership:** `Record` als beweglicher eigener Wert; eigene kopierte Feldwerte und eigener geteilter Tabellenbesitz. Kein `nullptr`-/Optional-Signal für fehlende Zeile.
- **Vor-/Nachbedingungen:** Regel D, Metadaten und alle erwarteten Felder lesbar. Nach Erfolg Werte in sortierter Spaltenreihenfolge. Datenbasis bleibt auch bei Fehler unverändert; kein konsistenter Zeitpunkt über mehrere Dateien zugesagt.
- **Fehler/Garantie:** Fehlende Zeile `"record not found"`; fehlende Tabelle, fehlende/fehlerhafte Feldlesung, Regel D. Stark hinsichtlich Datenbasis (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel D; Metadatenlesen/-sortieren, Zugriff auf jedes Feld, `make_shared<Table>`, Wertvektor und Strings. Der Record-Konstruktor kopiert den Wertvektor trotz des intern übergebenen `std::move`; zusätzliche Kopie der gelesenen Werte.

~~~cpp
auto row = mrdb::SelectSingle(tablename, recordId);
std::cout << row.GetValue("NAME") << '\n';
~~~

#### SelectAll

~~~cpp
auto SelectAll(std::string const & tablename) -> Records;
~~~

- **Zweck:** Liest alle ermittelten Zeilen vollständig ein.
- **Parameter:** `tablename` (`std::string const &`, in): gültige vorhandene Tabelle.
- **Rückgabe/Ownership:** Besitzender `Records`-Vektor, leer ohne Zeilen, nicht nullable; kein Teilergebnis bei Exception.
- **Vor-/Nachbedingungen:** Regel D, alle ermittelten Zeilen lesbar. Reihenfolge wie ID-Auflistung, nicht sortiert zugesagt; keine Zustandsänderung und keine Snapshot-Isolation.
- **Fehler/Garantie:** Wie `GetRecordIDs` und `SelectSingle`; Regel D. Bereits lokal erzeugte Ergebnisse werden bei Fehler automatisch freigegeben. Stark hinsichtlich Datenbasis (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel D; ID-Scan plus `SelectSingle` für jede Zeile. Derzeit wiederholte Schema-I/O und eigene Tabellenbeschreibung pro Zeile; gesamter Bestand im Speicher, Ergebnisvektor kann wachsen/reallokieren.

~~~cpp
auto const rows = mrdb::SelectAll(tablename);
for (auto const & row : rows) {
    std::cout << row.GetRowID() << '\n';
}
~~~

#### Select

~~~cpp
auto Select(std::string const & tablename, WhereVector const &) -> Records;
~~~

- **Zweck:** Liest Zeilen und behält diejenigen, die alle Bedingungen erfüllen (UND).
- **Parameter:** `tablename` (`std::string const &`, in): vorhandene Tabelle; `WhereVector const &` (in, Definition `whereVector`): Bedingungen für vorhandene, exakt geschriebene Feldnamen. Ein leerer Filter akzeptiert alle Zeilen. Filterwert ist beliebiger String; Operator `Equal`, `NonEqual` oder `Like`.
- **Rückgabe/Ownership:** Eigener `Records`-Vektor, leer ohne Treffer; keine nullable Rückgabe, kein Teilergebnis bei Fehler.
- **Vor-/Nachbedingungen:** Regel D. Stringvergleich exakt und case-sensitive; `Like` bedeutet Teilstring, keine SQL-Wildcards, keine regulären Ausdrücke. `Like` mit leerem Suchwert oder leerem Feldwert liefert aktuell `false`. Keine Änderung. Alle Bedingungen werden aktuell für jede Zeile ausgewertet, auch nach einem bereits falschen Ergebnis. Ein Feldfehler in einer späteren Bedingung wird also nicht dadurch unterdrückt. Ohne Zeilen werden Bedingungen nicht geprüft; ein ungültiger Feldname kann dann unbemerkt bleiben. Nicht vorgesehene gecastete Enumwerte ergeben derzeit `false`, keine Exception-Zusage.
- **Fehler/Garantie:** `"field not found in record"` für nicht gefundenes Filterfeld, möglicher Größenfehler; außerdem Lesefehler nach Regel D. Stark hinsichtlich Datenbasis (abgeleitet), keine Zusage konsistent zusammengehöriger gelesener Felder.
- **Nebenläufigkeit/Kosten:** Regel D; Vollscan und vollständiges Einlesen auch nicht passender Zeilen, keine Filterindizes. Je Zeile und Bedingung lineare Spaltensuchen, Wertkopie und Stringvergleich/-suche; Speicher für Treffer plus vorübergehend eine gelesene Zeile und alle IDs.

~~~cpp
auto where = mrdb::WhereVector{};
where.Add("ORT", mrdb::WhereOperator::Like, "Werk");
auto const matches = mrdb::Select(tablename, where);
std::cout << matches.size() << '\n';
~~~

#### isValidRecord

~~~cpp
void isValidRecord(Record const & record);
~~~

- **Zweck:** Prüft die strukturellen Mindestbedingungen eines Records; öffentlich deklariert, im Header als „Temporär“ kommentiert.
- **Parameter:** `record` (`Record const &`, in): auch ein manuell konstruierter Record mit leerem `TablePtr` darf zur Prüfung übergeben werden. Referenz selbst nicht nullable.
- **Rückgabe/Ownership:** `void`, **kein bool**; reguläre Rückkehr bedeutet bestandene Prüfung. Keine Ownership-Änderung.
- **Vor-/Nachbedingungen:** Regel S. Prüft der Reihe nach: nicht leere Tabelle, nicht leerer Tabellenname (nicht vollständige Namensprüfung), mindestens eine Spalte, Zeilen-ID ungleich null, mindestens ein Wert, gleiche Anzahl von Spalten und Werten. Keine Zustandsänderung. Prüft weder persistente Existenz noch `ReadOnly`, Spaltennamen, Pflichtwerte oder Tabellen-ID.
- **Fehler/Garantie:** `std::runtime_error` mit `"table not valid"`, `"table name not valid"`, `"table has no columns"`, `"recordID not valid"`, `"record has no values"` oder `"size of fields and columns not equal"`. Fehlerobjektkonstruktion kann zusätzlich allokieren. Stark hinsichtlich unverändertem Record (abgeleitet), kein `noexcept`.
- **Nebenläufigkeit/Kosten:** Regel S; konstante Zahl von Zustands-/Größenprüfungen, keine Feldkopien oder I/O, temporäre Smartpointerkopien.

~~~cpp
mrdb::isValidRecord(record);
~~~

<a id="record"></a>

### Record

Keine Standardkonstruktion; nicht kopierbar, bewegbar, `final`. Die folgenden Member sind alle öffentlich. Regeln S und die [Lebensdauerregeln](#lifecycle) gelten ergänzend.

#### Expliziter Konstruktor

~~~cpp
Record(unsigned int _rowID, std::vector<std::string> const & _values, TablePtr _tablePtr);
~~~

- **Zweck:** Erstellt einen Record aus ID, Wertliste und geteilter Tabelle.
- **Parameter:** `_rowID` (`unsigned int`, in), beliebige darstellbare ID beim Konstruktor, für gültigen Record positiv; `_values` (Vektor const-Referenz, in), wird kopiert, in Spaltenreihenfolge; `_tablePtr` (`TablePtr`, in), wird per Wert kopiert und geteilt. `nullptr` ist beim Konstruktor akzeptiert, aber für tabellenbezogene Member ungeeignet. Gültiger Record benötigt eine benannte Tabelle mit Spalten, mindestens einen Wert und gleich viele Werte wie Spalten.
- **Rückgabe/Ownership:** Konstruktor, Rückgabe nicht zutreffend. Record besitzt eigene Werte und einen mit weiteren Smartpointern geteilten Tabellenbesitz; Quelle darf nach Konstruktion wegfallen.
- **Vor-/Nachbedingungen:** Keine Initialisierung nötig, keine automatische Validierung. Nach Erfolg übergebene ID/Werte und geteilter Tabellenzeiger. Ungültige Kombinationen werden erst von späteren Prüfungen erkannt oder führen bei ungeprüftem Zugriff zu undefiniertem Verhalten.
- **Fehler/Garantie:** Wert-/Stringkopien können werfen. Kein vollständig konstruiertes Objekt bei Fehler, Eingaben unverändert (stark, abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel S; Wertkopien proportional zur Datenmenge, mögliche Heap-Allokationen; keine Dateien. Übergabe eines temporären Vektors oder `std::move` vermeidet diese Kopie nicht, weil der Parameter eine const-Referenz ist.

~~~cpp
auto description = std::make_shared<mrdb::Table>(table);
auto values = std::vector<std::string>{"Messschieber", "Werkstatt"};
auto local = mrdb::Record{recordId, values, description};
mrdb::isValidRecord(local);
~~~

#### Destruktor

~~~cpp
~Record() = default;
~~~

- **Zweck:** Zerstört die eigenen Werte und gibt den geteilten Tabellenbesitz frei.
- **Parameter/Rückgabe:** Nicht zutreffend; kein Ergebnisbesitz.
- **Vor-/Nachbedingungen:** Lebendes Objekt, keine gleichzeitige Verwendung. Danach alle darauf bezogenen Wertiteratoren/Referenzen ungültig. Andere Smartpointer halten die Tabelle weiter am Leben. Kein Speichern von `SetValue`-Änderungen und kein Löschen einer Datenbankzeile.
- **Fehler/Garantie:** Keine explizite `noexcept`-Schreibweise. No-Throw-Destruktion aus den Default-Standardmembern abgeleitet; kein eigener Fehlerpfad.
- **Nebenläufigkeit/Kosten:** Regel S; Freigabe der Werte, gegebenenfalls zuletzt auch der Tabellenbeschreibung; keine I/O.

~~~cpp
{
    auto local = mrdb::SelectSingle(tablename, recordId);
    std::cout << local.GetRowID() << '\n';
} // Automatische Destruktion, Datenbankzeile bleibt bestehen.
~~~

#### Move-Konstruktor

~~~cpp
Record(Record &&) = default;
~~~

- **Zweck:** Überträgt Werte und Tabellenbesitz in ein neues Objekt; ID wird als skalarer Member übernommen.
- **Parameter:** `Record &&` (in/out, Quelle), gültiger Record als Rvalue; keine Kopie. Keine Nullable-Frage bei der Referenz.
- **Rückgabe/Ownership:** Konstruktor, nicht zutreffend; Ziel besitzt die übertragenen Werte/Smartpointer.
- **Vor-/Nachbedingungen:** Regel S. Nach Erfolg Ziel mit vorherigem Inhalt der Quelle; Quelle ist zerstör-/zuweisbar, ihr tabellenbezogener Zugriff ist ohne neue gültige Zuweisung ungeeignet. Iteratoren vor einem Move für die Beispiele neu beschaffen; keine zusätzliche mrdb-Lebensdauergarantie.
- **Fehler/Garantie:** Nicht ausdrücklich `noexcept` geschrieben. No-Throw-Verhalten dieser defaulteten Member-Moves (Standardallocator) abgeleitet, nicht durch eine zusätzliche Bibliothekszusage.
- **Nebenläufigkeit/Kosten:** Regel S; Übernahme von Vektorspeicher/Smartpointer, keine Kopie aller Strings, keine I/O.

~~~cpp
auto source = mrdb::SelectSingle(tablename, recordId);
auto destination = mrdb::Record{std::move(source)};
std::cout << destination.GetRowID() << '\n';
~~~

#### Move-Zuweisung

~~~cpp
Record & operator=(Record &&) = default;
~~~

- **Zweck:** Ersetzt den Inhalt eines vorhandenen Records durch den Inhalt eines Rvalue-Records.
- **Parameter:** `Record &&` (in/out, Quelle), gültiges Objekt; Ziel ist ebenfalls `in/out`.
- **Rückgabe/Ownership:** `Record &` auf das Ziel, nicht nullable, gültig bis zu dessen Zerstörung; geliehene Referenz, keine Freigabe. Der Zielinhalt wird ersetzt, geteilter Besitz übertragen.
- **Vor-/Nachbedingungen:** Regel S. Nach Erfolg Ziel enthält übertragene Daten; frühere Zieliteratoren nicht weiterverwenden. Quelle besitzt keinen zugesagten nutzbaren Tabellenzustand. Selbst-Move gibt keine Zusage zum Erhalt des Inhalts.
- **Fehler/Garantie:** Kein ausdrücklich geschriebenes `noexcept`; No-Throw-Verhalten der defaulteten Member-Moves mit Standardallocator abgeleitet.
- **Nebenläufigkeit/Kosten:** Regel S; Freigabe bisherigen Zielinhalts und Übernahme der Quelle, keine neue Kopie ihrer Feldstrings, keine I/O.

~~~cpp
auto target = mrdb::SelectSingle(tablename, recordId);
auto source = mrdb::SelectSingle(tablename, recordId);
auto & assigned = (target = std::move(source));
std::cout << assigned.GetRowID() << '\n';
~~~

#### Gelöschte Kopieroperationen

~~~cpp
Record(Record const &) = delete;
Record & operator=(Record const &) = delete;
~~~

**Zweck:** Verbietet Kopierkonstruktion und Kopierzuweisung. Parameter wäre jeweils `Record const &` (in), bei Zuweisung wäre das Ziel `in/out`; tatsächlich ist kein Aufruf zulässig. Rückgabe, Ownership, Vor-/Nachbedingungen eines erfolgreichen Aufrufs, Laufzeitfehler, Exception Safety und Nebenläufigkeit: nicht zutreffend, weil die Verwendung schon bei der Kompilierung scheitert. Eine alternative Kopierfunktion wird nicht bereitgestellt. Für eine unabhängige Zeile erneut `SelectSingle` verwenden oder bewusst den expliziten Konstruktor nutzen.

~~~cpp
static_assert(!std::is_copy_constructible_v<mrdb::Record>);
static_assert(!std::is_copy_assignable_v<mrdb::Record>);
// mrdb::Record copy{record}; // Unzulässig: gelöschter Kopierkonstruktor.
// record = copy;            // Unzulässig: gelöschte Kopierzuweisung.
~~~

#### GetRowID

~~~cpp
auto GetRowID() const -> unsigned int;
~~~

- **Zweck:** Liefert die gespeicherte Zeilen-ID.
- **Parameter:** Nicht zutreffend.
- **Rückgabe/Ownership:** `unsigned int`, skalarer Wert, einschließlich null bei entsprechend manuell konstruiertem Record; kein Speicherbesitz/Lebensdauerproblem.
- **Vor-/Nachbedingungen:** Lebender Record; Tabelle darf leer sein. Keine Änderung.
- **Fehler/Garantie:** Kein eigener Fehlerpfad, No-Throw-Verhalten des Körpers abgeleitet; kein deklariertes `noexcept`.
- **Nebenläufigkeit/Kosten:** Regel S; `O(1)`, keine Allokation/I/O.

~~~cpp
auto const id = record.GetRowID();
std::cout << id << '\n';
~~~

#### GetRowIDAsString

~~~cpp
auto GetRowIDAsString() const -> std::string;
~~~

- **Zweck:** Formatiert die Zeilen-ID mit `std::to_string`.
- **Parameter:** Nicht zutreffend.
- **Rückgabe/Ownership:** Eigener `std::string` mit dezimaler ID, nicht nullable, unabhängig von der Record-Lebensdauer.
- **Vor-/Nachbedingungen:** Lebender Record; keine Tabelle nötig. Keine Änderung.
- **Fehler/Garantie:** Stringerzeugung kann allokieren/werfen; stark hinsichtlich Record (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel S; Formatierung und mögliche Stringallokation, keine I/O.

~~~cpp
auto const idText = record.GetRowIDAsString();
std::cout << idText << '\n';
~~~

#### GetValue mit Index

~~~cpp
auto GetValue(int index) const -> std::string;
~~~

- **Zweck:** Kopiert den Wert an einer Position der eigenen Wertliste.
- **Parameter:** `index` (`int`, in), `0 <= index < GetValueSize()`, zusätzlich durch `int` darstellbar. Negativ oder zu groß wird abgewiesen.
- **Rückgabe/Ownership:** Eigener `std::string`, auch leer zulässig; keine View, kein nullable Zeiger.
- **Vor-/Nachbedingungen:** Lebender Record, keine Tabelle nötig. Keine Änderung bei Erfolg/Fehler.
- **Fehler/Garantie:** `std::out_of_range("Index out of range")` für unzulässigen Index, Stringkopie kann werfen. Stark hinsichtlich Record (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel S; `O(1)` Positionszugriff plus längenabhängige Stringkopie und mögliche Allokation, keine I/O.

~~~cpp
auto const firstValue = record.GetValue(0);
std::cout << firstValue << '\n';
~~~

#### GetValue mit Spaltenname

~~~cpp
auto GetValue(std::string const & columnName) const -> std::string;
~~~

- **Zweck:** Sucht einen Spaltennamen in der geteilten Beschreibung und kopiert den zugeordneten Wert.
- **Parameter:** `columnName` (`std::string const &`, in), exakt mit `Column::Name` übereinstimmend; keine Großschreibungsnormalisierung.
- **Rückgabe/Ownership:** Eigene Stringkopie, leer zulässig, kein geliehener Speicher.
- **Vor-/Nachbedingungen:** Nicht leeres `GetTable()`; vorhandene Spalte mit gültiger Wertposition, Positionsberechnung muss in `int` passen. Keine Änderung. Leere Tabelle wird vor Dereferenzierung nicht geprüft: ungeeigneter Record kann undefiniertes Verhalten auslösen.
- **Fehler/Garantie:** Unbekannter Name: `std::invalid_argument("Column not found")`; unpassende Wertposition: `std::out_of_range("Index out of range")`; Allokationsfehler möglich. Bei erfüllten Vorbedingungen stark hinsichtlich Record (abgeleitet), keine Garantie für ungültigen Tabellenzeiger.
- **Nebenläufigkeit/Kosten:** Regel S; lineare Spaltensuche, Stringvergleiche und Wertkopie/Allokation, keine I/O.

~~~cpp
auto const name = record.GetValue("NAME");
std::cout << name << '\n';
~~~

#### GetTable

~~~cpp
auto GetTable() const -> TablePtr;
~~~

- **Zweck:** Liefert den geteilten Zugriff auf die Tabellenbeschreibung des Records.
- **Parameter:** Nicht zutreffend.
- **Rückgabe/Ownership:** `TablePtr` als Smartpointerkopie; nullable bei manuell leer konstruiertem/verschobenem Record. Geteilter Besitzer; Tabelle lebt bis zum letzten Besitzer. Kein `delete`. Die Tabelle ist durch diesen Pointer veränderbar, auch bei `Record const`.
- **Vor-/Nachbedingungen:** Lebender Record; keine Initialisierung oder nicht leere Tabelle nötig. Record unverändert, temporär weiterer geteilter Besitzer. Keine Aktualisierung aus der Datenbasis.
- **Fehler/Garantie:** Kopie des Standard-`shared_ptr` erzeugt keinen eigenen Kontrollblock; No-Throw-Verhalten abgeleitet, kein deklariertes `noexcept`.
- **Nebenläufigkeit/Kosten:** Regel S; `O(1)` Smartpointerkopie/Referenzzählung, keine Heap-Allokation für eine neue Tabelle oder I/O. Referenzzählung macht die veränderbare Tabelle nicht thread-sicher.

~~~cpp
auto description = record.GetTable();
if (description) {
    std::cout << description->Name << '\n';
}
~~~

#### GetValueSize

~~~cpp
auto GetValueSize() const -> size_t;
~~~

- **Zweck:** Liefert die Anzahl eigener Feldwerte.
- **Parameter:** Nicht zutreffend.
- **Rückgabe/Ownership:** `size_t`, skalar, null bei leerer Wertliste; kein Besitz/Lebensdauerproblem.
- **Vor-/Nachbedingungen:** Lebender Record, keine Tabelle nötig. Keine Änderung.
- **Fehler/Garantie:** Kein Fehlerpfad; No-Throw-Verhalten des Körpers abgeleitet, kein deklariertes `noexcept`.
- **Nebenläufigkeit/Kosten:** Regel S; `O(1)`, keine Allokation/I/O.

~~~cpp
auto const count = record.GetValueSize();
std::cout << count << '\n';
~~~

#### GetValuesAsMap

~~~cpp
auto GetValuesAsMap() const -> std::map<std::string, std::string>;
~~~

- **Zweck:** Erstellt eine Zuordnung von Spaltennamen zu Feldwerten.
- **Parameter:** Nicht zutreffend.
- **Rückgabe/Ownership:** Eigene Map mit eigenen Stringkopien, unabhängig vom Record; Reihenfolge nach dem Standard-Map-Schlüsselvergleich. Keine nullable Rückgabe.
- **Vor-/Nachbedingungen:** Nicht leere Tabelle; für jede Spalte muss ein Wert existieren. Konsistenz am besten vorher durch `isValidRecord` prüfen. Keine Änderung. Zu wenige Werte oder leerer Tabellenzeiger: ungeprüfter Zugriff kann undefiniertes Verhalten auslösen. Zusätzliche Werte ohne Spalte werden ignoriert, doppelte manuell erzeugte Spaltennamen überschreiben einen Map-Eintrag.
- **Fehler/Garantie:** Map-/Stringallokationen können werfen, keine eigene Validierungs-Exception. Bei erfüllten Vorbedingungen stark hinsichtlich Record (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel S; `O(c log c)` Map-Vergleiche bei `c` Spalten plus Textkopien, Map-Knoten und mögliche Stringallokationen; keine I/O.

~~~cpp
mrdb::isValidRecord(record);
auto const values = record.GetValuesAsMap();
std::cout << values.at("NAME") << '\n';
~~~

#### GetValuesAsKeyValues

~~~cpp
auto GetValuesAsKeyValues() const -> KeyValues;
~~~

- **Zweck:** Erstellt eine Liste von Spaltenname/Wert-Paaren in der Reihenfolge der Tabellenbeschreibung.
- **Parameter:** Nicht zutreffend.
- **Rückgabe/Ownership:** Eigener `KeyValues`-Vektor mit eigenen Stringkopien; unabhängig vom Record, nicht nullable.
- **Vor-/Nachbedingungen:** Nicht leere Tabelle und mindestens so viele Werte wie Spalten; `isValidRecord` prüft strenger auf Gleichheit. Keine Änderung. Fehlende Werte führen zu ungeprüftem Iteratordereferenzieren; leerer Tabellenzeiger zu ungeprüftem Pointerzugriff. Zusätzliche Werte werden ignoriert, doppelte Namen bleiben als separate Paare erhalten.
- **Fehler/Garantie:** Vektor-/Stringallokationen können werfen, keine eigene Konsistenzprüfung. Bei erfüllten Vorbedingungen stark hinsichtlich Record (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel S; `O(c)` Paarerzeugung plus Textkopien, Vektor reserviert für alle Spalten, Stringallokationen möglich; keine I/O.

~~~cpp
mrdb::isValidRecord(record);
auto const pairs = record.GetValuesAsKeyValues();
for (auto const & pair : pairs) {
    std::cout << pair.Key << ": " << pair.Value << '\n';
}
~~~

#### SetValue

~~~cpp
void SetValue(std::string const & columnName,std::string const & value);
~~~

- **Zweck:** Ändert einen Feldwert im Record-Speicher; speichert ihn nicht in die Datenbasis.
- **Parameter:** `columnName` (`std::string const &`, in), exakter vorhandener Name; `value` (`std::string const &`, in), beliebiger String einschließlich leer. Record ist `in/out`. Hier keine Prüfung von `Nullable` oder `ReadOnly`.
- **Rückgabe/Ownership:** `void`, nicht zutreffend; Wert wird in den eigenen String kopiert, Eingabe nicht gehalten.
- **Vor-/Nachbedingungen:** Nicht leere Tabellenbeschreibung, vorhandene Wertposition, Index in `int` darstellbar. Nach Erfolg lokaler Wert ersetzt, gespeicherte Dateien unverändert. Wertvektor wird nicht vergrößert; dessen konstante Elementiteratoren bleiben für diesen Aufruf gültig, Zeiger/Views in den geänderten Stringinhalt können ungültig werden.
- **Fehler/Garantie:** Unbekannte Spalte: `std::invalid_argument("Column not found")`; fehlende Position: `std::out_of_range("Index out of range")`; Stringzuweisung kann werfen. Starke Garantie der Stringzuweisung für den Record (abgeleitet, gültige Vorbedingungen). Leerer Tabellenzeiger ungeprüft, kein garantierter Fehlerpfad dafür.
- **Nebenläufigkeit/Kosten:** Regel S; lineare Spaltensuche, Textzuweisung mit möglicher Allokation, keine I/O. Gleichzeitiger Zugriff auf denselben Wert muss ausgeschlossen werden.

~~~cpp
record.SetValue("ORT", "Lager");
std::cout << record.GetValue("ORT") << '\n';
mrdb::Update(record); // Erst dieser gesonderte Aufruf persistiert.
~~~

#### begin und end

~~~cpp
auto begin() const;
auto end() const;
~~~

- **Zweck:** `begin()` liefert den Anfang der eigenen Werte, `end()` den End-Sentinel. Auch auf einem nicht konstanten Record nur konstante Iteration.
- **Parameter:** Jeweils nicht zutreffend.
- **Rückgabe/Ownership:** Deduziert jeweils `std::vector<std::string>::const_iterator`; geliehen, nicht nullable als Pointerkonzept. `end()` nie dereferenzieren; bei leerer Liste sind beide gleich. Keine Speicherfreigabe durch den Aufrufer.
- **Vor-/Nachbedingungen:** Lebender Record, keine Tabelle nötig. Keine Änderung. Iteration gültig bis zu Zerstörung/Ersetzung des zugrunde liegenden Vektors; bei Move-/Zuweisungsoperationen die Vektorregeln beachten und für weiterführende Nutzung neu beschaffen. `SetValue` verändert keine Vektorgröße; Iteratoren bleiben dabei bestehen, der beobachtete Text kann wechseln.
- **Fehler/Garantie:** Kein eigener Fehlerpfad; No-Throw-Verhalten der Vektorzugriffe abgeleitet, kein deklariertes `noexcept`.
- **Nebenläufigkeit/Kosten:** Regel S; je `O(1)`, keine Allokation/I/O; gleichzeitige Wertänderung nicht ungesichert lesen.

~~~cpp
auto const first = record.begin();
auto const last = record.end();
for (auto it = first; it != last; ++it) {
    std::cout << *it << '\n';
}
~~~

<a id="filter"></a>

### Filtertypen aus wherevector.h

Diese Typen sind Teil der von `Select` verwendeten öffentlichen Schnittstelle, obwohl ihre Definitionen im eingebundenen Header stehen. Private Prüfmethoden und der private Bedingungsvektor sind keine API-Einträge.

#### WhereOperator

~~~cpp
enum class WhereOperator
{
    Equal,
    NonEqual,
    Like
};
~~~

| Enumerator | Bedeutung bei `Select` |
| --- | --- |
| `Equal` | Feldwert und Filterwert sind exakt gleich. |
| `NonEqual` | Feldwert und Filterwert sind exakt ungleich. |
| `Like` | Nicht leerer Filterwert ist Teilstring eines nicht leeren Feldwerts. Groß-/Kleinschreibung wird unterschieden. |

Keine weiteren öffentlichen Vergleichsoperatoren. Enumwerte sind keine Fehlercodes. Ein künstlich erzeugter anderer Wert wird von `Add` derzeit nicht validiert und von der aktuellen Auswahl als nicht erfüllt behandelt; keine zugesicherte Erweiterungssemantik.

~~~cpp
auto const operation = mrdb::WhereOperator::NonEqual;
auto where = mrdb::WhereVector{};
where.Add("ORT", operation, "Lager");
~~~

#### WhereItem und operator==

~~~cpp
struct WhereItem
{
    std::string Fieldname;
    WhereOperator Operator{WhereOperator::Equal};
    std::string Value;

    auto operator==(WhereItem const & rhs) const -> bool;
};
~~~

`WhereItem` ist ein öffentliches Aggregat: Defaultinitialisierung ergibt leere Strings und `Equal`. Feldname/Operator/Wert sind öffentliche `in/out`-Felder am eigenen Objekt. Es ist implizit kopier-/bewegbar; Strings gehören dem Wert. Konstruktion/Zuweisung prüfen weder Spaltenexistenz noch Enumwert; keine Dateizugriffe. Für implizite Lebenszyklusoperationen gelten die oben erläuterten Standardmemberregeln wie bei `Column`; keine zusätzlich selbstdeklarierte Lifecycle-Signatur im Header.

- **Zweck des Operators:** Vergleicht **nur `Fieldname` und `Operator`**, nicht `Value`. Diese Gleichheit wird für die Duplikatprüfung in `WhereVector::Add` benutzt.
- **Parameter:** `rhs` (`WhereItem const &`, in), beliebiger gültiger WhereItem; exakter Stringvergleich des Feldnamens.
- **Rückgabe/Ownership:** `bool`; `true` bei gleichen Namen und Operatoren, selbst bei unterschiedlichen Werten. Kein Besitz-/Lebensdauerproblem.
- **Vor-/Nachbedingungen:** Regel S; keine Datenbank erforderlich, keine Zustandsänderung.
- **Fehler/Garantie:** Kein eigener Fehlerpfad, keine Allokation für den Vergleich; No-Throw-Verhalten der verwendeten Vergleiche abgeleitet, kein deklariertes `noexcept`.
- **Nebenläufigkeit/Kosten:** Regel S; Stringvergleich bis zur Namenslänge und Enumvergleich, keine I/O.

~~~cpp
auto const left = mrdb::WhereItem{"ORT", mrdb::WhereOperator::Equal, "Werkstatt"};
auto const right = mrdb::WhereItem{"ORT", mrdb::WhereOperator::Equal, "Lager"};
auto const sameConditionKey = (left == right); // true, trotz anderer Value.
std::cout << sameConditionKey << '\n';
~~~

#### WhereVector: Konstruktor und Destruktor

~~~cpp
WhereVector() = default;
~WhereVector() = default;
~~~

- **Zweck:** Konstruktor erzeugt eine leere Sammlung; Destruktor gibt deren Strings und Vektorspeicher frei.
- **Parameter/Rückgabe:** Jeweils nicht zutreffend; Konstruktion/Destruktion liefern kein Ergebnis.
- **Ownership:** WhereVector besitzt seine Bedingungen selbst. Kein Dateizugriff oder Speichern beim Zerstören, kein manuelles Freigeben.
- **Vor-/Nachbedingungen:** Regel S; Konstruktion benötigt keine Initialisierung, danach `begin() == end()`. Destruktion benötigt ein lebendes, nicht gleichzeitig benutztes Objekt; danach dessen Iteratoren ungültig.
- **Fehler/Garantie:** Leere Standardvektorkonstruktion und Memberdestruktion mit Standardallocator ohne eigenen Fehlerpfad; No-Throw-Verhalten abgeleitet, kein explizites `noexcept`.
- **Nebenläufigkeit/Kosten:** Regel S; Konstruktion ohne Speicherung von Bedingungen, Destruktion proportional zu gespeicherten Bedingungen und ihren Freigaben.

~~~cpp
{
    auto where = mrdb::WhereVector{};
    std::cout << (where.begin() == where.end()) << '\n';
} // Destruktor gibt die Sammlung frei.
~~~

#### WhereVector: gelöschte Kopier- und Moveoperationen

~~~cpp
WhereVector(WhereVector const &) = delete;
WhereVector(WhereVector &&) = delete;
auto operator=(WhereVector const &) -> WhereVector & = delete;
auto operator=(WhereVector &&) -> WhereVector & = delete;
~~~

**Zweck:** Verbietet sowohl Kopier- als auch Moveoperationen. Parameter wären jeweils Referenzen auf Quelle (`in` bei Copy, `in/out` bei Move), bei Zuweisung Ziel `in/out`. Erfolgreiche Rückgabe, Ownership, Vor-/Nachbedingungen zur Laufzeit, Exception Safety, Nebenläufigkeit und Laufzeitkosten: nicht zutreffend. Alle vier Verwendungen sind Kompilierungsfehler. Sammlung lokal konstruieren und per Referenz an `Select` übergeben. Direkte Prvalue-Initialisierung `auto where = mrdb::WhereVector{};` benötigt dank garantierter Copy Elision keinen Move.

~~~cpp
static_assert(!std::is_copy_constructible_v<mrdb::WhereVector>);
static_assert(!std::is_move_constructible_v<mrdb::WhereVector>);
static_assert(!std::is_copy_assignable_v<mrdb::WhereVector>);
static_assert(!std::is_move_assignable_v<mrdb::WhereVector>);
// mrdb::WhereVector copy{where};            // Unzulässig.
// mrdb::WhereVector moved{std::move(where)}; // Unzulässig.
// where = copy;                            // Unzulässig.
// where = std::move(moved);                 // Unzulässig.
~~~

#### WhereVector::Add mit Operator

~~~cpp
void Add(std::string const & fieldname, WhereOperator const whereOperator, std::string const & value);
~~~

- **Zweck:** Fügt eine Filterbedingung hinzu.
- **Parameter:** `fieldname` (`std::string const &`, in): nicht leer und nicht nur ASCII-Leerzeichen; kein Abgleich mit einem Tabellenschema, keine Namens-/Großschreibungsnormalisierung. `whereOperator` (`WhereOperator const`, in): einen der drei genannten Enumeratoren verwenden. `value` (`std::string const &`, in): beliebiger String, leer erlaubt. Sammlung `in/out`.
- **Rückgabe/Ownership:** `void`, nicht zutreffend. Eigene Kopien von Feldname und Wert in der Sammlung; Eingaben dürfen danach wegfallen.
- **Vor-/Nachbedingungen:** Regel S; keine Bedingung mit exakt demselben Feldnamen und Operator vorhanden. Nach Erfolg ein neuer Eintrag am Ende. Derselbe Feldname mit anderem Operator ist erlaubt; anderer Wert bei gleichem Operator macht kein neues zulässiges Duplikat. Bei Reallokation alle bisherigen Iteratoren ungültig; ohne Reallokation bleibt der alte Enditerator trotzdem nicht gültig.
- **Fehler/Garantie:** `std::runtime_error("Fieldname is empty or whitespace")` oder `std::runtime_error("WhereItem already exists")`; String-/Vektorallokationen können werfen. Starke Garantie für Sammlung aus Prüfungen und Standardvektoroperation mit diesem Elementtyp abgeleitet; keine absichtliche partielle Ergänzung.
- **Nebenläufigkeit/Kosten:** Regel S; lineare Duplikatsuche mit Namensvergleichen, Stringkopien; Vektorwachstum kann bestehende Elemente übertragen und allokieren. Kein Dateizugriff.

~~~cpp
auto where = mrdb::WhereVector{};
where.Add("NAME", mrdb::WhereOperator::Like, "Mess");
~~~

#### WhereVector::Add ohne Operator

~~~cpp
void Add(std::string const & fieldname, std::string const & value);
~~~

- **Zweck:** Fügt die Bedingung mit `WhereOperator::Equal` hinzu.
- **Parameter:** `fieldname` und `value` (je `std::string const &`, in) wie zuvor; Sammlung `in/out`. Leerwert erlaubt, leerer/Leerzeichen-Feldname nicht.
- **Rückgabe/Ownership:** `void`, nicht zutreffend; eigene Kopien in der Sammlung.
- **Vor-/Nachbedingungen:** Regel S; noch kein `Equal` mit exakt diesem Feldnamen. Nach Erfolg am Ende ergänzt, Iteratorinvalidierung wie bei der Operatorüberladung.
- **Fehler/Garantie:** Delegiert an die vorige Überladung: gleiche Runtime- und Allokationsfehler, starke Garantie für Sammlung (abgeleitet).
- **Nebenläufigkeit/Kosten:** Regel S; gleiche lineare Duplikatsuche und Allokationskosten, kein Dateizugriff.

~~~cpp
auto where = mrdb::WhereVector{};
where.Add("ORT", "Werkstatt");
~~~

#### WhereVector::begin und end

~~~cpp
auto begin() const;
auto end() const;
~~~

- **Zweck:** `begin()` liefert den Anfang der Bedingungen, `end()` deren End-Sentinel; konstante Iteration in Einfügereihenfolge.
- **Parameter:** Jeweils nicht zutreffend.
- **Rückgabe/Ownership:** Jeweils deduzierter `std::vector<WhereItem>::const_iterator`, geliehen, Pointer-Nullability nicht zutreffend. `end()` nie dereferenzieren; bei leerer Sammlung beide gleich.
- **Vor-/Nachbedingungen:** Lebende Sammlung, keine Initialisierung nötig. Keine Zustandsänderung. Bei `Add` mit Reallokation alle Iteratoren ungültig; ohne Reallokation mindestens bisheriger Enditerator ungültig. Nach Destruktion alle ungültig. Neu beschaffen nach Ergänzung.
- **Fehler/Garantie:** Kein eigener Fehlerpfad; No-Throw-Verhalten der Vektorzugriffe abgeleitet, kein deklariertes `noexcept`.
- **Nebenläufigkeit/Kosten:** Regel S; je `O(1)`, keine Allokation/I/O; kein paralleles `Add` während der Iteration.

~~~cpp
auto where = mrdb::WhereVector{};
where.Add("ORT", "Werkstatt");
auto const first = where.begin();
auto const last = where.end();
for (auto it = first; it != last; ++it) {
    std::cout << it->Fieldname << ": " << it->Value << '\n';
}
~~~

<a id="fehler"></a>

## Fehlerbehandlung und Garantien

### Signalisierung

Schreibaufrufe und Validierung geben `void` zurück und signalisieren erkannte Fehler durch Exceptions. Leseaufrufe geben Werte zurück oder werfen. Es gibt weder Statusobjekt noch `nullptr`-Fehlerresultat für eine Auswahl. Ein leerer `Records`-/ID-/Tabellenvektor kann ein erfolgreiches leeres Ergebnis sein. Nur `IsTableExist` wandelt intern **alle** Exceptions in `false` um; daraus lässt sich die Fehlerursache nicht ablesen.

`Record::GetTable()` kann bei manuell erzeugten oder verschobenen Records einen leeren Smartpointer liefern; das ist kein Dateizugriffsstatus. `GetValue(int)` meldet Bereichsfehler mit `std::out_of_range`; die Namensvariante und `SetValue` unbekannte Namen mit `std::invalid_argument` und unpassende Wertpositionen mit `std::out_of_range`.

Runtime-Fehlermeldungen des Headers sind in den API-Einträgen aufgeführt. Die I/O-Prüfungen können unter anderem fehlende Verzeichnisse/Elemente, nicht zu öffnende Dateien und Fehler beim Lesen, Schreiben oder Schließen melden. Nicht jeder Streamzustand wird vollständig ausgewertet. Keine Aussage „jede Betriebssystemstörung wird erkannt“ oder „reguläre Rückkehr bedeutet ausfallsicher auf dauerhaftem Medium“ ableiten. Die Metadaten- und ID-Konvertierungen können Standard-Konvertierungsexceptions weiterreichen; Allokationen und Dateisystemoperationen ebenfalls ihre Standardexceptions.

Ein sinnvolles äußeres Auffangnetz ist `catch (std::exception const &)`, wie in den vollständigen Beispielen. Meldungstexte sind kein zugesichertes, versioniertes Fehlercodeprotokoll. Automatische Wiederholung nach einem Schreibfehler kann bei `Insert` zusätzliche Zeilen erzeugen; zuerst den tatsächlichen Bestand prüfen.

### Einordnung der Exception Safety

Im untersuchten mrdb-Header und Filterheader ist kein `noexcept` ausdrücklich geschrieben. Die folgenden Kategorien sind **nicht ausdrücklich garantiert; aus der aktuellen Implementierung und den verwendeten Standardoperationen abgeleitet**:

| Operationen | Ableitbare Kategorie und Grenze |
| --- | --- |
| `IsTableExist` | No-Throw-Verhalten des Funktionskörpers durch `catch (...)`; Argumenterzeugung davor kann trotzdem werfen. |
| Skalare Getter, `GetTable` des Records, Iteratorzugriffe, WhereItem-Vergleich | No-Throw-Verhalten der benutzten Operationen; gültige Objekte und externe Synchronisation vorausgesetzt. Keine ergänzte `noexcept`-Deklaration. |
| Defaultete Record-Moves/-Destruktion und WhereVector-Defaultkonstruktion/-Destruktion | No-Throw-Verhalten der konkreten Standardmember abgeleitet; keine manuelle Dateiaktion. |
| Nur lesende Dateiaufrufe und kopierende Record-Getter | Stark bezüglich unveränderter Datenbasis bzw. Record; Ergebnis wird bei Fehler nicht geliefert. Keine Snapshot- oder Vollständigkeitsgarantie für einen parallel geänderten Bestand. |
| Record-/KeyValue-Konstruktion | Bei Fehler kein fertig konstruiertes Ergebnis, Eingaben unverändert; stark bezüglich Eingaben. |
| `Init`, `Record::SetValue`, `WhereVector::Add` | Stark für den jeweiligen bisherigen Speicherzustand aus Stringzuweisung bzw. Prüfungen und Standardcontaineroperationen abgeleitet. |
| Implizite Kopierzuweisungen mehrgliedriger öffentlicher Aggregate | Basal bezüglich gültiger Member; spätere Kopierfehler können nach früheren Memberänderungen eintreten, daher keine allgemeine starke Gesamtgarantie. |
| `CreateTable`, `SetTableReadOnly`, `Insert`, beide `Update`, `Delete` | Vor einer Dateiveränderung können Prüfungen ohne Änderungen scheitern. Nach Beginn der I/O weder starke noch basale Garantie eines konsistenten gespeicherten Bestands ableitbar. Kein Rollback, keine Transaktion, keine atomare Änderung aller betroffenen Dateien. |
| Nicht erfüllte Pointer-/Größen-Vorbedingungen bei ungeprüften Record-Zugriffen | Undefiniertes Verhalten möglich, daher keine Garantie; nicht durch `try/catch` zuverlässig behandelbar. |

RAII schließt lokale Streams und gibt Standardcontainer frei. Das sichert die Ressourcenverwaltung der lokalen Objekte, stellt aber keine Datenbanktransaktion dar. Betriebssystempuffer, Stromausfallsicherheit und ein vollständig erkennbarer Schreibabschluss werden nicht vertraglich beschrieben. Die untersuchten ESTW-Aufrufer ergänzen eigene Sicherungen und Sperren; diese Funktionen gehören zur Anwendung, nicht zu mrdb.

<a id="threads"></a>

## Thread-Sicherheit und Nebenläufigkeit

**Thread-Sicherheit ist nicht garantiert.** Die API enthält keine Mutex-Synchronisation und verwendet einen veränderbaren prozessweit gemeinsamen Initialisierungspfad. Es gibt keine getrennten Datenbankinstanzen, deren Konfiguration unabhängig wäre. `Init` gleichzeitig mit anderen `Init`- oder Dateiaufrufen kann einen Datenrennen-Zugriff auf diesen Zustand verursachen; alle solchen Vorgänge extern koordinieren.

Wenn der Pfad feststeht und während der Zugriffe unverändert bleibt, erscheinen parallele ausschließlich lesende Dateiaufrufe ohne eigenen schreibenden mrdb-Zustand möglich. Das ist eine Implementierungsbeobachtung, keine ausdrückliche Zusage. Fremde Prozesse oder Schreibaufrufe können Dateien während einer Auswahl verändern; eine Zeile wird über mehrere Dateizugriffe gelesen und hat keine Snapshot-Isolation.

Für unabhängige Record-/Filterobjekte ohne gemeinsame mutierbare Daten erscheinen parallele Speicherzugriffe möglich. Bei gemeinsamer unveränderter Tabellenbeschreibung erscheint gleichzeitiges Lesen ebenfalls möglich. Gegen gleichzeitiges `SetValue`, Move-Zuweisung, Destruktion, `WhereVector::Add` oder strukturelle Mutation einer geteilten `Table` muss jedoch extern synchronisiert werden. `shared_ptr` schützt die Besitzverwaltung seines Kontrollblocks, nicht die Felder des `Table`-Objekts oder ein gleichzeitig verändertes Record-Objekt.

Gleichzeitiges Einfügen kann dieselbe nächste ID berechnen. Existenzprüfung und Anlage einer Tabelle sind getrennt. Ein ReadOnly-Wechsel sperrt keine bereits laufende Schreiboperation. Updates mehrerer Dateien können sich überlagern, und Lesen während Schreiben/Löschen kann unvollständige oder gemischte Werte liefern bzw. Exceptions auslösen. Für solche Abläufe ist externe Thread-/Prozesskoordination erforderlich; mrdb liefert keine entsprechende Sperr-API. Die `tools`-Namensprüfung verwendet zusätzlich eine statische unveränderbare Menge reservierter Namen; diese ist keine Synchronisation der Datenbasis.

<a id="performance"></a>

## Performance und Ressourcen

Die beobachtete Speicherung verursacht zahlreiche kleine Dateizugriffe. Das Dateiformat und die interne Ablagestruktur sind kein stabil zugesicherter API-Vertrag; Anwendungen sollen die öffentliche API verwenden. Insbesondere wird hier keine direkte Bearbeitung interner Metadaten als unterstützte API beschrieben.

| Ablauf | Belastbar aus der Implementierung ableitbare Kosten |
| --- | --- |
| `GetRecordIDs` | Ein Durchlauf durch die Tabelleneinträge; Ergebnis enthält `r` IDs mit entsprechendem Speicherbedarf. |
| `GetTable` | Liest Spaltenbeschreibungen, sortiert `c` Spalten mit `O(c log c)` Vergleichen. Stringlängen und Dateisystemkosten kommen hinzu. |
| `Insert` | Liest Schema mehrfach, sucht Spalten/Pflichtfelder, scannt die vorhandenen IDs und legt Felder an; kein konstanter Aufwand unabhängig von Tabellenumfang. |
| `SelectSingle` | Metadatenlesung und alle `c` Feldlesungen, neue Tabellenbeschreibung und String-/Vektorspeicher; zusätzliche Wertkopie im Konstruktor. |
| `SelectAll` | ID-Scan und vollständige Zeilenlesung für alle `r` Zeilen, wiederholte Schemainlesung/-sortierung; Ergebnis hält Werte und derzeit je Zeile eine Tabellenbeschreibung. |
| `Select` | Wie vollständiger Zeilenscan, zusätzlich alle `w` Bedingungen je Zeile; keine Indexsuche. Nur Treffer bleiben als Records im Ergebnis. |
| `GetValue(name)`, `SetValue` | Lineare Spaltensuche in der Beschreibung plus Textkopie/-zuweisung. |
| `GetValuesAsMap` | `O(c log c)` Schlüsselvergleiche plus Kopien der Namen/Werte und Map-Knotenallokationen. |
| `GetValuesAsKeyValues` | `O(c)` Paarerzeugung plus Stringkopien; eine passende Vektorkapazität wird reserviert. |
| `WhereVector::Add` | Lineare Duplikatsuche, mögliche Vektorreallokation und Stringkopien. |

Diese Angaben sind keine Gesamtzeitgarantien; Dateisystem, Textlängen und Fehlerpfade beeinflussen die Kosten. Es gibt keinen Schema-Cache, keine Abfrageplanung, keine Paging- oder Streaming-Auswahl in der öffentlichen API. Auch kleine Teilupdates müssen das aktuelle Schema lesen. `Record`-Getter für Werte liefern Kopien, keine zero-copy Views.

Tabellenbeschreibungen, Records, Maps und Filter halten Heap-Speicher über Standardcontainer/Smartpointer. `Records`-Vektorwachstum kann Referenzen auf Record-Objekte invalidieren. `Record::SetValue` lässt dagegen den Wertvektor unverändert und kann nur den Stringinhalt umallokieren. Filterergänzungen können alle geliehenen Bedingungsiteratoren invalidieren. Smartpointerkopien erhalten die Tabelle unabhängig vom ursprünglichen Record am Leben, bieten aber keine unveränderliche Momentaufnahme, sobald jemand ihre öffentlichen Felder mutiert.

<a id="praxisbeispiele"></a>

## Praxisbeispiele

### Vollständiger Arbeitsablauf: anlegen, lesen, filtern, ändern, schützen, löschen

Das Programm benötigt ein leeres Verzeichnis `./mrdb-workflow` oder einen noch freien Tabellennamen darin. Eine bereits vorhandene Tabelle führt bewusst zu einem Fehler statt zu Änderungen an einem unbekannten Schema. Die ermittelte ID wird aus der Auswahl gelesen; es wird nicht angenommen, dass sie eins ist. Das Beispiel setzt alleinigen Zugriff voraus.

~~~cpp
#include "mrdb.h"

#include <exception>
#include <filesystem>
#include <iostream>
#include <stdexcept>
#include <string>

auto main() -> int
{
    try {
        std::filesystem::create_directories("./mrdb-workflow");
        mrdb::Init({"./mrdb-workflow/"});

        auto const tablename = std::string{"WERKZEUGE_WORKFLOW"};
        if (mrdb::IsTableExist(tablename)) {
            throw std::runtime_error("Für dieses Beispiel einen freien Tabellennamen verwenden");
        }

        auto table = mrdb::Table{};
        table.Name = tablename;
        table.Columns = {{"01", "NAME", false}, {"02", "ORT", true}};
        mrdb::CreateTable(table);

        auto const description = mrdb::GetTable(tablename);
        std::cout << "Spalten: " << description.Columns.size() << '\n';

        mrdb::Insert(tablename, {{"NAME", "Messschieber"}, {"ORT", "Werkstatt"}});
        auto filter = mrdb::WhereVector{};
        filter.Add("NAME", "Messschieber");
        auto records = mrdb::Select(tablename, filter);
        if (records.size() != 1) {
            throw std::runtime_error("Genau ein Werkzeug erwartet");
        }

        auto & record = records.front();
        mrdb::isValidRecord(record);
        auto const recordId = record.GetRowID();
        record.SetValue("ORT", "Lager");
        mrdb::Update(record);

        mrdb::Update(tablename, recordId, {{"ORT", "Messraum"}});
        auto const reloaded = mrdb::SelectSingle(tablename, recordId);
        for (auto const & pair : reloaded.GetValuesAsKeyValues()) {
            std::cout << pair.Key << ": " << pair.Value << '\n';
        }
        // record ist eine ältere lokale Momentaufnahme und enthält noch "Lager".

        mrdb::SetTableReadOnly(tablename, true);
        if (!mrdb::GetTable(tablename).ReadOnly) {
            throw std::runtime_error("ReadOnly-Flag nicht gelesen");
        }
        mrdb::SetTableReadOnly(tablename, false);
        mrdb::Delete(tablename, recordId);

        std::cout << "Verbleibende Zeilen: "
                  << mrdb::SelectAll(tablename).size() << '\n';
        // Die nun leere Tabelle bleibt erhalten; es gibt keine DropTable-API.
        return 0;
    }
    catch (std::exception const & error) {
        std::cerr << "Arbeitsablauf fehlgeschlagen: " << error.what() << '\n';
        // Bereits erfolgreiche Dateiänderungen werden dadurch nicht zurückgerollt.
        return 1;
    }
}
~~~

### Fehlerbehandlung: Pflichtwert ablehnen und gültige Eingabe speichern

Dieses eigenständige Programm legt eine eigene Tabelle an. Der erste `Insert` scheitert bei der Werteprüfung vor der Zeilenanlage, weil `NAME` Pflichtfeld ist. Nach dem erwarteten Fehler wird ein gültiger Datensatz eingefügt. Andere Fehler werden im äußeren Catch behandelt. Voraussetzung ist ein freier Tabellenname und unveränderter Bestand während des Beispiels.

~~~cpp
#include "mrdb.h"

#include <exception>
#include <filesystem>
#include <iostream>
#include <stdexcept>
#include <string>

auto main() -> int
{
    try {
        std::filesystem::create_directories("./mrdb-fehlerbeispiel");
        mrdb::Init({"./mrdb-fehlerbeispiel/"});

        auto const tablename = std::string{"PFLICHTFELD_BEISPIEL"};
        auto table = mrdb::Table{};
        table.Name = tablename;
        table.Columns = {{"01", "NAME", false}};
        mrdb::CreateTable(table); // Wirft auch bei bereits vorhandenem Namen.

        auto const before = mrdb::GetRecordIDs(tablename);
        try {
            mrdb::Insert(tablename, {{"NAME", ""}});
            std::cerr << "Pflichtfeldprüfung hat unerwartet keinen Fehler gemeldet\n";
            return 1;
        }
        catch (std::runtime_error const & error) {
            std::cout << "Einfügen abgelehnt: " << error.what() << '\n';
        }

        if (mrdb::GetRecordIDs(tablename) != before) {
            throw std::runtime_error("Datensatzbestand nach Ablehnung unerwartet verändert");
        }

        mrdb::Insert(tablename, {{"NAME", "Messschieber"}});
        auto const records = mrdb::SelectAll(tablename);
        if (records.empty()) {
            throw std::runtime_error("Gespeicherten Datensatz erwartet");
        }

        try {
            std::cout << records.front().GetValue(-1) << '\n';
        }
        catch (std::out_of_range const & error) {
            std::cout << "Indexfehler: " << error.what() << '\n';
        }
        return 0;
    }
    catch (std::exception const & error) {
        std::cerr << "Datenzugriff fehlgeschlagen: " << error.what() << '\n';
        return 1;
    }
}
~~~

Der Bestandstest ist eine zusätzliche Prüfung des Beispiels. Er ist kein Beleg für einen allgemeinen Rollback bei I/O-Fehlern. Ein `std::runtime_error` allein identifiziert auch nicht zweifelsfrei einen Pflichtfeldfehler; dafür existiert kein eigener Fehlertyp.

<a id="quellen"></a>

## Quellen und Prüfstand

Die untersuchten lokalen Quellen liegen in zwei Repositories: der Bibliothekscode unter `D:/csharp/CPP/tools/`, die Pages-Quellen unter `D:/git/mathias/`. `mrdb.md` im Wurzelverzeichnis des Pages-Repositories ist die Quelle der Einstiegsseite `mrdb.html`. Deren Navigation verwendet relative `.md`-Links; diese Seite folgt derselben Struktur und liegt daneben. Die Konfiguration `_config.yml` enthält keine aktive abweichende Permalink- oder Ausgabevorgabe. Die beiden Seiten verweisen gegenseitig aufeinander.

Maßgebliche Dateien und ergänzend untersuchte Verwendungen:

- `CPP/tools/mrdb.h`: öffentliche Deklarationen, Record und sämtliche Implementierungen; API-Quelle dieser Referenz.
- `CPP/tools/wherevector.h`: Filtertypen, Add-Überladungen und Iteratoren.
- `CPP/tools/tools.h`, `CPP/tools/tools.cpp`: Link-Abhängigkeit, Namensnormalisierung/-prüfung, Leerwert- und Teilstringsemantik.
- `CPP/tools/mrdbtest/CMakeLists.txt`, `main.cpp`: C++20 und CRUD-/Filterverwendungen; ältere kommentierte Abschnitte mit anderen Typen/Methoden sind keine Quelle zusätzlicher aktueller API.
- `CPP/tools/kvs.h`: Nutzung über `Init`, Filter und CRUD; dieser Wrapper kann den gemeinsamen Pfad neu setzen.
- `CPP/estw/estw3/mrdbstore.cpp`, `tests/mrdb_tests.cpp`, `datensatzdetails.cpp`: Schemaeinrichtung, Wertezugriff, Teilupdates und ReadOnly-Verwendung. Anwendungseigene Sperren, Transaktionssicherungen und Datentypkonvertierungen werden nicht mrdb zugeschrieben.

Die Bibliothek wurde für diese Dokumentation nicht verändert; der SHA-256-Vergleich von `mrdb.h` vor und nach der Bearbeitung stimmt überein. Alle 45 expliziten öffentlichen Funktions-/Member-Signaturen aus den beiden Headern wurden statisch mit der Referenz abgeglichen, einschließlich Überladungen und gelöschter Operationen. Die Beispielaufrufe wurden gegen die vorhandenen API-Namen geprüft; es gibt drei vollständige Programme. Alle 25 relativen Markdown-Datei-/Abschnittslinks beider Seiten wurden auf vorhandene Ziele geprüft, ohne Fehler. Auch Codeblockbegrenzungen und doppelte explizite Anker wurden geprüft.

Entsprechend der Projektanweisung wurden keine Builds und keine Beispielkompilierungen ausgeführt. Ein lokaler Jekyll-Build wurde nicht ausgeführt; im untersuchten Dokumentationsrepository sind kein Gemfile und kein lokaler Buildworkflow vorhanden. Die Beispiele sind vollständige Programme oder mit dem angegebenen Setup direkt verwendbare Ausschnitte; erfolgreiche Kompilierung und Laufzeit sind damit noch nicht nachgewiesen.

Zurück zur [mrdb-Startseite](mrdb.md).
{% endraw %}
