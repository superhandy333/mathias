# mrdb
Mathias-Rentsch-Daten-Bank

## Overview

mrdb ist eine Funktionssammlung in C++, die es ermöglicht, Daten in Textdateien zu speichern und auszulesen. Sie richtet sich an Entwickler, die strukturierte Daten effizient persistieren und später wieder einlesen möchten, ohne auf externe Bibliotheken angewiesen zu sein.

Die [ausführliche Dokumentation](mrdb-dokumentation.md) beschreibt das Datenmodell, die vollständige öffentliche API, Fehlerbehandlung, Lebensdauer, Nebenläufigkeit und Praxisbeispiele für den aktuellen Header `CPP/tools/mrdb.h`.

## Integration

Die Funktionen sind im Namespace `mrdb` implementiert. Der aktuelle Stand unter `CPP/tools/` ist Header-only: Eine `mrdb.cpp` ist dafür nicht erforderlich. Benötigt werden auch `wherevector.h`, `tools.h` und die Implementierung `tools.cpp`. Behalte die Verzeichnisstruktur der Header bei. Die Headerdatei wird inkludiert:

~~~cpp
#include "mrdb.h"
~~~

Für C++20 kann ein Projekt im Verzeichnis `CPP/tools/meinprojekt/` so eingebunden werden:

~~~cmake
cmake_minimum_required(VERSION 3.28)
project(MyProject LANGUAGES CXX)

add_executable(MyProject main.cpp ../tools.cpp)
target_compile_features(MyProject PRIVATE cxx_std_20)
target_include_directories(MyProject PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/..")
set_target_properties(MyProject PROPERTIES CXX_EXTENSIONS OFF)
~~~

## Datentypen

mrdb unterstützt nur einen Datentyp: `std::string`.

## Start

Vor Dateizugriffen ist die Datenbank zu initialisieren. Die Struktur `InitData` enthält momentan nur einen Wert: `NAME_DATABASE_DIRECTORY`. Darin ist der Pfad zu den Datendateien anzugeben. Die Pfadangabe muss mit einem `/` enden; das Basisverzeichnis muss bereits existieren. `Init` speichert lediglich den prozessweit gemeinsamen Pfad und legt kein Verzeichnis an.

~~~cpp
mrdb::InitData data;
data.NAME_DATABASE_DIRECTORY = "C:/Daten/";
mrdb::Init(data);
~~~

Ein vollständiger [Schnelleinstieg](mrdb-dokumentation.md#schnelleinstieg) und weitere [Praxisbeispiele](mrdb-dokumentation.md#praxisbeispiele) stehen in der Langdokumentation.

## Autor

Mathias Rentsch<br>
rentsch@online.de

