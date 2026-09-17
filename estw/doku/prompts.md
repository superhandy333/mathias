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

