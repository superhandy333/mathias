# mrdb
Mathias-Rentsch-Daten-Bank

## Overview

mrdb ist eine Funktionssammlung in C++, die es ermöglicht, Daten in Textdateien zu speichern und auszulesen. Sie richtet sich an Entwickler, die strukturierte Daten effizient persistieren und später wieder einlesen möchten, ohne auf externe Bibliotheken angewiesen zu sein.

## Integration

Die Funktionen sind im Namespace `mrdb` implementiert und können leicht in bestehende Projekte eingebunden werden. Dazu sind einfach die Dateien `mrdb.h` und `mrdb.cpp` in das Projektverzeichnis zu kopieren. Die Headerdatei wird inkludiert:

~~~cpp
#include "mrdb.h"
~~~

Falls es sich um ein cmake-Projekt handelt, ist die cpp-Datei in der Steuerdatei anzumelden:

~~~txt
cmake_minimum_required(VERSION 3.22.0)

project(MyProject)
add_compile_options(-std=c++17)
...

add_executable(MyProject
main.cpp
mrdb.cpp
...
)
~~~

## Datentypen

mrdb unterstützt nur einen Datentyp: `std::string`.

## Start

Vor dem ersten Aufruf der Funktionen ist die Datendank zu initialisieren. Die Struktur `InitData` enthält momentan nur einen Wert: `NAME_DATABASE_DIRECTORY`. Darin ist der Pfad zu den Datendateien anzugeben. Die Pfadangabe muss mit einem `/` enden.

~~~cpp
mrdb::InitData data;
data.NAME_DATABASE_DIRECTORY = "C:/Daten/";
mrdb::Init(data);
~~~

## Autor

Mathias Rentsch<br>
rentsch@online.de

