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


## Neuer Untermodus für EditModus

Die derzeitige Funktionalität zum Ändern der Bezeichnung von Elementen im EditModus soll künftig von den bisherigen UnterModi getrennt werden. Diese Bezeichnungsänderungen sollen nur in einem neu schaffenden UnterModus möglich sein. Dieser Modus soll in der Leiste des Segmentcontrols für Untermodi ganz links vor allen anderen Buttons eingefügt werden und als Symbol auf dem neuen Button ein normales Mauszeigersymbol angezeigt werden. Nur in diesem Untermodi soll die Eingabezeile für Bezeichnung sichtbar sein. Schritt 1: Ändere die md-Dokumentation entsprechend ab, in dem du den neuen Untermodi an den passenden Stellen einfügst. Schritt 2: Implementiere den neuen Untermodi im Code.

## Fahrstrassenauswahl

Generiere einen für Codex optimierten Prompt: Füge im Simulationsmodus in der ToolForm einen Button "Verarbeiten" hinzu. In diesem Modus lassen sich nur Signale und Blindziele mit der linken Maustaste markieren. Wird auf ein Feld (Element) mit der linken Maustaste geklickt, welches kein Signal, Blindziel oder Auflöseelement ist, passiert nichts. Wird auf das Auflöseelement geklickt, wird das Element nicht markiert, sondern die Fahrstrassen-Auflöse-Mechanik der aktivierten Fahrstrasse gestartet werden, für die dieses Element projektiertes Auflöseelement ist. Ist im Simulationsmodus nur ein Element markiert, soll der Verarbeiten-Button deaktiviert sein. Implementiere eine Fahrstrassenauswahl-Mechanik im Simulationsmodus, die dann greift, wenn 2 Elemente markiert sind. Dann soll der Verarbeiten-Button aktiviert werden. Nach Klick auf den Button soll geprüft werden, ob es anhand der beiden markierten Elemente eine Fahrstrasse gibt, für die die Start-/Zielelement-Kombination zutrifft. Falls es so eine Fahrstrasse gibt, soll die Markierung der beiden Elemente gelöscht werden und die Logik der Fahrstrassenanschaltung mit dem Start der jeweiligen Zulasssungprüfung gestartet werden. Gibt es so eine Fahrstrasse nicht, soll eine Messagebox die Meldung "abgewiesen" bringen. Diese Message soll später mal in einer Art Kommandozeile ausgegeben werden, die später noch implementiert werden soll. 