# MariaDbConnector

`C++ Connector` ist eine von MariaDB bereitgestellte C++-Bibliothek, die es ermöglicht, auf MariaDB-Datenbanken zuzugreifen.

`MariasDbConnector` ist eine eigene C++-Klasse für die Kapselung von Funktionen der Bibliothek `C++ Connector`. Sie ermöglicht die einfache Anbindung an eine MariaDB-Datenbank und die Durchführung von SQL-Abfragen.

## Integration

### Installation der Bibliothek

Lade den MariaDB `Connector C++ (Windows 64-bit x86)` von der offiziellen [Website](https://mariadb.com/downloads/connectors/connectors-data-access/cpp-connector) herunter und installiere ihn.

Beispiel-Pfad

~~~
c:/Program Files/MariaDB/MariaDB C++ Connector 64-bit
~~~

### Einbindung in cmake

<!-- <details>
<summary>CMakeLists.txt</summary> -->

~~~
cmake_minimum_required(VERSION 4.1)
project(maria)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_COMPILER clang++)
set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_CXX_SCAN_FOR_MODULES OFF)

add_executable(maria
main.cpp
../maria.cpp      # <<-- Hier die eigene Klasse einbinden
../../tools.cpp
)

set(MARIADB_CPP_ROOT "c:/Program Files/MariaDB/MariaDB C++ Connector 64-bit")

find_path(MARIADB_CPP_INCLUDE
  NAMES mariadb/conncpp.hpp
  PATHS "${MARIADB_CPP_ROOT}/include"
  NO_DEFAULT_PATH
)

if(NOT MARIADB_CPP_INCLUDE)
   message(FATAL_ERROR "Include-Pfade nicht gefunden. Prüfe MARIADB_C_ROOT / MARIADB_CPP_ROOT.")
endif()

target_include_directories(maria PRIVATE
  "${MARIADB_CPP_INCLUDE}"
)

find_library(MARIADB_CPP_LIB
  NAMES mariadbcpp mariadbcpp-static
  PATHS "${MARIADB_CPP_ROOT}"
  NO_DEFAULT_PATH
)

if(NOT MARIADB_CPP_LIB)
  message(FATAL_ERROR "Libs nicht gefunden. Prüfe lib/lib64 unter den Connector-Installationen.")
endif()

target_link_libraries(maria PRIVATE "${MARIADB_CPP_LIB}")

# ---- DLLs zur Laufzeit automatisch neben die EXE kopieren
set(MARIADB_CPP_DLL "${MARIADB_CPP_ROOT}/mariadbcpp.dll")

add_custom_command(TARGET maria POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
          "${MARIADB_CPP_DLL}"
          "$<TARGET_FILE_DIR:maria>/mariadbcpp.dll"
  VERBATIM
)
~~~
<!-- </details> --> 
<!-- Dieses Tag wird vom Konverter nicht korrekt umgesetzt. Daher ist es hier auskommentiert. Schade.-->

## In C++ inkludieren

Inkludiere die Header-Datei `maria.h` in dein C++-Projekt:

~~~cpp
#include "maria.h"
~~~

## Verbindung zur Datenbank

~~~cpp
maria::ConnectionConfig config;
config.Host     = "192.168.xxx.xxx";
config.Port     = 3306;
config.Database = "firma";
config.User     = "test";
config.Password = "abcd1234";

maria::MariaDbConnector conn(config);
conn.Connect();
std::cout << "Verbunden: " << (conn.isConnected() ? "ja" : "nein") << std::endl;
~~~

## Einfache Abfrage

~~~cpp
std::string query = "SELECT * FROM personen";
auto table = conn.ExecuteQuery(query);
for (auto const & row : table->Rows) {
    auto id = row.getInt("id");
    auto vorname = row.get("vorname");
    auto nachname = row.get("nachname");
    auto kennzahl = row.getInt("kennzahl");
 
    std::cout << "ID: " << id << ", Vorname: " << vorname << ", Nachname: " << nachname << ", Kennzahl: " << kennzahl << std::endl;
}
~~~

## Abfrage mit Prepared Statement

~~~cpp
std::string query = "SELECT * FROM personen WHERE id > ?";
auto table = conn.ExecuteQuery(query, 1);

...
~~~

## Autor

Mathias Rentsch<br>
rentsch@online.de  
<small>Juni 2026</small>