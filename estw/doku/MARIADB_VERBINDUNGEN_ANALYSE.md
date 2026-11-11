# Mehrere MariaDB-Verbindungen: Anforderungs- und Erweiterungsanalyse

Stand: 11. November 2026. Untersuchte Revision: `29ab5bfb` mit dem vorhandenen Arbeitsverzeichnis.

## 1. Ergebnis und Abgrenzung

Die Erweiterung lässt sich auf Konfigurationsmodell, Konfigurationsspeicherung, Setup und die zentrale Auflösung der aktiven Verbindung begrenzen. Die vorhandenen MariaDB-Lese- und Schreiboperationen benötigen weiterhin jeweils genau einen Satz Verbindungsdaten. Mehrere gespeicherte Verbindungen erfordern weder parallele Datenbankverbindungen noch einen Verbindungspool oder Änderungen an den MariaDB-Fachtabellen.

Folgende Befunde sind für den späteren Auftrag besonders wichtig:

1. Das als **Setup** angezeigte Fenster wird durch **`ConfigForm`** implementiert. Eine Klasse `SetupForm` existiert im untersuchten Projekt nicht. Der tatsächliche Enumwert heißt **`DataSource::MariaDB`**, nicht `DataSource::MariaDb`.
2. Ein Button **Speichern** existiert bereits. Er übernimmt die Eingaben und schließt das Fenster. Dauerhaft gespeichert wird erst anschließend durch `setup()` in `main.cpp`. Das ist für eine Verwaltung mehrerer Einträge und die Behandlung von Speicherfehlern zu berücksichtigen.
3. Zugangsdaten liegen im lokalen, dateibasierten KVS unter `d:/rentsch/database/`, nicht in einer INI-Datei, in `config.toml` oder in MariaDB.
4. Das Passwort wird unverändert gespeichert, in gewöhnlichen Strings verarbeitet und nach erfolgreichem Setup über `showConfig()` auf der Konsole ausgegeben. Das Passwortfeld ist derzeit auch nicht maskiert.
5. Ein erfolgreicher Setup-Abschluss leert bereits den geladenen Datenbestand. Diese Eigenschaft muss auch beim Wechsel zwischen zwei MariaDB-Verbindungen erhalten bleiben.
6. **Keine Auswahl nach Löschen ist bereits festgelegt**, auch wenn andere Verbindungen verbleiben. Offen ist dagegen, wie ein solcher Zustand dauerhaft übernommen und für die weiterhin ausgewählte Datenquelle MariaDB behandelt wird.
7. Die vorhandene Änderungsprüfung vergleicht das gesamte Setup mit seinem Öffnungszustand. Für Neu/Löschen ist zusätzlich ein Vergleich der fünf Verbindungsfelder mit dem zuletzt geladenen beziehungsweise zurückgesetzten Feldstand nötig.

**Kennzeichnung im Bericht:** „Befund“ bezeichnet untersuchten Code oder bestehende Dokumentation; „Anforderung“ bezeichnet die vorgegebene spätere Funktion; „Vorschlag“ bezeichnet eine noch nicht beschlossene Umsetzung. Alle Zielmodelle und Speicherformate sind Vorschläge.

Dieser Auftrag erstellt ausschließlich diesen neuen Bericht. Code, vorhandene Dokumentation, gemeinsame Tools, MkDocs-Konfiguration und produktive Konfigurationsdaten werden nicht geändert. Es werden keine Datenbankverbindungen aufgebaut, keine gespeicherten Zugangsdaten ausgelesen und keine Builds oder Anwendungstests gestartet.

## 2. Relevante Fundstellen

Zeilenangaben beziehen sich auf den untersuchten Stand. Links öffnen die zugehörige Quelldatei; die Zeilen stehen daneben.

| Bereich | Fundstelle | Bedeutung |
| --- | --- | --- |
| Konfigurationsmodell | [structs.h](structs.h), 129–156 | `ConnectionData`, `DataSource`, `ConfigData` |
| Quellenwahl | [datasource.h](datasource.h), 15–23 | Auswahltexte und Schreibfreigabe für MariaDB/mrdb |
| Setup-Klasse | [configform.h](configform.h), 9–53 | `ShowModal`, Öffnungs- und Ergebnisstand, Controlnamen |
| Dialogaufbau | [configform.cpp](configform.cpp), 112–246 | feste Fenstergröße, Quellenwahl, fünf Felder, Verbindungstest |
| Speichern/Schließen | [configform.cpp](configform.cpp), 248–316 | Übernahme, Portprüfung, Verwerfungsabfrage, Rückgabe |
| Änderungsprüfung/Sichtbarkeit | [configform.cpp](configform.cpp), 326–375 | Vergleich mit `m_initialData`, virtuelle Panels |
| KVS-Schlüssel und Laden/Speichern | [main.cpp](main.cpp), 52–59 und 234–270 | genau ein gespeicherter Verbindungssatz |
| Passwortausgabe/Setup-Koordination | [main.cpp](main.cpp), 271–297 | `showConfig()`, Persistenz nach Dialogende |
| Start/Ladecache/Setup-Abschluss | [main.cpp](main.cpp), 303–410 | verzögertes Laden, Fehleranzeige, vollständiger Cache-Reset |
| Verbindungsanzeige | [mainform.cpp](mainform.cpp), 136–163 | Host/Port der einen Verbindung |
| Verbindungstest | [aktionen.cpp](aktionen.cpp), 517–530 | eigene kurzlebige Verbindung, keine Fachabfragen |
| MariaDB-Lesen | [aktionen.cpp](aktionen.cpp), 559–665 und 703–742 | sechs Tabellen, Verbindung aus `ConfigData.Connection` |
| Umwandlung der Zugangsdaten | [aktionen.cpp](aktionen.cpp), 811–819 | `makeConnectionConfig()` für Schreibpfade |
| MariaDB-Schreiben | [aktionen.cpp](aktionen.cpp), 1159–1252, 1406–1452 und 1501–1535 | Elemente, Fahrstraßen und Projekttexte |
| Aufrufer/Detailbearbeitung | [main.cpp](main.cpp), 61–169; [datensatzdetails.cpp](datensatzdetails.cpp), 97–108 und 209–284 | Weitergabe derselben Konfiguration bis zum Speichern |
| Vorhandenes UI-Vergleichsmuster | [projektform.cpp](projektform.cpp), 155–196 | Änderungsvergleich, Buttonaktualisierung, neuer Vergleichsstand erst nach Erfolg |
| Quellübergreifende Ansichtsschlüssel | [lupeform.cpp](lupeform.cpp), 996–1036 | nur Projekt-ID im KVS-Schlüssel |
| Vorhandene Regressionen | [tests/mrdb_tests.cpp](tests/mrdb_tests.cpp), 324–379 und 477–500 | Setup-Sichtbarkeit, Quellwerte, ungültiger MariaDB-Port bei mrdb, CSV/Test |
| KVS-Bibliothek, nur lesend untersucht | [../../tools/kvs.h](../../tools/kvs.h), 13–17, 23–33 und 42–123 | Speicherort, Lesen, Upsert, Präfixabfrage, keine Löschschnittstelle |
| Dateiformat, nur lesend untersucht | [../../tools/mrdb.h](../../tools/mrdb.h), 160–179, 248–272, 291–318 und 590–594 | Verzeichnisse und unveränderte Feldwerte, einzelne Dateischreibvorgänge |
| Connector, nur lesend untersucht | [../../tools/maria.h](../../tools/maria.h), 111–118, 168–215 und 721–740 | interne Konfiguration, Verbindung, Validierung und Fehler |
| UI-Bibliothek, nur lesend untersucht | [../../tools/win32form/win32form.h](../../tools/win32form/win32form.h), 478–479, 510–532 und 631 | Sichtbarkeit, Aktivierung, Textänderung, ComboBox und Meldungen |

Bestehende fachliche Vorgaben: [.agents/estw.md](.agents/estw.md), Abschnitt 2.8; [.agents/mainform.md](.agents/mainform.md), Zeilen 5–11; [.agents/lademechanismus_daten.md](.agents/lademechanismus_daten.md), Zeilen 3–5; [MRDB.md](MRDB.md), Zeilen 3–10 und 63–85. Die allgemeinen Projektregeln verlangen C++20, unveränderte öffentliche Schnittstellen ohne ausdrücklichen Folgeauftrag und keine Builds durch den Agenten. Gemeinsame Dateien unter `../../tools/` dürfen nur gelesen werden.

## 3. Bestehender Zustand

### 3.1 Datenmodell und aktive Verbindung

**Befund:** `ConnectionData` besitzt genau fünf `std::string`-Felder: `Host`, `Port`, `Database`, `User`, `Password`. Auch der Port ist ein String. Eine ID, ein Anzeigename, eine Liste und eine gesonderte aktive Auswahl fehlen. `ConfigData` enthält `DataSource`, genau ein `ConnectionData Connection` und `LocalCsvData LocalCsv` ([structs.h](structs.h), 129–156).

Die aktive Verbindung ist damit implizit immer `ConfigData.Connection`, sobald `DataSource == DataSource::MariaDB` ist. Auch bei CSV, Test oder mrdb bleiben diese fünf Werte im Konfigurationsobjekt vorhanden; sie werden lediglich nicht für diese Datenquelle verwendet.

Die gespeicherten Quellwerte sind `LocalCsv = 0`, `MariaDB = 1`, `Test = 2`, `Mrdb = 3`. Die ComboBox wird über `dataSourceOptions` gefüllt; mehrere MariaDB-Profile dürfen nicht als zusätzliche Werte dieser Quellenliste eingeführt werden ([structs.h](structs.h), 143–148; [datasource.h](datasource.h), 15–19).

### 3.2 Speicherort und Format

**Befund:** `main.cpp` verwendet folgende Schlüssel:

| KVS-Schlüssel | Inhalt |
| --- | --- |
| `estw3_dataSource` | numerischer Quellenwert als Text |
| `estw3_maria_host` | Host |
| `estw3_maria_port` | Port als Text |
| `estw3_maria_database` | Datenbankname |
| `estw3_maria_user` | Benutzer |
| `estw3_maria_password` | Passwort im Klartext |
| `estw3_localcsv_path` | CSV-Verzeichnis |

`saveConfig()` schreibt diese Werte nacheinander und unabhängig davon, welche Datenquelle gewählt wurde. `loadConfig()` liest sie ebenfalls unabhängig von der Quelle. Fehlende oder nicht lesbare Verbindungswerte ergeben über `getValue(key, "")` leere Strings. Ein ungültiger Quellenwert führt zur Voreinstellung CSV mit Konsolenhinweis ([main.cpp](main.cpp), 234–269).

Der KVS verwendet die mrdb-Tabelle `KVS` mit den Spalten `KEY` und `VALUE`. Die Basis ist in `../../tools/kvs.h:23` fest auf `d:/rentsch/database/` gesetzt. Auf Dateiebene entspricht ein Eintrag dem Schema `d:/rentsch/database/KVS/<interne RowID>/KEY` und `.../VALUE`; die RowID ist nicht der Schlüsseltext. Feldwerte werden als Text ohne anwendungsseitige Verschlüsselung geschrieben und wieder gelesen. Metadaten liegen in der mrdb-Tabellenstruktur, unter anderem `.META` ([../../tools/mrdb.h](../../tools/mrdb.h), 160–179, 248–272 und 291–318).

Die im Tools-Verzeichnis ebenfalls vorhandenen `kvsini.h/.cpp` sind nicht der von `main.cpp` eingebundene Speichermechanismus.

**Wichtige Grenzen:**

- `kvs::setValue()` ist ein Upsert pro Schlüssel. Mehrere passende Datensätze beim Schreiben führen zu einem Fehler.
- `getValue(key, default)` und `tryGetValue()` fangen sämtliche Lesefehler ab. Ein fehlender Eintrag lässt sich über diese Aufrufe nicht zuverlässig von einem beschädigten oder unzugänglichen KVS unterscheiden.
- Die öffentliche KVS-Schnittstelle hat keine Löschfunktion und keine Transaktion für mehrere Schlüssel. Eine Präfixabfrage ist vorhanden.
- Die unterliegende Datei wird direkt per `ofstream` geschrieben. Auch ein einzelner Aufruf ist keine belegte atomare oder absturzsichere Ersetzung.
- Die ESTW-Sperre und Rücksicherung in `mrdbstore.cpp` schützen die sechs `estw_*`-Tabellen. Sie umfassen die KVS-Konfiguration ausdrücklich nicht ([MRDB.md](MRDB.md), 74–85).

### 3.3 Programmstart, Übernahme und Fehler

**Befund:** Der Start lädt nur die Konfiguration und zeigt das Hauptfenster. Daten werden erst bei einer entsprechenden Hauptaktion durch `loadData()` geladen. Dabei wird ein kompletter `EstwData`-Bestand einschließlich `FullProjekte` aufgebaut. Das gilt nicht nur für das spätere Öffnen eines einzelnen Projekts, sondern bereits etwa für die Projekt- oder Elementliste ([main.cpp](main.cpp), 309–350 und 365–400).

Nach erfolgreichem `setup()` liest der Hauptablauf die Konfiguration neu, aktualisiert die Hauptfensteranzeige, setzt `isDataLoaded = false`, leert die Meldungen und ersetzt den gesamten Datenbestand durch `EstwData{}`. Dieser Reset erfolgt bereits heute bei jedem erfolgreichen Setup, auch wenn nur MariaDB-Zugangsdaten statt der Datenquellenart geändert wurden ([main.cpp](main.cpp), 402–410). Die bestehende Dokumentation verlangt verzögertes Laden und einen Reset bei Quellenänderung; der Code ist hier weiter gefasst.

Schlägt ein Ladevorgang mit einer Exception fehl, wird der Datenzugriff abgebrochen und `isDataLoaded` nicht auf wahr gesetzt. Ein späterer Versuch ist möglich. Eine automatische Auswahl einer anderen Verbindung oder ein automatisches Öffnen des Setup existiert nicht. Ein Kommentar in `aktionen.cpp:375–376` beschreibt einen gewünschten Korrekturablauf, implementiert ihn aber nicht.

Davon zu unterscheiden sind fachliche Fehler oder Warnungen im Meldungssammler: Kehrt `generate_estw_data()` regulär zurück, setzt `loadData()` den Bestand bereits auf geladen und zeigt gegebenenfalls eine Meldungsliste. Solche Meldungen allein bewirken keinen Verbindungswechsel und keinen erneuten Ladeversuch.

Beim MariaDB-Lesen wird eine Exception aus `Connect()` zunächst nur mit `Error connecting to MariaDB` protokolliert; danach werden trotzdem die Tabellenleser aufgerufen. Diese können wegen fehlender Verbindung mit `No active database connection` scheitern. Dadurch kann die ursprüngliche Verbindungsursache verdeckt werden. Beim manuellen Test wird die ursprüngliche Standardexception dagegen angezeigt ([aktionen.cpp](aktionen.cpp), 720–742; [../../tools/maria.h](../../tools/maria.h), 737–740; [configform.cpp](configform.cpp), 227–245).

### 3.4 Setup und bestehender Speichervorgang

**Befund:** Das Fenster ist nicht vergrößerbar und hat eine Clientgröße von 360 × 270 Pixeln. Die fünf MariaDB-Felder liegen unterhalb der Quellenwahl; unten befinden sich Test und Speichern. Verbindungsauswahl und weitere Buttons benötigen daher eine neue Anordnung beziehungsweise eine größere feste Fensterfläche ([configform.cpp](configform.cpp), 114–207 und 248–250).

`ShowModal()` kopiert die übergebene Konfiguration nach `m_initialData` und `m_resultData`, füllt alle Felder und zeigt nur den zur Quelle passenden Bereich. Bei MariaDB sind Test und die fünf Felder sichtbar; bei mrdb/Test sind diese und der CSV-Bereich ausgeblendet ([configform.cpp](configform.cpp), 292–375).

Speichern prüft nur für die aktuell gewählte Quelle MariaDB den Port. Er muss vollständig als Zahl lesbar sein und zwischen **1024 und 49151** liegen. Host, Database und User werden hier nicht auf Vollständigkeit geprüft. Test verwendet dieselbe Portprüfung. Der Connector akzeptiert Ports zwischen **1 und 65535** und verlangt Host und User; er fordert weder ein nichtleeres Passwort noch ausdrücklich einen nichtleeren Datenbanknamen. Die unterschiedlichen Portbereiche sind Bestand, kein durch mehrere Profile erforderlicher Änderungsauftrag.

Beim Speichern werden auch ausgeblendete CSV-/MariaDB-Felder übernommen. Anschließend setzt der Dialog `m_accepted`, schließt und gibt das Ergebnis zurück. Erst `setup()` schreibt in den KVS. Bei einem Speicherfehler ist das Fenster bereits geschlossen; der Hauptablauf behält seine bisherige Konfiguration und seinen Cache. Weil mehrere KVS-Schreibvorgänge vorausgegangen sein können, kann der persistente Stand trotzdem teilweise geändert sein ([configform.cpp](configform.cpp), 251–271; [main.cpp](main.cpp), 281–297 und 402–410).

Die Schließabfrage vergleicht Quelle, CSV-Pfad und sämtliche Verbindungsfelder mit dem Öffnungsstand. Änderungen werden nicht laufend für Buttons ausgewertet. Ein Bearbeiten und Zurücksetzen auf exakt dieselben Werte gilt nach dieser aktuellen Vergleichslogik wieder als unverändert ([configform.cpp](configform.cpp), 274–288 und 326–349).

## 4. Passwörter: belegter Umgang und Konsequenzen

**Befunde:**

| Stelle | Aktuelles Verhalten |
| --- | --- |
| Eingabe | Normale TextBox ohne `ES_PASSWORD` beziehungsweise gesetztes Passwortzeichen: `configform.cpp:201–203`, `win32form.cpp:1095–1097` |
| Speicherung | Unveränderte Übergabe an `kvs::setValue(nameKeyMariaPassword, ...)`: `main.cpp:268` |
| Speicherformat | Unveränderter Text in der KVS-Felddatei; keine Verschlüsselung im untersuchten Pfad |
| Arbeitsspeicher | Kopien in `ConfigData`, `m_initialData`, `m_resultData`, Testdaten und Connector-Konfiguration |
| Verbindungsaufbau | Übergabe als Connector-Property `password`: `../../tools/maria.h:192–195` |
| Ausgabe | `showConfig()` gibt das Passwort aus; Aufruf nach erfolgreichem Setup in `main.cpp:296`. Der entsprechende Aufruf beim Start ist auskommentiert. |

Es wurden keine tatsächlichen Passwörter gelesen. Dateirechte, Sicherungen und die tatsächlich ausgehandelte Transportverschlüsselung wurden nicht untersucht. Aus dem fehlenden zusätzlichen TLS-Feld im Projekt darf kein gesicherter Schluss auf den Transportzustand gezogen werden.

**Vorschlag:** Die Passwortausgabe bei einer späteren Implementierung entfernen und das Eingabefeld maskieren. Beides verbessert den bestehenden Umgang unabhängig von der Wahl der dauerhaften Geheimnisspeicherung. Maskierung verschlüsselt weder KVS noch Arbeitsspeicher. Passwortinhalte dürfen nicht in ComboBox-Beschriftungen, Löschabfragen oder neue Diagnosen gelangen.

**Entscheidung erforderlich:** Klartextspeicherung beibehalten oder einen gesonderten Windows-Geheimnisspeicher beziehungsweise benutzergebundenen Schutz vorsehen. Ein solcher Schutz benötigt zusätzlich Regeln für Migration, Rechner-/Benutzerwechsel, Sicherung und nicht entschlüsselbare Altwerte. Er ist nicht bereits durch den Auftrag beschlossen. Das Beibehalten alter KVS-Schlüssel würde auch deren bisherige Klartextpasswörter erhalten; eine spätere Bereinigung muss ausdrücklich zum Migrationskonzept gehören.

## 5. Mögliches Zielmodell und Persistenz

### 5.1 Eine Liste, stabile Identität und genau eine aktive Auswahl

**Vorschlag:** `ConnectionData` als Werttyp der fünf Felder erhalten und um ein Profilmodell ergänzen:

| Modell | Vorgeschlagener Inhalt | Zweck |
| --- | --- | --- |
| `MariaDbProfile` | stabile `id`, optionaler fachlich zu entscheidender `name`, `ConnectionData connection` | Identität unabhängig von Hoständerung, Anzeigename und Listenposition |
| `MariaDbSettings` | `std::vector<MariaDbProfile> profiles`, `std::optional<std::string> activeId` | gespeicherte Verbindungen und ausdrücklich darstellbare Nichtauswahl |
| `ConfigData` | bestehende Quellenwahl und CSV-Daten, zusätzlich MariaDB-Einstellungen | gemeinsame Laufzeitkonfiguration |
| zentrale Auflösung | Profil zu `activeId` suchen, Daten als `ConnectionData` liefern oder verständlichen Fehler erzeugen | dieselbe Verbindung für Anzeige, Lesen und Schreiben |

`ConnectionData` selbst muss keine Profilliste enthalten. `EstwData`, `FullProjekt` und die MariaDB-Fachtabellen benötigen für das Grundziel keine neuen Profilfelder. Die stabile Profil-ID ist keine Projekt-ID und kein ComboBox-Index.

Der aktive Verweis muss entweder leer sein oder genau ein vorhandenes Profil treffen. Ungültige oder doppelte Profil-IDs werden beim Laden erkannt. Nach Löschen darf ein alter Vektorindex nicht unbeabsichtigt das nächste Profil aktivieren. Auch eine Umbenennung oder Bearbeitung von Host/User darf die Profilidentität nicht verändern.

Eine dauerhaft parallel gepflegte Kopie `ConfigData.Connection` neben Liste und `activeId` wäre fehleranfällig. Wenn für einen begrenzten Migrationsschritt eine solche Projektion erhalten bleibt, muss sie ausschließlich zentral aus dem gewählten Profil abgeleitet werden. Kein Leser oder Schreiber darf einen unabhängig veralteten Verbindungssatz nutzen. Langfristig ist die direkte zentrale Auflösung vorzuziehen.

### 5.2 Auswahl eines Speicherformats

**Vorschläge im Vergleich:**

| Variante | Vorteil | Aufwand/Grenzen |
| --- | --- | --- |
| Versionierter Gesamtsatz als JSON in einem neuen KVS-Wert | Liste, IDs und aktive Auswahl zusammen; logisches Löschen durch erneutes Schreiben; kein KVS-Lösch-API erforderlich | Serializer und strenge Prüfung nötig; ein KVS-Aufruf ist weiterhin nicht absturzsicher; Quelle/CSV bleiben bei getrennten Schlüsseln außerhalb dieses Gesamtsatzes |
| Einzelne KVS-Schlüssel pro Profil und ein Index/Manifest | Bestehende Stringzugriffe gut wiederverwendbar; Präfixabfrage vorhanden | Viele Teilzustände; fehlendes Lösch-API; Index und Werte können auseinanderlaufen; verwaiste Passwörter bleiben zurück |
| Neue anwendungseigene Konfigurationsdatei mit kontrollierter Ersetzung | Speichervorgang einschließlich Sicherung gezielt gestaltbar | Neuer Speicherort, Migration und Fehlerpfad; weitergehender Eingriff als bloße KVS-Erweiterung |

**Begründeter Vorschlag für den ersten Entwurf:** Ein versionierter MariaDB-Gesamtsatz unter einem neuen Schlüssel, beispielsweise `estw3_maria_profiles_v1`. Sein Schema enthält `schemaVersion`, `profiles` und `activeId`; IDs und aktive Auswahl werden nicht in getrennten KVS-Aufrufen geschrieben. Ein JSON-Header ist bereits unter `../../tools/json.hpp` vorhanden, wird bisher aber nicht von diesem Konfigurationspfad verwendet. Seine Nutzung kann innerhalb des Projekts erfolgen, ohne die Tools-Datei zu ändern.

Der Gesamtsatz muss auch die absichtlich leere Liste und `activeId = null` speichern können. Ein gültiger leerer Gesamtsatz ist ein neuer gültiger Zustand, kein Anlass zur erneuten Altimportierung. Passwort und andere Strings müssen korrekt serialisiert werden; eine eigene Trennung an Komma, Semikolon oder Doppelpunkt wäre dafür ungeeignet.

Dieser Vorschlag löst die **logische Zusammengehörigkeit**, nicht automatisch die vorhandene KVS-Schreibsicherheit. Vor Umsetzung ist festzulegen, ob ein anwendungslokaler Sicherungs-/Wiederherstellungsmechanismus, serialisierte Zugriffe und das Zurücklesen des gespeicherten Gesamtsatzes zum Umfang gehören. Auch der gemeinsame Abschluss mit Quellenwahl/CSV muss definiert werden. Keinesfalls darf nach einem fehlgeschlagenen Schreibvorgang pauschal „gespeichert“ angezeigt oder der Vergleichsstand zurückgesetzt werden.

### 5.3 Abwärtskompatibilität und Migration

**Vorschlag für eine wiederholbare, verlustvermeidende Migration:**

1. Neuen Konfigurationseintrag zuverlässig lesen. **Nicht vorhanden**, **vorhanden und gültig**, **beschädigt/nicht lesbar** und **unbekannte Version** unterscheiden. Die derzeitigen KVS-Aufrufe mit Defaultwert reichen dafür allein nicht aus; ein projektnaher Leseadapter muss Fehler erhalten. Die Präfixabfrage kann als Baustein dienen, muss aber auf doppelte Schlüssel geprüft werden, da ihre Map gleiche Schlüssel zusammenfasst.
2. Existiert ein gültiger neuer Satz, ist er allein maßgeblich, auch wenn Liste oder aktive Auswahl leer sind.
3. Nur bei nachweislich fehlendem neuen Satz die fünf alten Schlüssel importieren. Das alte Modell enthielt keine Pflicht zum erfolgreichen Verbindungstest; unvollständige Altwerte deshalb erhalten und als korrekturbedürftig kennzeichnen, statt zu verwerfen.
4. Für einen importierbaren Altbestand genau ein Profil mit stabiler ID und einer nach der Namensentscheidung erzeugten Beschriftung anlegen. Für die Kompatibilität die frühere implizite Verbindung als aktive Auswahl übernehmen. Bei vollständig leeren Altwerten ist gesondert zu entscheiden, ob überhaupt ein Profil angelegt wird. Die aktuelle Quelle und der CSV-Pfad bleiben erhalten.
5. Migration zunächst im Speicher vorbereiten. Vorschlag: dauerhaft erst bei erfolgreichem Speichern des Setup übernehmen; ein bloßes Öffnen/Abbrechen schreibt dann nichts. Ein anderer Migrationszeitpunkt muss ausdrücklich festgelegt werden.
6. Erst nach bestätigtem Speichern den neuen Satz als maßgeblich behandeln. Bei Fehler bleiben Altwerte erhalten und der Fehler sichtbar. Beim nächsten Versuch wird kein zweites Importprofil erzeugt.
7. Vorhandenen, aber beschädigten neuen Satz nicht still durch Altwerte oder eine leere Liste ersetzen. Fehler melden und eine gezielte Wiederherstellung ermöglichen, ohne vorhandene Daten automatisch zu überschreiben.

Die Enumwerte bleiben unverändert. **Abwärtskompatibilität zu gespeicherten Altbeständen** bedeutet nicht automatisch, dass eine alte Programmversion neue Profillisten lesen kann. Ein Rückweg zu alten Binärversionen, das zusätzliche Spiegeln eines aktiven Profils in die alten Schlüssel und die anschließende Bereinigung alter Geheimnisse sind eigene Entscheidungen. Dauerhaftes Spiegeln erzeugt erneut mehrere Schreibpunkte und kann gelöschte Zugangsdaten zurücklassen.

## 6. UI-Verhalten und Änderungszustände

### 6.1 Bereits vorgegebenes Verhalten

**Anforderungen:** Nur bei gewählter Quelle MariaDB erscheint zusätzlich eine ComboBox mit der Beschriftung **Verbindung**. Sie listet alle gespeicherten Profile auf. Die Auswahl lädt Host, Port, Database, User und Passwort in die vorhandenen Felder.

Neu setzt die Auswahl auf „keine“ und leert alle fünf Felder. Damit wird ein neuer Entwurf begonnen. Die Profilmenge wird dadurch nicht gelöscht. „ComboBox leeren“ ist hier als **Auswahl leeren** zu verstehen, weil andernfalls die verlangte Auflistung aller gespeicherten Verbindungen verloren ginge.

Löschen erfordert eine Sicherheitsabfrage und entfernt bei Bestätigung genau das gewählte Profil. Anschließend sind Auswahl und Felder leer. Andere Profile bleiben in der Liste; es wird **keines automatisch ausgewählt**. Verneinen erhält Profil, Auswahl und Felder.

Sobald eines der fünf Felder geändert ist, sind Neu und Löschen deaktiviert. Nach Laden, erfolgreichem Speichern oder eindeutig definiertem Zurücksetzen werden beide neu bewertet.

### 6.2 Zwei getrennte Änderungsvergleiche

**Technischer Vorschlag:** Drei Zustandsbereiche auseinanderhalten:

- **Feldentwurf:** aktuell ausgewählte Profil-ID oder kein Profil, fünf sichtbare Werte und ihr letzter Lade-/Rücksetzstand.
- **Setup-Arbeitsstand:** vollständige Profilliste, vorgesehene aktive ID, Quelle und CSV-Pfad.
- **Dauerhaft bestätigter Stand:** letzte nachweislich erfolgreiche Speicherung der gesamten betroffenen Konfiguration.

`fieldsDirty` vergleicht die fünf Entwurfswerte mit ihrem Feldvergleichsstand. `setupDirty` berücksichtigt darüber hinaus neue/gelöschte/geänderte Profile, aktive Auswahl, Quelle und CSV-Pfad gegenüber dem dauerhaften Stand. Ein sauber geladenes Profil kann also `fieldsDirty = false` haben, während wegen eines vorher gelöschten Profils weiterhin `setupDirty = true` ist. Diese Unterscheidung verhindert, dass ein Auswahlwechsel eine noch nicht gespeicherte Löschung aus der Schließabfrage verschwinden lässt.

`ConfigForm::hasUnsavedChanges()` lässt sich für die globale Prüfung ausbauen; als alleinige Neu-/Löschen-Bedingung ist es ungeeignet. Eine Änderung der Datenquelle oder des CSV-Pfads soll nicht automatisch als Änderung eines der fünf MariaDB-Felder behandelt werden.

**Vorgeschlagene Zustandsmatrix, noch zu bestätigen:**

| Zustand bei Quelle MariaDB | Neu | Löschen | Weiteres Verhalten |
| --- | --- | --- | --- |
| Vorhandenes Profil frisch geladen | aktiv | aktiv | Felder entsprechen Ladezustand |
| Mindestens ein Feld abweichend | deaktiviert | deaktiviert | Wechsel/Abbruch nach beschlossener Entwurfsregel |
| Nach Neu oder bestätigtem Löschen: keine Auswahl, Felder leer | aktiv | deaktiviert | leerer Rücksetzstand; andere Profile bleiben auswählbar |
| Profil erfolgreich gespeichert | aktiv | aktiv, falls anschließend ausgewählt | neuer Vergleichsstand erst nach Persistenzerfolg |
| Speichern fehlgeschlagen | anhand des unveränderten Entwurfs | anhand des unveränderten Entwurfs | keine Erfolgsvortäuschung, Eingaben erhalten |
| Andere Datenquelle | ausgeblendet | ausgeblendet | Profilbestand erhalten; keine MariaDB-Pflichtprüfung |

Die zusätzliche Bedingung „Löschen nur mit gültiger Auswahl“ ist ein technischer Vorschlag; ein Klick ohne Profil darf jedenfalls keinen beliebigen Datensatz löschen. Ob ein Rückgängigmachen aller Feldänderungen die Buttons sofort wieder aktiviert, ist noch zu entscheiden. Ein Wertvergleich würde dies wie im bestehenden Projektfenster leisten, ein bis zum Speichern gesetztes Änderungsflag dagegen nicht.

### 6.3 Wiederverwendbare UI-Mechanismen

**Befunde und daraus abgeleitete Vorschläge:**

| Aufgabe | Wiederverwendbarer Mechanismus |
| --- | --- |
| Profile auflisten | `createComboBox(..., DropDownList)`, `addComboBoxItem()`, Auswahlereignis und Indexabfrage |
| Zuordnung | Projektseitige Liste der Profil-IDs in derselben Reihenfolge; Anzeige niemals als Identität benutzen |
| Keine Auswahl | `setComboBoxSelectedIndex(name, -1)` nutzt vorhandenen `CB_SETCURSEL`-Wrapper |
| Liste neu füllen | Öffentliches Leeren-/Entfernen-API fehlt; projektnah über Controlhandle und `CB_RESETCONTENT`, anschließend vorhandenes Hinzufügen. `setComboBoxText("")` ist bei DropDownList ungeeignet und kann eine Exception werfen. |
| Buttons | `createButton()`, `setButtonClicked()`, `setControlEnabled()` |
| Änderungen | `setTextBoxTextChanged()` für alle fünf Felder; `updateModeButtons()`/`hasProjectChanges()` in `ProjektForm` als Muster |
| Programmgesteuertes Laden | Felder und Vergleichsstand kontrolliert setzen, Zwischenereignisse über Ladeflag abfangen und am Ende einmal vollständig neu bewerten |
| Sichtbarkeit | `setPanel2Visibility()` auf die neuen MariaDB-Controls erweitern |
| Meldungen | Für Information/Fehler `Win32Form::showMessage()` entsprechend Projektregel |
| Bestätigung | `showMessage()` liefert `void` und unterstützt nur OK. Für eine Ja/Nein-Entscheidung fehlt eine passende Funktion; daher projektlokal `MessageBoxW` mit Besitzer, `MB_YESNO`, Nein als Standard und Prüfung auf `IDYES`, wie bei der vorhandenen Verwerfungsabfrage. Keine Änderung der gemeinsamen Bibliothek nötig. |

Fundstellen: `win32form.h:478–479, 510–532, 631`; `win32form.cpp:1559–1579, 1609–1681, 3345–3349, 4687–4706`; `projektform.cpp:155–196`; `configform.cpp:274–288`.

Der zentrale Ereignisverteiler der UI-Bibliothek fängt Exceptions in Control-Callbacks teilweise kommentarlos ab (`win32form.cpp:3335–3400`). Erwartbare Speicher-/Auswahlfehler sollten deshalb im projektnahen Callback selbst behandelt und angezeigt werden; bloßes Werfen gewährleistet keine Fehlermeldung.

### 6.4 Offener Zeitpunkt des Speicherns und Löschens

Der bisherige Button Speichern ist ein **Setup speichern und schließen**. Folgende Erweiterungen sind möglich, aber noch nicht vorgegeben:

- Bestehenden Button beibehalten: Neu/Bearbeiten/Löschen ändern zunächst einen Setup-Arbeitsbestand. Ein abschließendes Speichern übernimmt alles und schließt. Abbrechen kann alle noch nicht gespeicherten Verwaltungsänderungen verwerfen.
- Verbindungen im geöffneten Fenster dauerhaft speichern/löschen; zusätzlich einen klaren Abschluss für Quellenwahl und aktive Auswahl definieren. Nach einem solchen Commit darf Schließen nicht behaupten, bereits gespeicherte Änderungen zu verwerfen.

**Vorschlag:** Für den kleineren ersten Schritt den vollständigen Setup-Arbeitsbestand gemeinsam übernehmen und den bestehenden Abschluss grundsätzlich erhalten. Den dauerhaften Schreibversuch aber über einen projektnahen Callback vor das erfolgreiche Schließen verlegen, damit Speicherfehler Eingaben und Dialog erhalten. Dies würde `ConfigForm`/`setup()` und gegebenenfalls deren öffentliche Schnittstelle betreffen und muss zum Umfang des Folgeauftrags gehören.

Ein eigener „Verbindung speichern“-Button oder „Übernehmen ohne Schließen“ ist nur nötig, wenn mehrere neue/bearbeitete Verbindungen innerhalb derselben geöffneten Sitzung gesichert werden sollen. Der vorhandene Button fehlt also nicht; seine gewünschte künftige Bedeutung ist offen.

## 7. Auswirkungen auf die vorhandenen Funktionen

| Bereich | Bestehender Zusammenhang | Erforderliche Behandlung/Vorschlag |
| --- | --- | --- |
| Datenquellenwahl | eine der vier Quellen; Profilwerte bleiben auch bei anderen Quellen gespeichert | Quellenart unverändert lassen, Auswahl innerhalb MariaDB ergänzen |
| Start | KVS lesen, danach Hauptfenster ohne Fachdatensatz laden | Profile und aktive ID laden/prüfen; weiterhin keine obligatorische Netzwerkverbindung beim Start |
| Hauptfenster | zeigt bei MariaDB nur Host und Port | Beide Werte aus dem aktiv aufgelösten Profil; Nichtauswahl ausdrücklich anzeigen. Name/Database zusätzlich wären eine optionale Erweiterung der bisherigen Anzeige. |
| Datenladen | eine Verbindung liest alle sechs Tabellen | genau einmal gewähltes Profil für den gesamten Ladevorgang verwenden; fehlende aktive Auswahl vor `Connect()` melden |
| Verbindungstest | arbeitet mit aktuellen Eingabefeldern und eigenem Connector | auch ungespeicherten Entwurf testbar halten; Test wählt/speichert kein Profil und ersetzt keine Schema-/Schreibprüfung |
| Elemente | `persistElementChanges()` mit neuer Verbindung, Transaktion und Originalvergleich | Verbindung aus demselben Profilkontext wie der geladene Bestand; bestehende Ergebniszustände erhalten |
| Fahrstraßen/Zuordnungen | `persistFahrstrassenChanges()` mit Projekt-/Element-/Fahrstraßenvergleich | gleicher Profilkontext; keine Änderung der fachlichen Prüfungen oder SQL-Tabellen erforderlich |
| Projekttexte | `persistProjectDetails()` verwendet dieselbe Konfiguration und eine Transaktion | gleiches Ziel wie Projektcache; bestehende Validierung und Konfliktprüfung erhalten |
| Detailfenster | leitet Sofortspeicherung an Element-/Fahrstraßenpersistenz weiter | keine zweite Profilauswahl einführen; Fehler erhalten wie bisher Eingaben |
| Projektneuanlage | im Hauptablauf ausschließlich für mrdb freigegeben | mehrere MariaDB-Profile begründen keine Erweiterung auf MariaDB-Neuanlage |
| Simulation | verwendet das aus einer Quelle geladene `FullProjekt` | keine Änderung der Stellwerksmechanik; keinen gemischten Datenbestand erzeugen |
| Cache | ein globaler Bestand, Reset nach erfolgreichem Setup | beim Profilwechsel auch ohne Wechsel der Quellenart zurücksetzen; niemals nach fehlgeschlagenem Commit als erfolgreich wechseln |
| mrdb | gleicher Basispfad wie KVS, eigene `estw_*`-Tabellen und eigener Sicherungsumfang | keine Profilmigration in diese Tabellen, keine Änderung ihres Pfads oder Journals |
| CSV/Test | keine Verwendung der MariaDB-Felder | auch ohne Profil/mit leerem oder ungültigem Entwurf weiter nutzbar; verdeckte Feldwerte nicht pauschal löschen |
| Ansichtseinstellungen | `estw3_lupe_project_<ID>_<Option>` ohne Quelle/Profil | gleiche Projekt-ID in zwei Datenbanken teilt aktuell die Ansicht. Fachlich entscheiden, ob das bleiben darf; getrennte Schlüssel wären eine zusätzliche Erweiterung. |

Es existieren fünf zentrale Connector-Erzeugungspfade im Projekt: manueller Test, Gesamtdatenladen, Elementspeicherung, Fahrstraßenspeicherung und Projekttextspeicherung. Die letzten drei nutzen bereits `makeConnectionConfig()`, die ersten zwei duplizieren die Feldzuordnung. Ein gemeinsamer, rein projektinterner Resolver plus Konverter verhindert unterschiedliche Profilwahl und Portbehandlung. Der gemeinsame Connector unter `../../tools/maria.h` muss dafür nicht erweitert werden.

**Besonders relevantes Risiko:** Projekt-, Element- und Fahrstraßen-IDs sind nur innerhalb ihres Datenbestands aussagekräftig. Originalstandsprüfungen gegen eine versehentlich andere Datenbank verhindern nicht jeden falschen Schreibzugriff: Zwei Datenbanken könnten dieselben IDs und denselben Originalinhalt besitzen. Der vorhandene vollständige Cache-Reset nach Setup und ein konsistenter Profilkontext sind deshalb wesentlich. Die aktuelle modale Dialogführung begrenzt Wechsel während der Projektbearbeitung; diese Einschränkung sollte die erste Ausbaustufe erhalten, statt einen Wechsel in laufenden Editoren einzuführen.

## 8. Sortierte Anforderungen

### 8.1 Bereits klar festgelegte Anforderungen

| ID | Anforderung aus dem Auftrag |
| --- | --- |
| A1 | Mehrere gespeicherte MariaDB-Verbindungen verwalten und genau die gültige Auswahl anzeigen. |
| A2 | Bei Quelle MariaDB zusätzliche ComboBox mit Beschriftung „Verbindung“ anzeigen; alle gespeicherten Verbindungen auflisten. |
| A3 | Jedes Profil enthält mindestens Host, Port, Database, User und Passwort. |
| A4 | Profilwahl lädt alle fünf Werte in die vorhandenen Felder. |
| A5 | Neu hebt die Auswahl auf und leert alle fünf Felder; vorhandene Profile bleiben erhalten. |
| A6 | Löschen erfordert Bestätigung und löscht nur das gewählte Profil. |
| A7 | Nach bestätigtem Löschen bleiben Auswahl und Felder leer, auch bei verbleibenden Profilen. |
| A8 | Jede Änderung eines der fünf Felder deaktiviert Neu und Löschen. |
| A9 | Laden, erfolgreiches Speichern und definiertes Zurücksetzen bewerten die Buttonzustände neu. |
| A10 | Dieser Auftrag bleibt Analyse; Code und bestehende Dokumentation werden nicht geändert. |

### 8.2 Technische Voraussetzungen und Abhängigkeiten

- Liste und stabile Profilidentität sowie ein ausdrücklich darstellbarer Zustand ohne aktive Verbindung.
- Verlässliche Auflösung derselben aktiven Verbindung in Anzeige, Lesen und sämtlichen Schreibpfaden.
- Getrennte Feld- und Setup-Änderungsverfolgung einschließlich neuer/gelöschter Profile und aktiver Auswahl.
- Festgelegte Persistenzgrenze, Fehlerbehandlung und Migration; keine Gleichsetzung von „nicht vorhanden“ und „nicht lesbar“.
- Erhaltung der Enumwerte, der anderen Datenquellen, des verzögerten Ladens und des vollständigen Cache-Resets.
- Neue Dialoganordnung und Controls unter Wiederverwendung der vorhandenen Win32Form-Funktionen.
- Einbeziehung der bestehenden MRDB-/Setup-Regressionen, deren bisherige Annahme einer einzelnen ComboBox bei Erweiterung überprüft werden muss.
- Umsetzung allein im Projekt; `../../tools/` und bestehende öffentliche Schnittstellen nicht ohne entsprechenden Folgeauftrag ändern.

### 8.3 Entscheidungen, die vor der Implementierung getroffen werden müssen

Anzeigename und Eindeutigkeit; Behandlung gleicher Endpunkte; dauerhafte aktive Auswahl; Neuanlage versus Aktualisierung; Commit-/Abbruchverhalten; Entwurfswechsel; MariaDB ohne aktive Auswahl; Löschzeitpunkt und Persistenzfehler; Passwortschutz und Altwertbereinigung; Migrations- und Speicherformat; Bedeutung „geändert“ beim Wiederherstellen der Originalwerte. Die konkreten Rückfragen stehen am Ende.

Die unmittelbare Nichtauswahl nach Löschen ist hier **keine** offene Entscheidung. Eine automatische Ersatzwahl würde die bereits festgelegte Anforderung ändern.

### 8.4 Optionale Verbesserungen außerhalb des zwingenden ersten Schritts

- Zusätzliche Hauptfensteranzeige von Profilname und Datenbank, damit zwei Profile am selben Host/Port unterscheidbar sind.
- Nach Erreichbarkeitstest gesonderte Prüfung des erwarteten ESTW-Schemas und der benötigten Rechte; ein erfolgreicher `Connect()` genügt dafür nicht.
- Komfortfunktionen wie Kopieren eines Profils, Sortierung, Import/Export und „Übernehmen ohne Schließen“.
- Asynchroner Verbindungstest mit Fortschritt/Abbruch und einem definierten Zeitlimit. Der aktuelle Test läuft synchron im UI-Callback; konkrete Treiberzeitlimits sind im Projekt nicht konfiguriert.
- Getrennte Ansichtseinstellungen nach Quelle/Profil und Projekt-ID einschließlich Übernahme alter Ansichtsschlüssel.
- Weitergehende TLS-/Zertifikatsoptionen und ein umfassendes Geheimnisspeicherkonzept, sofern nicht schon durch die Passwortentscheidung verpflichtend gemacht.
- Änderung des bisherigen Portbereichs nur mit eigenem fachlichem Beschluss; mehrere Profile benötigen diese Änderung nicht.

Das Vermeiden neuer Passwortausgaben gehört unabhängig davon zu einer sachgerechten Umsetzung; bestehende Geheimnisse müssen nicht zur Fehlersuche protokolliert werden.

### 8.5 Risiken und Sonderfälle

| Fall | Risiko und notwendige Behandlung |
| --- | --- |
| Profilmenge leer oder aktive ID fehlt | Kein Zugriff mit implizit leeren Zugangsdaten und keine stille Auswahl irgendeines Profils. Setup bleibt erreichbar. |
| Aktives Profil gelöscht | In der UI sofort Nichtauswahl; Zeitpunkt der dauerhaften Deaktivierung festlegen. Andere Profile bleiben erhalten. |
| MariaDB gewählt, nach Löschen Felder leer | Bisherige Portprüfung verhindert das abschließende Speichern. Entscheiden, ob MariaDB ohne aktive Verbindung speicherbar ist oder ein anderer expliziter Abschluss nötig ist. |
| Nicht aktives Profil gewählt und gelöscht | Dialogauswahl und bisher dauerhaft aktives Profil auseinanderhalten; Löschziel eindeutig bestimmen. |
| Neu mit leerem Port | Leerer Rücksetzstand ist für Neu zulässig, aber noch kein gültiger zu speichernder Verbindungssatz. |
| Name oder Endpunkt doppelt | Ohne stabile ID und eindeutige Anzeige drohen falsche Aktualisierung oder Löschung. |
| Auswahlwechsel/Quellenwechsel mit Entwurf | Ungespeicherte Werte dürfen nicht ohne beschlossene Regel verschwinden oder als Änderung eines anderen Profils gespeichert werden. |
| Verdeckte MariaDB-Felder nach Quellenwechsel | Globaler Abschluss darf weder ein halbes neues Profil erzwingen noch die gespeicherte Liste mit leeren Feldern überschreiben. |
| Server nicht erreichbar/Anmeldung abgelehnt | Profil und Eingaben erhalten; klare Diagnose mit nicht geheimen Identifikationsdaten; kein automatischer Zugriff auf Ersatzdatenbank. |
| Connect erfolgreich, Schema/Rechte falsch | Test ist keine Zusicherung, dass alle sechs Tabellen oder Schreibrechte verfügbar sind. |
| Speicherfehler/Programmabbruch beim Schreiben | Alte und neue Werte könnten auseinanderlaufen; keine Erfolgsmeldung, keine vorzeitige Cache-Umschaltung, definierte Wiederherstellung. |
| Beschädigte neue Konfiguration | Kein stilles Altimportieren oder Überschreiben; auch unbekannte Schemaversion sichtbar behandeln. |
| Leere neue Liste plus alte Schlüssel | Ohne Erkennung des gültigen neuen Leerzustands würde das gelöschte Altprofil wieder erscheinen. |
| Zwei Anwendungsinstanzen | KVS hat in diesem Pfad keinen gemeinsamen Commit-/Versionsschutz; verlorene Profiländerungen sind möglich. Schutzumfang festlegen. |
| Passwort mit Sonderzeichen/Leerzeichen | Keine pauschale Normalisierung von Passwortstrings; der bisherige mrdb-Leser macht ausschließlich aus Leerraum bestehende Feldinhalte zu leerem Text. Eine neue Serialisierung muss die gewünschte Werterhaltung festlegen. |
| Gleiche Projekt-ID in zwei Datenbanken | Ansichtsschlüssel kollidieren; falsche Verbindung beim Schreiben ist durch Inhaltsvergleich allein nicht sicher erkennbar. |
| Speicherung der Quellenwahl vor Profilmenge | Teilfehler kann Quelle MariaDB mit unpassender/fehlender aktiver Auswahl hinterlassen; gemeinsam definierter Abschluss nötig. |

## 9. Begründete Umsetzung in Teilschritten

Die Schritte sind ein Vorschlag für einen späteren Auftrag. Sie sind in diesem Analyseauftrag nicht ausgeführt.

| Schritt | Inhalt und betroffene Stellen | Prüfkriterium/Abhängigkeit |
| --- | --- | --- |
| 1. Fachvertrag festlegen | Rückfragen entscheiden; Zustände für Auswahl, Entwurf, Speichern, Löschen und Abbruch verbindlich beschreiben | Insbesondere Nichtauswahl, Persistenzzeitpunkt, Profilname und Passwortschutz geklärt |
| 2. Modell und Auflösung | `structs.h`, projektnaher Konfigurationsbaustein; `ConnectionData` weiter als fünfteiligen Wert verwenden | IDs stabil/eindeutig; aktive ID leer oder auflösbar; kein stiller Fallback |
| 3. Persistenz und Migration | `loadConfig()`/`saveConfig()` aus `main.cpp` sinnvoll kapseln; neuen Gesamtsatz lesen, prüfen und schreiben | Altbestand verlustfrei übernommen; Leerzustand dauerhaft; Fehler bleiben von „fehlend“ unterscheidbar; vereinbarte Wiederherstellung |
| 4. Verbraucher vereinheitlichen | `aktionen.cpp` für Test/Lesen/alle drei Schreibpfade; `mainform.cpp` für Anzeige | Alle Pfade nutzen denselben aktiven Satz; kein veraltetes `Connection` neben Profilen |
| 5. Setup erweitern | `configform.h/.cpp`: Layout, Verbindungsliste, Neu/Löschen, zwei Änderungsstände, Bestätigungsdialog | Alle A1–A9 und beschlossenen Entwurfsregeln erfüllt; Fehler erhalten Eingaben |
| 6. Laufzeitübernahme absichern | `setup()`/`OnSetup` und modale Aufrufer in `main.cpp` | Erst nach erfolgreicher Persistenz umschalten; Cache/Fehlerliste zurücksetzen; Nichtauswahl blockiert MariaDB-Datenzugriff verständlich |
| 7. Regression und gezielte Integration | Bestehende Testtargets ergänzen, besonders Setup/MRDB, plus Konfigurations-/Migrationsfälle | Andere Quellen unverändert; Testdatenbank A/B werden nie vermischt; keine Produktiv-KVS-Schreibtests |

Modell und Migration vor den UI-Ereignissen zu bearbeiten, macht Identität und Speicherzustand unabhängig vom ComboBox-Index überprüfbar. Die anschließende zentrale Auflösung reduziert das Risiko, dass Lesen und ein weniger sichtbarer Schreibpfad verschiedene Verbindungen verwenden. Der abschließende Laufzeitreset knüpft an das bereits funktionierende Cache-Konzept an.

Für öffentliche Schnittstellen, neue Quelldateien/Targets und gegebenenfalls Bibliotheksverknüpfungen muss der Folgeauftrag den nötigen Umfang ausdrücklich abdecken. Ein Eingriff in `../../tools/` ist für diesen Vorschlag nicht erforderlich. Ein Build bleibt gemäß Projektvorgabe dem Benutzer vorbehalten.

## 10. Vorgeschlagene Abnahmefälle für den Folgeauftrag

Diese Fälle sind nicht bereits durch die Untersuchung getestet:

1. Alter vollständiger KVS-Verbindungssatz wird genau einmal als Profil übernommen; Quelle und CSV-Pfad bleiben erhalten. Unvollständiger Altbestand geht nicht verloren. Vollständig leerer Bestand folgt der beschlossenen Regel.
2. Zwei Profile mit unterschiedlichen Datenbanken über Neustart laden; aktive Auswahl, fünf Felder, Hauptfensteranzeige und tatsächliches Datenziel stimmen überein.
3. Jedes der fünf Felder einzeln ändern: Neu/Löschen deaktiviert. Rückkehr zum Originalwert folgt dem beschlossenen Vergleichsvertrag. Programmgesteuertes Laden bleibt unverändert.
4. Neu setzt Auswahl und alle fünf Felder leer, bewahrt die Liste und ermöglicht das Speichern eines neuen Profils. Bearbeiten aktualisiert genau die gewählte ID, sofern dieses Verhalten beschlossen ist.
5. Löschen mit Nein verändert nichts; mit Ja verschwindet nur das Zielprofil. Keine Auswahl bleibt sowohl bei leerer als auch bei noch gefüllter Liste bestehen. Löschen ohne Auswahl ist wirkungslos/deaktiviert.
6. Speichern ohne gültiges Profil bei Quelle MariaDB folgt der beschlossenen Regel; nach letzter Löschung existiert kein unlösbarer Dialogzustand. CSV/Test/mrdb bleiben unabhängig davon wähl- und speicherbar.
7. Profilwechsel und Quellenwechsel mit geändertem Entwurf sowie Schließen nach neuer/gelöschter Profilmenge folgen dem definierten Commit-/Abbruchvertrag.
8. Fehlender KVS-Schlüssel, unzugänglicher KVS, doppelte Schlüssel, beschädigter Gesamtsatz, doppelte IDs und unbekannte Version werden unterschieden. Ein gültiger neuer Leerzustand importiert keine alten Werte erneut.
9. Erzwungener Schreibfehler setzt weder Feldvergleichsstand noch Laufzeitverbindung/Cache auf einen behaupteten Erfolg. Wiederöffnung/Neustart folgt der vereinbarten Wiederherstellung.
10. Manueller Test prüft den aktuellen Entwurf ohne Speicherung. Fehlerdiagnosen bleiben verständlich und enthalten kein Passwort; erfolgreicher Connect wird nicht als Nachweis aller ESTW-Rechte ausgegeben.
11. Isolierte Testdatenbanken A/B mit bewusst gleichen IDs verwenden: Nach Profilwechsel stammen Listen, Projekttexte, Element- und Fahrstraßenschreibzugriffe vollständig vom richtigen Datenbestand.
12. Bestehende MRDB-Regressionen bleiben erhalten: feste Enumwerte, versteckte MariaDB-Controls, kein Portzwang bei mrdb und unveränderte CSV-/Test-Lader. Persistenztests verwenden ein ausdrücklich isoliertes Testverzeichnis, nicht den fest verdrahteten Produktiv-KVS.
13. Namen, Benutzerdaten und Passwörter mit Umlauten, Anführungszeichen und Trennzeichen bleiben nach dem festgelegten Format unverändert; falls Passwortschutz gewählt wird, auch Migration und Entschlüsselungsfehler prüfen.

## 11. Prüfstand dieses Berichts

Die Konfigurationsstruktur, sämtliche direkten `ConfigData.Connection`-Verbraucher, alle fünf Connector-Erzeugungspfade, KVS- und mrdb-Dateispeicherung sowie die verwendbaren UI-Schnittstellen wurden statisch untersucht. Die Angaben zum Speicherformat beruhen auf dem Quelltext, nicht auf dem Öffnen produktiver KVS-Dateien. Die bestehenden Tests wurden als Fundstellen gelesen, nicht ausgeführt. Es wurden keine Codeänderungen und keine Änderungen an bestehender Dokumentation vorgenommen.

## 12. Konkrete Rückfragen vor der Implementierung

Die folgenden Fragen betreffen noch offene Entscheidungen. Die Vorschläge in der letzten Spalte sind keine bereits geltenden Anforderungen.

| ID | Rückfrage | Vorschlag zur Entscheidung |
| --- | --- | --- |
| F1 | Soll jedes Profil einen frei vergebbaren Namen erhalten, oder wird die Beschriftung aus `User@Host:Port/Database` gebildet? Welche Eindeutigkeit wird für Namen verlangt? | Zunächst abgeleitete Beschriftung vermeidet ein sechstes Eingabefeld; stabile technische ID unabhängig davon verwenden. |
| F2 | Dürfen mehrere Profile denselben Host, Port, Database und User haben, insbesondere bei verschiedenen Passwörtern? Wie sollen sie dann unterscheidbar sein? | Identität immer über ID; fachliche Dublettenregel ausdrücklich festlegen. Bei erlaubten Dubletten zusätzliche eindeutige Beschriftung verlangen. |
| F3 | Soll die gewählte Profil-ID dauerhaft gespeichert werden, und wird eine bloße Auswahl erst mit Setup-Speichern wirksam oder sofort? | Auswahl zusammen mit dem bestätigten Setup persistieren, bis dahin bisherige Laufzeitverbindung behalten. |
| F4 | Darf die Quelle MariaDB dauerhaft ohne aktive Verbindung gespeichert sein, etwa nach dem Löschen des letzten/aktiven Profils? Falls nein: Wie kann der Benutzer die Löschung abschließen? | Nichtauswahl zulassen, Datenzugriffe verständlich blockieren und Setup erreichbar lassen. Keine automatische Ersatzwahl. |
| F5 | Soll das bestehende Speichern weiter das gesamte Setup speichern und schließen, oder sollen Profile innerhalb des geöffneten Fensters gespeichert werden können? Wann wird eine bestätigte Löschung dauerhaft: sofort oder erst beim Abschluss? | Für die erste Stufe gesamter Arbeitsbestand mit gemeinsamem Abschluss; Abbruch verwirft noch nicht gespeicherte Verwaltungsänderungen. |
| F6 | Wird beim Speichern mit bestehender Auswahl genau dieses Profil aktualisiert und ohne Auswahl ein neues angelegt? Soll das gespeicherte neue Profil anschließend aktiv sein? | Bestehende ID aktualisieren; neue ID anlegen und bei erfolgreicher Übernahme als aktive Auswahl speichern. |
| F7 | Was geschieht bei Auswahl- oder Quellenwechsel mit geänderten Feldern: Wechsel blockieren, Verwerfen bestätigen oder vorher speichern? Soll das Wiederherstellen aller ursprünglichen Werte Neu/Löschen sofort aktivieren? | Originalwertvergleich; Wechsel bei geändertem Entwurf nur nach ausdrücklichem Verwerfen oder erfolgreichem Speichern. |
| F8 | Bleibt die Passwortspeicherung wie bisher oder wird neuer Schutz verlangt? Sollen Passwortmaskierung, Entfernen der Konsolenausgabe und Bereinigung alter Klartextschlüssel Teil des Folgeauftrags sein? | Maskierung und Entfernen der Ausgabe einplanen; dauerhaften Schutz und Altwertbereinigung separat verbindlich bestimmen. |
| F9 | Soll vor jedem Speichern ein erfolgreicher Verbindungstest verlangt werden oder bleibt Test freiwillig und der reguläre Zugriff erfolgt beim ersten Datenladen? | Test freiwillig halten; reguläres Laden bleibt verzögert. Ein obligatorischer Schema-/Rechtetest wäre eine zusätzliche Anforderung. |
| F10 | Wann wird eine Alt-Konfiguration dauerhaft migriert, und muss eine alte Programmversion danach weiter denselben aktiven Verbindungssatz lesen können? Wie werden ganz leere Altwerte behandelt? | Im Speicher importieren und beim ersten erfolgreichen Setup-Speichern festschreiben; keine allgemeine Rückwärtslesbarkeit alter Binärversionen zusagen. Ganz leere Altwerte eher als „kein Profil“ behandeln. |
| F11 | Reicht die bisherige KVS-Schreibsicherheit oder gehören Wiederherstellung bei Teilfehler/Abbruch und Schutz vor parallelen Setup-Änderungen zur ersten Stufe? | Einen definierten Schutzumfang wählen; ein einzelner JSON-KVS-Wert allein ist keine Atomaritätszusage. |
| F12 | Dürfen Ansichtseinstellungen gleicher Projekt-IDs in verschiedenen Profilen gemeinsam bleiben, oder sollen sie getrennt gespeichert werden? Soll das Hauptfenster zusätzlich Name/Database anzeigen? | Getrennte Ansichten und zusätzliche Anzeige als gesonderten kleinen Folgeschritt behandeln, falls die gemeinsame Nutzung zunächst akzeptiert wird. |

**Bereits geklärt und daher keine Rückfrage:** Nach bestätigtem Löschen bleibt die ComboBox ohne Auswahl, unabhängig von der Zahl verbliebener Profile. Ein automatisches Auswählen einer anderen Verbindung ist nicht Bestandteil der vorgegebenen Funktion.
