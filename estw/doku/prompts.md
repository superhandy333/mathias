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

## Fachliche Vorgaben für Zugstrassen

Für die Fahrstrassenlogik in Zugstrassen soll folgende fachliche Vorgabe für Signale bindend sein: Ein Signal besitzt in seiner Zustandslogik eine Hauptsignalgeschwindigkeit und eine Vorsignalgeschwindigkeit.

### Hauptsignalgeschwindigkeit

Die Hauptsignalgeschwindigkeit eines Signals gibt an, ob und mit welcher Geschwindigkeit ein Zug das Signal passieren darf.

Der ganzzahlige Wert der Hauptsignalgeschwindigkeit hat einen Wertebereich von -3 bis 16.

Als Haltstellung wird eine Hauptsignalgeschwindigkeit von 0 oder kleiner verstanden.

Als Fahrtstellung wird eine Hauptsignalgeschwindigkeit größer als 0 verstanden. Bei Fahrtstellung entspricht der Wert x 10 der maximal zulässigen Geschwindigkeit in km/h.

Als Rangiersignalstellung wird eine Hauptsignalgeschwindigkeit von -1 oder -2 verstanden.

|Wert|Bedeutung|
|-|-|
|-3||
|-2||
|-1||
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

Die Vorsignalgeschwindigkeit eines Signals gibt an, mit welcher Geschwindigkeit 