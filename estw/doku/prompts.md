# Prompts für KI-Entwicklung estw3

## Persistierung

Im EditModus des LupeForm werden Elemente hinzugefügt, geändert oder gelöscht. Untersuche den derzeitigen Code-Stand und finde heraus, wie diese Änderungen in der Datenbank persistiert werden können. Diese Speichermöglichkeit soll nur möglich sein, wenn die DataSource im aktuellen Programmzustand auf `MariaDb`steht. Das LupeForm erhält die Daten im Aufruf als Instanz von fullProjekt. Analysiere die Struktur von fullProjekt und beschreibe, wie die Änderungen an den Elementen in der Datenbank gespeichert werden können. Achte darauf, dass die Persistierung nur dann erfolgt, wenn die DataSource auf `MariaDb` eingestellt ist. Verwende zur Analyse auch den Bericht in der Datei `..\estw\estw3\.github\plans\plan-lupeFormEditModePersistence.prompt.md`. Da seit der Erstellung dieses Berichts Änderungen am Code vorgenommen wurden, überprüfe, ob die dort beschriebenen Punkte noch aktuell sind. Beschreibe die notwendigen Schritte, um die Persistierung der Änderungen in der Datenbank zu implementieren, und gehe dabei auf die relevanten Codebereiche ein. 

## Klärung der Zuordnungen FahrstrassenElementTypen zu ElementTypen

Hier ist eine Aufstellung der Zuordnungen zwischen FahrstrassenElementTypen und ElementTypen:

ElementTyp
- Liste der möglichen FahrstrassenElementTypen

Gleis
- Fahrwegelement
- Flankenschutztransportelement

Weiche
- Fahrwegelement
- Flankenschutzelement
- Flankenschutztransportelement

Signal
- Fahrwegelement
- Fahrstrassenstart
- Fahrstrassenziel
- Flankenschutzelement
- Flankenschutztransportelement
- Vorwegelement
- Unterwegselement

Blindziel
- Fahrwegelement
- Fahrstrassenziel
- Flankenschutztransportelement

Auflöseelement
- Fahrwegelement
- Auflöseelement
- Flankenschutztransportelement

Analysiere bitte in den bestehenden md-Dateien, ob diese Zuordnungen korrekt und vollständig dokumentiert sind. Falls nicht, ergänze die fehlenden Zuordnungen und korrigiere eventuelle Fehler.

## Fachliche Vorgaben für Signale

Für Signale soll folgende fachliche Vorgabe bindend sein: 

Ein Element vom Typ Signal mit Unterelementart 1 ist im fachlichen Sinn ein Hauptsignal als Mehrabschnittssignal mit integriertem Rangiersignal.

Ein Element vom Typ Signal mit Unterelementart 2 ist im fachlichen Sinn ein Rangiersignal.

Ein Signal besitzt in seiner Zustandslogik eine Hauptsignalgeschwindigkeit und eine Vorsignalgeschwindigkeit.

### Hauptsignalgeschwindigkeit

Die Hauptsignalgeschwindigkeit (HSV) eines Signals gibt an, ob und mit welcher Geschwindigkeit ein Zug das Signal passieren darf.

Der ganzzahlige Wert der Hauptsignalgeschwindigkeit hat einen Wertebereich von -3 bis 16.

Als Haltstellung für Zugstrassen wird eine Hauptsignalgeschwindigkeit von 0 oder kleiner verstanden. 

Als Fahrtstellung für Zugstrassen wird eine Hauptsignalgeschwindigkeit größer als 0 verstanden. Bei Fahrtstellung entspricht der Wert x 10 der maximal zulässigen Geschwindigkeit in km/h.

Hauptsignalgeschwindigkeiten größer 0 sind für Rangierstrassen ungültig und daher als Haltstellung zu interpretieren.

Als Haltestellung für Rangierstrassen wird eine Hauptsignalgeschwindigkeit von 0 verstanden. 

Als Fahrtstellung für Rangierstrassen wird eine Hauptsignalgeschwindigkeit von -1 oder -2 verstanden.

Hauptsignalgeschwindigkeit -3 bedeutet, dass das Signal für die Fahrstrasse nicht relevant ist.

|Wert|Bedeutung|
|-|-|
|-3|Signal für die Fahrstrasse nicht relevant|
|-2|Fahrtstellung für Rangierstrassen (ohne Rot)|
|-1|Fahrtstellung für Rangierstrassen (mit Rot)|
|0|Halt|
|1|Fahrt mit maximal 10 km/h|
|2|Fahrt mit maximal 20 km/h|
|3|Fahrt mit maximal 30 km/h|
|4|Fahrt mit maximal 40 km/h|
|5|Fahrt mit maximal 50 km/h|
|6|Fahrt mit maximal 60 km/h|
|7|Fahrt mit maximal 70 km/h|
|8|Fahrt mit maximal 80 km/h|
|9|Fahrt mit maximal 90 km/h|
|10|Fahrt mit maximal 100 km/h|
|11|Fahrt mit maximal 110 km/h|
|12|Fahrt mit maximal 120 km/h|
|13|Fahrt mit maximal 130 km/h|
|14|Fahrt mit maximal 140 km/h|
|15|Fahrt mit maximal 150 km/h|
|16|Fahrt mit maximal 160 km/h|

### Vorsignalgeschwindigkeit

Die Vorsignalgeschwindigkeit (VSV) eines Signals ist nur für Zugstrassen relevant und gibt an, mit welcher Geschwindigkeit der Zug das im Fahrweg vorausliegende Hauptsignal passieren darf.

Der ganzzahlige Wert der Vorsignalgeschwindigkeit hat einen Wertebereich von 0 bis 16.

In der Fahrstraßenlogik wird die Vorsignalgeschwindigkeit aus der Hauptsignalgeschwindigkeit des Elementes ermittelt, welches in der Zugstrasse als Vorwegelement projektiert ist. Wenn kein Vorwegelement projektiert ist, wird das Zielelement zur Ermittlung der Vorsignalgeschwindigkeit verwendet. 

Für das Signalsystem KS gilt:

|VSV||
|-|-|
|0|KS2|
|>0|KS1|


## Neuer Untermodus für EditModus (erledigt)

Die derzeitige Funktionalität zum Ändern der Bezeichnung von Elementen im EditModus soll künftig von den bisherigen UnterModi getrennt werden. Diese Bezeichnungsänderungen sollen nur in einem neu schaffenden UnterModus möglich sein. Dieser Modus soll in der Leiste des Segmentcontrols für Untermodi ganz links vor allen anderen Buttons eingefügt werden und als Symbol auf dem neuen Button ein normales Mauszeigersymbol angezeigt werden. Nur in diesem Untermodi soll die Eingabezeile für Bezeichnung sichtbar sein. Schritt 1: Ändere die md-Dokumentation entsprechend ab, in dem du den neuen Untermodi an den passenden Stellen einfügst. Schritt 2: Implementiere den neuen Untermodi im Code.

## Fahrstrassenauswahl (erledigt)

Generiere einen für Codex optimierten Prompt: Füge im Simulationsmodus in der ToolForm einen Button "Verarbeiten" hinzu. In diesem Modus lassen sich nur Signale und Blindziele mit der linken Maustaste markieren. Wird auf ein Feld (Element) mit der linken Maustaste geklickt, welches kein Signal, Blindziel oder Auflöseelement ist, passiert nichts. Wird auf das Auflöseelement geklickt, wird das Element nicht markiert, sondern die Fahrstrassen-Auflöse-Mechanik der aktivierten Fahrstrasse gestartet werden, für die dieses Element projektiertes Auflöseelement ist. Ist im Simulationsmodus nur ein Element markiert, soll der Verarbeiten-Button deaktiviert sein. Implementiere eine Fahrstrassenauswahl-Mechanik im Simulationsmodus, die dann greift, wenn 2 Elemente markiert sind. Dann soll der Verarbeiten-Button aktiviert werden. Nach Klick auf den Button soll geprüft werden, ob es anhand der beiden markierten Elemente eine Fahrstrasse gibt, für die die Start-/Zielelement-Kombination zutrifft. Falls es so eine Fahrstrasse gibt, soll die Markierung der beiden Elemente gelöscht werden und die Logik der Fahrstrassenanschaltung mit dem Start der jeweiligen Zulasssungprüfung gestartet werden. Gibt es so eine Fahrstrasse nicht, soll eine Messagebox die Meldung "abgewiesen" bringen. Diese Message soll später mal in einer Art Kommandozeile ausgegeben werden, die später noch implementiert werden soll. 

## Abfrage von Element-Beanspruchungen

Generiere einen für codex optimierten Prompt: In den Bedingungen für die Zulassungsprüfung rangierstrasse soll geprüft werden, ob bei den für die Fahrstrasse benötigten Elementen bereits eine Beanspruchung vorliegt. Dabei ist es völlig unabhängig ob die Beanspruchung von der eigenen oder von einer fremden fahrstraße kommt.

   Im Bericht zur Analyse der Möglichkeiten zu Implementierung der Fahrstraßenlogik Rangierstraßen wird erwähnt, dass "Beanspruchungen weder Besitzer noch Referenzzählung besitzen. Damit lassen sich durch eine andere Fahrstraße und die konfliktfreie Nutzung nicht zuverlässig ausdrücken." Weder Besitzverhältnisse noch Referenzzählung sind jedoch erforderlich, da jeweils mit der Zulassungsprüfung lediglich geprüft werden soll, ob eine überhaupt eine Beanspruchung vorliegt oder nicht, und es egal ist, ob die aus der eigenen Fahrstraße herrührt oder aus fremden. Korrigiere bitte die Textstellen in der Dokumentation wo explizit auf "eine andere Fahrstraße" Bezug genommen wird, damit in späteren Analysen dieser vermeintliche Mangel nicht mehr erwähnt wird.



## Foreign Keys

Ergebnis
Nein, die MariaDB-Referenzintegrität wird in MRDB nicht vollständig nachgebildet.
Für alle sechs Foreign Keys gilt:
- Bei allen vorgesehenen Anwendungsschreibwegen wird der vollständige Zielzustand vor dem ersten MRDB-Schreibzugriff validiert. Ein erfolgreich abgeschlossener Anwendungsschreibvorgang hinterlässt daher keine verwaisten Datensätze.
- Löschungen mit weiterhin vorhandenen abhängigen Datensätzen werden abgewiesen. Beim Löschen einer Fahrstraße werden ihre Zuordnungen ausdrücklich mitgelöscht.
- Die MariaDB-Regel ON UPDATE CASCADE wird jedoch für keinen Foreign Key allgemein implementiert. Schlüssel sind lediglich im normalen UI-Ablauf unveränderlich. Der generische MRDB-Synchronisierer interpretiert eine Schlüsseländerung als Löschen plus Einfügen; alle abhängigen Schlüssel müsste der Aufrufer selbst ändern.
- Bereits inkonsistente oder extern manipulierte MRDB-Daten werden beim Laden zwar gemeldet, aber nicht abgewiesen.
- Die MRDB-„Transaktion“ schützt Anwendungsschreibvorgänge mittels Sperre, Journal, Rücksicherung und Lesevergleich. Sie ist jedoch keine echte Datenbanktransaktion und schützt nicht vor externen Werkzeugen, Stromausfall oder allen Hardwarefehlern.
Gesamtklassifikation für jeden Foreign Key: „nur durch die Benutzeroberfläche beziehungsweise den normalen Programmablauf geschützt“.
Die Unteraspekte „Referenzprüfung vor dem Schreiben“ und „konsistenter Endzustand nach erfolgreichem Schreiben“ sind innerhalb der Anwendung dagegen vollständig gewährleistet; die automatische Behandlung von Schlüsseländerungen ist nicht gewährleistet.
MariaDB-Schema und MRDB-Zuordnung
Grundlage ist das Projekt-DDL [create_database.sql (line 10)](D:/csharp/CPP/estw/sql/create_database.sql:10). Die Constraints sind dort nicht benannt; die Tabelle verwendet deshalb logische Bezeichnungen. Da ON DELETE jeweils fehlt, gilt implizit RESTRICT beziehungsweise NO ACTION. ON UPDATE CASCADE ist bei allen sechs Beziehungen explizit angegeben.
Alle produktiven MRDB-Änderungen laufen letztlich über:
- Database::write() mit Vorabvalidierung in [mrdbstore.cpp (line 267)](D:/csharp/CPP/estw/estw3/mrdbstore.cpp:267)
- requireValid() und isvalid_estwdata() in [mrdbstore.cpp (line 218)](D:/csharp/CPP/estw/estw3/mrdbstore.cpp:218) und [validate.cpp (line 157)](D:/csharp/CPP/estw/estw3/validate.cpp:157)
- die einzige produktive Verwendung von mrdb::Insert, mrdb::Update und mrdb::Delete in Database::synchronize() in [mrdbstore.cpp (line 365)](D:/csharp/CPP/estw/estw3/mrdbstore.cpp:365)
MariaDB-Foreign-Key	Beteiligte Tabellen und Spalten	MariaDB-Regel	Zugehöriger MRDB-Codepfad	Ergebnis	Bestehende Lücke
FK-E-P, DDL [Zeile 48 (line 48)](D:/csharp/CPP/estw/sql/create_database.sql:48)	elemente.projekt_id → projekte.id	ON DELETE RESTRICT/NO ACTION; ON UPDATE CASCADE	Projektanlage: main.cpp::showProjektListForm → estwMrdb::createProject() [mrdbstore.cpp (line 492)](D:/csharp/CPP/estw/estw3/mrdbstore.cpp:492). Element-CRUD: LupeForm::saveChanges → persistElementChanges → estwMrdb::saveProjectElements() [aktionen.cpp (line 1084)](D:/csharp/CPP/estw/estw3/aktionen.cpp:1084), [mrdbstore.cpp (line 518)](D:/csharp/CPP/estw/estw3/mrdbstore.cpp:518). Außerdem saveProject() und saveData().	Nur durch UI/normalen Ablauf geschützt. isvalid_estwdata() prüft für jedes Element die Existenz des Projekts. Eine Projektlöschung mit verbleibenden Elementen wird vor dem Schreiben abgewiesen.	Keine allgemeine Kaskade bei Änderung von projekte.id. Die UI bietet keine ID-Änderung oder Projektlöschung. saveData() verlangt bei einer Renummerierung, dass der Aufrufer sämtliche Element.ProjektId selbst anpasst.
FK-E-T, DDL [Zeile 49 (line 49)](D:/csharp/CPP/estw/sql/create_database.sql:49)	elemente.typ_id → elementtypen.id	ON DELETE RESTRICT/NO ACTION; ON UPDATE CASCADE	Elementanlage und Typänderung: LupeForm::addNeuesElement() beziehungsweise changeElementType() [lupeform.cpp (line 334)](D:/csharp/CPP/estw/estw3/lupeform.cpp:334), [lupeform.cpp (line 476)](D:/csharp/CPP/estw/estw3/lupeform.cpp:476) → persistElementChanges() → saveProjectElements(). Kataloginitialisierung in createProject(). Generische Änderungen über saveProject()/saveData().	Nur durch UI/normalen Ablauf geschützt. Der Typ muss vor jedem erfolgreichen Schreiben in Elementtypen vorhanden sein. Eine Kataloglöschung mit verbleibenden Elementen scheitert an der Gesamtvalidierung.	Keine Kaskade bei Änderung von elementtypen.id; Katalogschlüssel sind nur konventionell unveränderlich. createProject() ergänzt die Kataloge außerdem nur, wenn der jeweilige Katalog vollständig leer ist, nicht wenn einzelne IDs fehlen.
FK-F-P, DDL [Zeile 62 (line 62)](D:/csharp/CPP/estw/sql/create_database.sql:62)	fahrstrassen.projekt_id → projekte.id	ON DELETE RESTRICT/NO ACTION; ON UPDATE CASCADE	Fahrstraßenanlage/-änderung/-löschung: FahrstrassenEditor [fahrstrasseneditor.cpp (line 50)](D:/csharp/CPP/estw/estw3/fahrstrasseneditor.cpp:50) → persistFahrstrassenChanges() [aktionen.cpp (line 1305)](D:/csharp/CPP/estw/estw3/aktionen.cpp:1305) → saveProjectFahrstrassen() [mrdbstore.cpp (line 561)](D:/csharp/CPP/estw/estw3/mrdbstore.cpp:561). Außerdem saveProject()/saveData().	Nur durch UI/normalen Ablauf geschützt. Neue Fahrstraßen erhalten die feste Projekt-ID des geöffneten Editors. validateFahrstrassenChanges() und die Gesamtvalidierung weisen fehlende oder falsche Projekte ab.	Keine Kaskade bei Änderung von projekte.id. Eine Fahrstraßen-Projekt-ID kann in der UI nicht geändert werden; generische APIs erwarten eine vom Aufrufer vollständig angepasste Momentaufnahme.
FK-FE-F, DDL [Zeile 81 (line 81)](D:/csharp/CPP/estw/sql/create_database.sql:81)	fahrstrassenelemente.fahrstrasse_id → fahrstrassen.id	ON DELETE RESTRICT/NO ACTION; ON UPDATE CASCADE	Zuordnung: FahrstrassenEditor::toggleElement() [fahrstrasseneditor.cpp (line 90)](D:/csharp/CPP/estw/estw3/fahrstrasseneditor.cpp:90). Fahrstraßenlöschung: deleteSelected() entfernt die gesamte Fahrstraße mitsamt eingebetteten Zuordnungen. replaceFahrstrassenTables() entfernt alte Zuordnungen und Fahrstraßen gemeinsam [fahrstrassenpersistence.cpp (line 52)](D:/csharp/CPP/estw/estw3/fahrstrassenpersistence.cpp:52). Bei neuen Fahrstraßen ersetzt saveProjectFahrstrassen() die temporäre Fahrstraßen-ID in allen Zuordnungen. Außerdem saveProject()/saveData().	Nur durch UI/normalen Ablauf geschützt. Existenz wird geprüft; neue IDs werden korrekt in neue Zuordnungen übernommen. Fahrstraßenlöschung hinterlässt keine Zuordnungen.	Die Löschung ist eine manuelle Anwendungskaskade, obwohl das MariaDB-FK selbst RESTRICT ist. Für die Änderung einer bereits vorhandenen fahrstrassen.id existiert keine Kaskade; sie wird als Löschen/Einfügen behandelt.
FK-FE-E, DDL [Zeile 82 (line 82)](D:/csharp/CPP/estw/sql/create_database.sql:82)	fahrstrassenelemente.element_id → elemente.id	ON DELETE RESTRICT/NO ACTION; ON UPDATE CASCADE	Zuordnung über FahrstrassenEditor::toggleElement(), das nur Elemente des aktuellen Projekts akzeptiert. Speicherung über persistFahrstrassenChanges()/saveProjectFahrstrassen(). Elementlöschung über LupeForm::deleteElement() [lupeform.cpp (line 516)](D:/csharp/CPP/estw/estw3/lupeform.cpp:516) → persistElementChanges()/saveProjectElements(). Außerdem saveProject()/saveData().	Nur durch UI/normalen Ablauf geschützt. Die Löschung eines weiterhin referenzierten Elements wird durch die Kandidatenvalidierung beziehungsweise spätestens durch Database::write() abgewiesen. Dafür existiert auch ein Test.	Keine Kaskade bei Änderung von elemente.id. IDs sind im Editor nicht bearbeitbar; generische APIs müssen sämtliche ElementId-Werte selbst remappen.
FK-FE-T, DDL [Zeile 83 (line 83)](D:/csharp/CPP/estw/sql/create_database.sql:83)	fahrstrassenelemente.fahrstrassenelementtyp_id → fahrstrassenelementtypen.id	ON DELETE RESTRICT/NO ACTION; ON UPDATE CASCADE	Rollenauswahl aus den fest codierten IDs 1–8 in [fahrstrasseneditor.h (line 14)](D:/csharp/CPP/estw/estw3/fahrstrasseneditor.h:14); Schreiben über toggleElement() → persistFahrstrassenChanges() → saveProjectFahrstrassen(). Kataloginitialisierung über createProject(). Außerdem saveProject()/saveData().	Nur durch UI/normalen Ablauf geschützt. isvalid_estwdata() verlangt, dass TypId im Katalog existiert. Eine Löschung eines verwendeten Typs wird vor dem Schreiben abgewiesen.	Keine Kaskade bei Änderung von fahrstrassenelementtypen.id. Die UI setzt stabile IDs voraus. Ein teilweise vorhandener Katalog wird durch createProject() nicht repariert; Schreibversuche mit fehlenden Rollen werden lediglich abgewiesen.


Prüfung der geforderten Einzelaspekte
Referenzierte Datensätze vor dem Speichern
Für alle sechs Beziehungen vollständig gewährleistet, sofern über die MRDB-Zugriffsschicht geschrieben wird.
Database::write() ruft requireValid() noch vor ID-Prüfung, Sicherung und erstem Schreibzugriff auf. Die eigentlichen FK-Prüfungen stehen in:
- Element.ProjektId und Element.TypId: [validate.cpp (line 258)](D:/csharp/CPP/estw/estw3/validate.cpp:258)
- Fahrstrasse.ProjektId: [validate.cpp (line 334)](D:/csharp/CPP/estw/estw3/validate.cpp:334)
- alle drei Referenzen des Fahrstraßenelements: [validate.cpp (line 371)](D:/csharp/CPP/estw/estw3/validate.cpp:371)
Die fachbezogenen Persistenzfunktionen validieren zusätzlich früher:
- Elemente: validateCandidate() vor saveProjectElements()
- Fahrstraßen: validateFahrstrassenChanges() vor und innerhalb von saveProjectFahrstrassen()
Damit kann auch ein direkter Aufruf von estwMrdb::saveData() oder saveProject() keinen ungültigen Endzustand schreiben.
Verhinderung verwaister Datensätze
Nach einem erfolgreich abgeschlossenen Anwendungsschreibvorgang vollständig gewährleistet. Das gilt auch für die öffentlichen Vollbestandsfunktionen saveData() und saveProject().
Nicht vollständig gewährleistet ist die Integrität der MRDB-Datenquelle als solcher:
- estwMrdb::loadData() liest nur Tabellen, Datentypen und eindeutige IDs, ruft aber keine Gesamtvalidierung auf [mrdbstore.cpp (line 444)](D:/csharp/CPP/estw/estw3/mrdbstore.cpp:444).
- generate_estw_data() validiert anschließend, gibt den ungültigen Bestand aber trotzdem zurück [aktionen.cpp (line 786)](D:/csharp/CPP/estw/estw3/aktionen.cpp:786).
- main.cpp zeigt die Fehler an und setzt den Bestand dennoch als geladen [main.cpp (line 283)](D:/csharp/CPP/estw/estw3/main.cpp:283).
Ein extern erzeugter verwaister MRDB-Datensatz kann daher in die laufende Anwendung gelangen. Beim Aufbau von FullProjekte können solche Datensätze zusätzlich aus den aggregierten Projektansichten herausfallen, obwohl sie in den flachen Tabellen weiter vorhanden sind.
Löschsperren und Kaskaden
- Projekt-, Elementtyp- und Fahrstraßenelementtyp-Löschungen werden von der UI derzeit nicht angeboten. Der Schutz beruht dort primär auf dem fehlenden Bedienweg und sekundär auf der Gesamtvalidierung der generischen APIs.
- Das Löschen eines referenzierten Elements wird zuverlässig abgewiesen.
- Das Löschen einer Fahrstraße löscht bewusst auch alle Zuordnungen. Das ist eine manuelle Anwendungskaskade; sie entspricht nicht der isolierten MariaDB-Regel ON DELETE RESTRICT, erzeugt aber denselben konsistenten Endzustand wie „Kinder zuerst, danach Eltern“.
- saveData() arbeitet mit vollständigen Soll-Momentaufnahmen. Entfernt der Aufrufer Eltern und Kinder gemeinsam, wird die Änderung akzeptiert. Das ist kein automatisches FK-Verhalten, sondern eine explizite Vollbestandssynchronisation.
Schlüsseländerungen
Für alle sechs Beziehungen nicht gewährleistet.
Database::synchronize() ordnet Datensätze anhand der fachlichen Id zu. Eine geänderte ID wird daher als neuer Datensatz eingefügt und die alte ID anschließend gelöscht. Es gibt keine Alt-ID/Neu-ID-Zuordnung und keine automatische Aktualisierung abhängiger Fremdschlüssel.
Praktische Auswirkung:
- Ändert ein Aufrufer nur den Primärschlüssel, wird der Zielbestand wegen der alten Referenzen abgewiesen.
- Ändert er Primär- und Fremdschlüssel vollständig koordiniert, wird der Bestand akzeptiert.
- MariaDB würde dagegen bereits aus der Änderung des Primärschlüssels allein die abhängigen Werte automatisch kaskadieren.
Dass Schlüssel in der aktuellen UI nicht bearbeitbar sind, verhindert das Problem lediglich im normalen Bedienablauf.
Zusammengesetzte Schreibvorgänge und Fehler
Für gewöhnliche, erkennbare Schreibfehler ist der Schutz gut:
1. Vollständige Validierung vor dem ersten Schreibzugriff.
2. Sicherung aller sechs ESTW-Tabellen.
3. ready-Journalmarker.
4. Synchronisation und vollständiger Rücklesevergleich jeder Tabelle.
5. Bei Ausnahme Rücksicherung aller sechs Tabellen.
6. Wiederherstellung eines liegen gebliebenen Journals beim nächsten Zugriff.
7. Prozessinterner Mutex und exklusive .estw.lock-Datei.
Die Implementierung steht in Database::write(), backup() und recover() in [mrdbstore.cpp (line 267)](D:/csharp/CPP/estw/estw3/mrdbstore.cpp:267).
Einschränkungen:
- Es handelt sich nicht um eine MRDB-Transaktion mit Isolation und Durability.
- Die physische Synchronisationsreihenfolge ist Projekte → Elementtypen → Elemente → Fahrstraßen → Fahrstraßenelementtypen → Fahrstraßenelemente. Bei Löschungen werden daher Eltern physisch vor Kindern gelöscht. Während des Schreibens können vorübergehend verwaiste Datensätze existieren.
- Anwendungseigene Leser werden durch die Sperre ferngehalten, externe MRDB-Werkzeuge beachten diese Sperre aber nicht zwingend.
- Bei einem normalen Fehler wird zurückgesichert; bei Prozessabbruch wird beim nächsten Zugriff wiederhergestellt. Eine harte Zusicherung bei Strom- oder Hardwareausfall besteht ausdrücklich nicht.
Daher: für Anwendungsausnahmen weitgehend vollständig gewährleistet, als datenbankweite ACID-Garantie dagegen nur durch den normalen Programmablauf geschützt.
Zusätzliche Beziehung außerhalb der Foreign Keys
MariaDB besitzt zusätzlich zwei Trigger, die bei INSERT und UPDATE von fahrstrassenelemente prüfen, ob das referenzierte Element und die referenzierte Fahrstraße demselben Projekt angehören [create_database.sql (line 87)](D:/csharp/CPP/estw/sql/create_database.sql:87).
MRDB bildet diese Regel in isvalid_estwdata() nach [validate.cpp (line 396)](D:/csharp/CPP/estw/estw3/validate.cpp:396). Außerdem lässt FahrstrassenEditor::toggleElement() nur Elemente des geöffneten Projekts zu.
Diese Regel ist für Anwendungsschreibvorgänge vollständig gewährleistet und teilweise sogar stärker als der MariaDB-Trigger: Die MRDB-Gesamtvalidierung erkennt die Abweichung auch nach einer späteren Änderung von Element.ProjektId oder Fahrstrasse.ProjektId, während der MariaDB-Trigger nur bei einer Änderung am Fahrstraßenelement selbst ausgeführt wird.
Es gibt keinen MariaDB-Foreign-Key für fahrstrassen.typ_id; dort existiert lediglich ein CHECK (typ_id IN (1, 2)). Auch unterelementart ist keine Tabellenreferenz.
Vorhandene Tests und Abdeckungslücken
Vorhanden sind unter anderem:
- Ablehnung der Löschung eines referenzierten Elements: [mrdb_tests.cpp (line 134)](D:/csharp/CPP/estw/estw3/tests/mrdb_tests.cpp:134)
- Rücksicherung nach einem Schreibfehler in einer früheren Tabelle: [mrdb_tests.cpp (line 159)](D:/csharp/CPP/estw/estw3/tests/mrdb_tests.cpp:159)
- Rücksicherung eines zusammengesetzten Fahrstraßenschreibvorgangs: [fahrstrassen_tests.cpp (line 205)](D:/csharp/CPP/estw/estw3/tests/fahrstrassen_tests.cpp:205)
- Löschung einer Fahrstraße ohne verbleibende Zuordnungen: [fahrstrassen_tests.cpp (line 218)](D:/csharp/CPP/estw/estw3/tests/fahrstrassen_tests.cpp:218)
Nicht systematisch getestet werden:
- ungültige Inserts und Updates für jeden der sechs Foreign Keys,
- Löschsperren für Projekte und beide Typkataloge,
- sämtliche ON UPDATE CASCADE-Fälle,
- das Laden bereits verwaister MRDB-Daten,
- parallele Zugriffe durch Werkzeuge, die .estw.lock ignorieren,
- Abbrüche in jeder einzelnen Phase des sechs Tabellen umfassenden Schreibvorgangs.
Der Bericht beruht auf statischer Analyse; ich habe keinen Build oder Testlauf ausgelöst.
Empfehlungen
1. Schlüsselstrategie verbindlich festlegen. Am einfachsten ist, alle fachlichen Primärschlüssel nach ihrer Erzeugung ausdrücklich als unveränderlich zu definieren und dies auch in saveData() und saveProject() zu prüfen. Falls Schlüsseländerungen unterstützt werden sollen, muss eine zentrale Remap-Struktur sämtliche sechs ON UPDATE CASCADE-Beziehungen aktualisieren.
2. MRDB beim Laden „fail closed“ behandeln. estwMrdb::loadData() sollte den vollständigen gelesenen Bestand validieren und bei Referenzfehlern abbrechen. Die Anwendung sollte ungültige Daten nicht als erfolgreich geladen markieren.
3. Expliziten Mutationsplan verwenden. Inserts sollten Eltern vor Kindern, Deletes Kinder vor Eltern ausführen. Dadurch sinkt auch ohne echte Constraints das Zeitfenster vorübergehender Inkonsistenz.
4. Bulk-Schnittstellen absichern. saveData() und saveProject() sollten entweder als interne Import-/Migrationsschnittstellen gekennzeichnet werden oder dieselben unveränderlichen Schlüsselregeln wie die Editoren erzwingen.
5. Tests je Foreign Key ergänzen. Pro Beziehung mindestens: fehlender Elternsatz bei Insert, ungültige Änderung des Fremdschlüssels, Elternlöschung mit bestehendem Kind, gemeinsame Eltern-/Kindlöschung und definierte Reaktion auf Primärschlüsseländerung.
6. Katalogvollständigkeit prüfen. Nicht nur leere Kataloge initialisieren, sondern die benötigten stabilen IDs für Elementtypen und Fahrstraßenelementtypen explizit prüfen.
7. Für echte Datenbankgarantien MRDB erweitern. Langfristig wären deklarative Constraints oder ein zentraler MRDB-Commitmechanismus mit atomarem Verzeichniswechsel und dauerhafter Journal-Synchronisation erforderlich.

## Frage zu Punkt 2

Punkt 2 meint den Übergang von den gespeicherten Projektierungsdaten zu den Laufzeitobjekten der Simulation.
In den Stammdaten besitzt jedes FahrstrassenElement unter anderem:
- ElementId
- TypId – seine Rolle in der Fahrstraße
- SollStellung – beispielsweise -1/1 für die benötigte Weichenlage
Das ist in [structs.h (line 74)](D:/csharp/CPP/estw/estw3/structs.h:74) definiert.
Beim Aufbau der Simulation erzeugt LupeForm daraus ein FsInitData. Darin stehen hauptsächlich Zeiger auf die Laufzeitelemente: Startsignal, Ziel, Auflöseelement sowie Listen von Fahrwegelementen. Die SollStellung wird aber nicht unmittelbar zusammen mit dem jeweiligen Laufzeitelement abgelegt. Außerdem befüllt der aktuelle Aufbau nur die Rollen 1, 2, 3, 4 und 6; die vorhandenen Listen für die Rollen 5, 7 und 8 werden nicht gefüllt. Siehe [fs.h (line 39)](D:/csharp/CPP/estw/estw3/estw/fs.h:39) und [lupeform.cpp (line 884)](D:/csharp/CPP/estw/estw3/lupeform.cpp:884).
Für eine Rangierstraße wird die SollStellung insbesondere benötigt, um:
- bei den Zulassungsbedingungen 4 und 5 festzustellen, welche Weichenlage widersprechend wäre,
- in Status 1 den benötigten Weichenstrang B oder C zu beanspruchen,
- in Status 2 die Weiche gezielt nach links oder rechts zu stellen,
- in Status 4 die projektierte Rangierfahrtstellung des Startsignals einzustellen.
Die Formulierung „verwirft die SollStellung“ ist allerdings zu stark: Fs speichert zusätzlich eine vollständige Kopie der Fahrstrasse, einschließlich ihrer Fahrstrassenelemente und damit auch der SollStellung; siehe [fs.cpp (line 5)](D:/csharp/CPP/estw/estw3/estw/fs.cpp:5). Die Information ist also noch vorhanden. Es fehlt vielmehr eine direkte, bequem nutzbare Verbindung:
Laufzeit-Weiche ↔ Rolle in der Fahrstraße ↔ SollStellung
Die Logik könnte diese Verbindung anhand der ElementId wiederherstellen. Punkt 2 beschreibt daher eine ungünstige und unvollständige Laufzeitaufbereitung, aber keinen vollständigen Informationsverlust.
Auch die Aussage zu den „mehreren Rollen“ ist für Rangierstraßen nur eingeschränkt relevant: Die Rollen 5 bis 8 sind dort laut Fachvorgabe ohnehin verboten. Sie werden vor allem für spätere Zugstraßen benötigt.
Präziser wäre Punkt 2 somit:
Die Laufzeitprojektion ordnet den beteiligten Laufzeitelementen Rolle und Sollstellung nicht unmittelbar zu und befüllt einige vorgesehene Rollen nicht. Die vollständigen Fahrstraßendaten bleiben zwar in Fs erhalten, müssen für die Fahrstraßenlogik aber anhand der Element-ID erneut mit den Laufzeitelementen verknüpft werden.

## Ergänzung Regel Zulassungsprüfung Rangierstraßen um die Programmfälle PR (erledigt)

Generiere einen für Codex optimierten Prompt: Ergänze die Regeln für Zulassungsprüfung Rangierstraßen um folgende zwei Bedingungen:

(1) Bei Fahrstrassenelementen von ElementTyp Gleis Darf die Zulassungsprüfung nur dann positiv verlaufen, wenn der Programmfall PR gesetzt ist. 

(2) Bei Signalen, die in der Rangierstrasse als Zielsignal dienen, verläuft die Zulassungsprüfung nur dann positiv, wenn (neben den anderen Bedingungen) der Programmfall PZSP nicht gesetzt ist oder, falls dieser Programmfall gesetzt ist, das Signal nicht gerade in einer Zugstrasse als Flankenschutzelement dient.

## Regelprüfung für Fahrstrassen nach Fahrstrassenedit nur Warnung (erledigt)

Generiere einen für Codex optimierten Prompt: Nach dem Edit einer Fahrstrasse werden die Bedingungen für eine gültige Fahrstrasse überprüft, und im Fehlerfall als harte Fehlermeldung ausgegeben. Danach wird die Speicherung der geänderten Fahrstrassen nicht durchgeführt. Ändere den Code so, dass die Meldungen zwar angezeigt werden, nicht vollständige oder fehlerhafte Fahrstrassen (bzw. alle geänderten Fahrstrassen) trotzdem gespeichert werden. Damit soll verhindert werden, dass Änderungen an den Fahrstrassen verworfen werden müssen, wenn auf Grund von noch vorhandenen topologischen Fehlern, Fahrstrassenelemente nicht gewählt werden können, welche für die Gültigkeit der Fahrstrasse aber vorhanden sein müssen.

## Einführung PropertyGrid im EditModus (erledigt)

Generiere einen für Codex optimierten Prompt: Ersetze die bisherige Textbox für die Eingabe des bezeichnungsstrings im EditModus (untermodus Auswahl) durch ein PropertyGrid, das die Bearbeitung der Eigenschaften des selektierten Elements ermöglicht. Damit soll es nicht nur möglich sein die Bezeichnung des selektierten Elementes zu ändern sondern auch die Programmfälle und andere Eigenschaften. Die Liste der Programmfälle, die ein Element unterstützt, kann mit der Methode `getProgrammfaelle()` abgerufen werden. Schaffe eine Funktionalität im PropertyGrid, die es erlaubt, die Programmfälle zu aktivieren oder zu deaktivieren. Die Datenänderung soll im Modus mrdb und MariaDb möglich sein. In der Art, wie bereits jetzt die Persistierung der Bezeichnung erfolgt, soll auch die Persistierung der Programmfälle und anderer Eigenschaften erfolgen.

## WeichenLupeElement unterscheidet Simulation- und EditModus

Generiere einen für GPT-5.6 Sol optimierten Prompt: Das WeichenLupeElement soll künftig, wenn es im EditModus dargestellt werden soll, so dargestellt werden, dass beide Weichenlagemelder (links und rechts) gleichzeitig sichtbar sind. Im FahrstrassenEditModus soll es wie im Simulationsmodus dargestellt werden, aber mit in der Lage, wie es aus der Bestimmung der Solllage hervorgeht. Also -1 für links und 1 für rechts. Wird bei der Bearbeitung im FahrstrassenEditModus die sollstellung geändert, soll die Anzeige entsprechend aktualisiert werden.

## Prüfung der Sollstellung in den Estwdaten (erledigt)

Füge in die validierungsroutine von isvalid_estwdata() eine Prüfung ein, die prüft, ob bei Fahrstrassenelementen, die auf EstwElemente eines bestimmten Elementtyps referenzieren, nur bestimmte Sollstellungen zulässig sind.

|Elementtyp|Zulässige Sollstellung|
|-|-|
|Gleis|0|
|Weiche|-1,1|
|Signal|-3..16|
|AuslöseElement|0|
|Blind|0|

Füge diese regeln auch als Dokumentation in die Datei rules_isvalid_estwdata.md ein.