# mrdb

Mathias-Rentsch-Daten-Bank

## Überblick

mrdb ist eine C++-Bibliothek zum Speichern und Lesen von Tabellen in Textdateien. Sie bietet benannte Spalten, Datensätze mit numerischer ID und Filter für Gleichheit, Ungleichheit und Teiltexte. Alle Feldwerte sind `std::string`; ein externer Datenbankserver ist nicht erforderlich.

Das [Anwenderhandbuch mit vollständiger API-Referenz](mrdb-dokumentation.md) erklärt die öffentlichen Typen und Funktionen anhand konkreter C++-Beispiele. Das [verlinkte Funktionsverzeichnis](mrdb-dokumentation.md#funktionsverzeichnis) führt direkt zu den einzelnen Detailbeschreibungen.

## Einbindung

```cpp
#include "mrdb.h"
```

`mrdb.h` ist Header-only; eine `mrdb.cpp` ist nicht erforderlich. Die mitgelieferten Header `wherevector.h` und `tools.h` müssen verfügbar sein. Zusätzlich muss `tools.cpp` mit dem eigenen Programm kompiliert und gelinkt werden. Behalte die mitgelieferte Verzeichnisanordnung der Header bei und ergänze das Verzeichnis von `mrdb.h` im Include-Suchpfad. Die Beispiele im Handbuch verwenden C++20.

## Erste Schritte

Lege ein eigenes Datenverzeichnis an und initialisiere mrdb mit einem Pfad, der mit `/` endet:

```cpp
#include "mrdb.h"

#include <filesystem>

void prepareDatabase()
{
    std::filesystem::create_directories("./projektdaten");
    mrdb::Init({"./projektdaten/"});
}
```

Der [vollständige Schnelleinstieg](mrdb-dokumentation.md#schnelleinstieg) zeigt das Anlegen einer Tabelle, das Schreiben und Ändern von Projekten sowie das anschließende Lesen. Weitere Beispiele stehen unter [typische Anwendungsfälle](mrdb-dokumentation.md#praxisbeispiele); Hinweise zu Exceptions unter [Fehlerbehandlung](mrdb-dokumentation.md#fehler).

## Autor

Mathias Rentsch<br>
rentsch@online.de
