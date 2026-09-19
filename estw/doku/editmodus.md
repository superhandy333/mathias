# Edit-Modus

Im Editmodus können Elemente neu hinzugefügt, bearbeitet, verschoben oder gelöscht werden. Es gibt Unter-Modi innerhalb des Editmodus, die jeweils unterschiedliche Bearbeitungsfunktionen bereitstellen. 

## Element-Eigenchaften

### Elementtyp

Jedes Element ist von einem der folgenden Elementtypen:

- Gleis
- Weiche
- Signal
- Blindziel
- AuflöseElement

### Unterelementtyp

Jedes Element hat einen Unterelementtyp, der den Elementtyp näher spezifiziert.

Für Elemente vom Typ **Signal** ist die fachliche Bedeutung der Unterelementarten in [signale.md](signale.md#signalarten-nach-unterelementart) verbindlich festgelegt.

- 1
- 2
- 3

### Rotation

Die Rotation gibt an, um wie viele Grad das Element im Uhrzeigersinn gedreht ist.

- 0 (0°)
- 1 (90°)
- 2 (180°)
- 3 (270°)

### Mirror

Die Mirror-Eigenschaft gibt an, ob das Element gespiegelt ist.

- N (nicht gespiegelt)
- J (gespiegelt)

## Untermodi im Editmodus

- Auswahl
- Hinzufügen
- Verschieben
- Drehen
- Spiegeln
- Typ ändern
- Unterelementtyp ändern
- Löschen

Der aktuelle Untermodus wird im ToolForm durch ein SegmentControl angezeigt. Der Untermodus **Auswahl** ist ganz links vor allen anderen Untermodi angeordnet und verwendet als Symbol einen normalen Mauszeiger beziehungsweise Auswahlpfeil. Die Reihenfolge der übrigen Untermodi bleibt unverändert.

Beim Wechsel des Modus müssen eventuell begonnene, aber noch nicht abgeschlossene Aktionen zurückgesetzt werden. Das betrifft insbesondere ein im Modus „Verschieben“ bereits ausgewähltes Element.

## Elementauswahl und Bezeichnung

Die Eingabezeile für die Elementbezeichnung ist ausschließlich im Untermodus **Auswahl** sichtbar und nutzbar. Nur in diesem Untermodus darf die Bezeichnung eines Elements über diese Eingabezeile geändert werden. In allen anderen Untermodi sind die Bezeichnungsbeschriftung und die Eingabezeile ausgeblendet; eine Bezeichnungsänderung über diese Funktion ist dort ausgeschlossen.

Markierungen, die andere Untermodi für ihre jeweilige Bearbeitungsaktion benötigen, bleiben davon unberührt. Insbesondere darf der Untermodus „Verschieben“ sein Quellelement weiterhin vorübergehend markieren, ohne dadurch eine Änderung der Bezeichnung zu ermöglichen.

## Untermodus „Auswahl“

Dieser Untermodus dient ausschließlich dazu, ein vorhandenes Element auszuwählen beziehungsweise zu markieren und dessen Bezeichnung zu ändern.

Verhalten bei einem Mausklick auf das Gitter:

* Wird auf ein vorhandenes Element geklickt, wird genau dieses Element ausgewählt und markiert. Seine aktuelle Bezeichnung wird in der Eingabezeile des ToolForm angezeigt.
* Wird auf ein leeres Feld geklickt, wird die bestehende Auswahl aufgehoben. Die Eingabezeile wird geleert und deaktiviert, damit kein veralteter Wert übernommen werden kann.

Beim Wechsel in den Untermodus „Auswahl“ wird die Eingabezeile anhand des aktuellen Auswahlzustands aktualisiert. Ist bereits ein Element ausgewählt, bleibt diese Auswahl bestehen und ihre aktuelle Bezeichnung wird angezeigt. Ist kein Element ausgewählt, ist die Eingabezeile leer und deaktiviert.

Eine geänderte Bezeichnung wird wie bisher beim Verlassen des Eingabefelds validiert und in die Arbeitsdaten übernommen. Beim Speichern des Editmodus wird sie zusammen mit den übrigen Elementänderungen persistiert. Beim Verlassen des Untermodus „Auswahl“ werden die Eingabezeile und ihre Beschriftung unmittelbar ausgeblendet. Eine noch ausgelöste Übernahme aus einem anderen Untermodus muss wirkungslos bleiben.

## Untermodus „Hinzufügen“

Wenn dieser Modus aktiv ist, werden im ToolForm 4 Gruppen (SegmentControl) zusätzlicher Optionen für Elementtyp, Unterelementtyp, Rotation und Mirror angezeigt. Mit diesen SegmentControls wird die aktuelle Auswahl der jeweiligen Gruppe getroffen.

Verhalten bei einem Mausklick auf das Gitter:

* Wird auf ein leeres Feld geklickt, wird dort ein Element mit den aktuell ausgewählten Werten der 4 Gruppen eingefügt.
* Wird auf ein bereits belegtes Feld geklickt, erfolgt keine Änderung.

## Untermodus „Verschieben“

Das Verschieben erfolgt in zwei Schritten:

### Schritt 1: Quelle auswählen

* Wird auf ein belegtes Feld geklickt, wird das dort vorhandene Element als zu verschiebendes Element ausgewählt.
* Das ausgewählte Quellfeld muss visuell hervorgehoben werden.
* Wird auf ein leeres Feld geklickt, wird die Aktion abgebrochen beziehungsweise eine bestehende Auswahl aufgehoben.

### Schritt 2: Zielfeld auswählen

* Wird anschließend auf ein leeres Feld geklickt, wird das ausgewählte Element dorthin verschoben.
* Das ursprüngliche Feld wird danach geleert.
* Wird auf ein bereits belegtes Feld geklickt, wird der Verschiebevorgang abgebrochen und die Auswahl aufgehoben.
* Nach einem erfolgreichen oder abgebrochenen Verschiebevorgang befindet sich der Modus wieder im Ausgangszustand und wartet auf die Auswahl eines neuen Quellelements.

## Untermodus „Drehen“

Bei einem Klick auf ein vorhandenes Element wird dessen Rotation zyklisch geändert:

`0° → 90° → 180° → 270° → 0°`

* Jeder Klick schaltet genau zur nächsten Rotation weiter.
* Wird auf ein leeres Feld geklickt, erfolgt keine Änderung.

## Untermodus „Spiegeln“

Bei einem Klick auf ein vorhandenes Element wird dessen Mirror-Eigenschaft umgeschaltet:

`N (nicht gespiegelt) ↔ J (gespiegelt)`

* Wird auf ein leeres Feld geklickt, erfolgt keine Änderung.

## Untermodus „Typ ändern“

Wenn dieser Modus aktiv ist, wird im ToolForm ein SegmentControl für die Auswahl des Elementtyps angezeigt:

* Gleis
* Weiche
* Signal
* Blind
* Auflöse

Mit diesen Schaltflächen wird der neue Elementtyp ausgewählt. Der aktuell ausgewählte Typ muss eindeutig gekennzeichnet sein.

Verhalten bei einem Mausklick auf das Gitter:

* Wird auf ein vorhandenes Element geklickt, wird dessen Typ durch den aktuell ausgewählten Elementtyp ersetzt.
* Wird auf ein leeres Feld geklickt, erfolgt keine Änderung.

## Untermodus „Löschen“

* Wird auf ein belegtes Feld geklickt, wird das dort vorhandene Element gelöscht.
* Wird auf ein leeres Feld geklickt, erfolgt keine Änderung.

## Benutzeroberfläche

* Das Gitter nimmt den verfügbaren Bereich links neben dem ToolForm ein.
* Das ToolForm befindet sich dauerhaft am rechten Fensterrand.
* Die Schaltflächen müssen übersichtlich angeordnet und eindeutig beschriftet sein.
* Die Eingabezeile für die Elementbezeichnung und ihre Beschriftung dürfen nur im Untermodus „Auswahl“ sichtbar sein.
* Schaltflächen für Elementtypen dürfen nur in den Modi „Hinzufügen“ und „Typ ändern“ sichtbar sein.
* Mausklicks außerhalb des Gitters dürfen keine Gitteraktion auslösen.
* Die Zuordnung einer Mausposition zu einem Gitterfeld muss auch nach einer Änderung der Fenstergröße korrekt funktionieren.
* Verhindere sichtbares Flackern beim Neuzeichnen, beispielsweise durch Double Buffering.

## Erwartete Projektstruktur

Erstelle mindestens folgende Dateien:

* `CMakeLists.txt`
* `src/main.cpp`
* geeignete Header- und Implementierungsdateien für:

  * das Datenmodell des Gitters,
  * die Elemente,
  * die Bearbeitungslogik,
  * das Hauptfenster und die GDI+-Darstellung
* `README.md` mit Build- und Startanleitung

Passe die genaue Aufteilung an, wenn eine andere Struktur fachlich sinnvoller ist.

## Umsetzungsvorgehen

1. Erstelle zunächst eine kurze technische Planung mit Klassenstruktur, Zuständen und Ereignisabläufen.
2. Implementiere anschließend die Anwendung vollständig.
3. Konfiguriere die Verknüpfung der benötigten Windows- und GDI+-Bibliotheken über CMake.
4. Kompiliere das Projekt und behebe sämtliche Compilerfehler.
5. Prüfe insbesondere alle Untermodi, ungültige Mausklicks, Moduswechsel und das Verhalten nach einer Größenänderung des Fensters.
6. Fasse abschließend die erstellten Dateien, die Architektur und die durchgeführten Prüfungen kurz zusammen.

Die Aufgabe ist erst abgeschlossen, wenn das Projekt mit CMake erfolgreich konfiguriert und kompiliert werden kann und alle beschriebenen Bedienabläufe implementiert sind.
