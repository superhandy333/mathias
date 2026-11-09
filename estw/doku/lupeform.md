# Form LupeForm

Das LupeForm kann in verschiedenen Modi geöffnet werden:

- Simulation
- Edit
- Fahrstasse

## Simulation

Im Simulationsmodus wird die "normale" ESTW-Logik ausgeführt.

Das LupeForm startet bei jedem Öffnen im Simulationsmodus maximiert. Das separate ToolForm liegt am unteren Rand des Monitor-Arbeitsbereichs, auf dem das LupeForm dargestellt wird. Es hat beim Öffnen dieselbe sichtbare Fensterbreite wie das LupeForm und ein Fünftel seiner Fensterhöhe. Bei Größen- oder Positionsänderungen des LupeForm werden Größe und Position des ToolForm entsprechend angepasst. Das ToolForm bleibt manuell in der Größe veränderbar; seine Buttons bleiben dabei unverändert 100 × 25 Pixel groß und sind mit 10 Pixel Abstand am rechten Rand untereinander angeordnet.

Links neben den Aktionsbuttons enthält das Werkzeugfenster ausschließlich im Simulationsmodus eine `Win32Form::CommandLine`. Sie nutzt die verfügbare Breite und Höhe mit Abstand zum Fensterrand und zu den Buttons und wird bei Größenänderungen angepasst. Simulationsmeldungen werden dort als Historienzeilen ausgegeben. Die Historie behält entsprechend der Komponente nur die sichtbare Zeilenanzahl; ältere Zeilen entfallen. Reset löscht die Historie nicht. Das Eingabefeld ist vorhanden; eine Auswertung eingegebener Befehle ist noch nicht implementiert.

### Fahrstraßenauswahl

Mit der linken Maustaste können im Simulationsmodus ausschließlich **Signale** und **Blindziele** für eine Fahrstraße markiert werden. Die beiden markierten Elemente bilden gemeinsam die Start-/Zielkombination; ihre Auswahlreihenfolge ist ohne Bedeutung. Eines der Elemente muss als **Fahrstraßenstart** und das andere als **Fahrstraßenziel** derselben Fahrstraße projektiert sein. Die Auswahl enthält höchstens zwei Elemente.

- Ein Klick auf ein bereits markiertes Element wählt es ab. Die relative Reihenfolge der verbleibenden Auswahl bleibt erhalten.
- Ein Klick auf ein weiteres Signal oder Blindziel ergänzt eine Auswahl mit weniger als zwei Elementen.
- Sind bereits zwei Elemente markiert, beginnt der Klick auf ein drittes zulässiges Element eine neue Auswahl; die bisherigen Markierungen werden entfernt und nur das angeklickte Element bleibt markiert.
- Ein Linksklick auf ein Gleis, eine Weiche oder einen sonstigen nicht zulässigen Elementtyp verändert die bestehende Auswahl nicht.

Das `ToolForm` enthält nur im Simulationsmodus den Button **„Verarbeiten“**. Er ist bei keiner oder genau einer Markierung deaktiviert und ausschließlich bei genau zwei Markierungen aktiviert. Beim Verlassen oder Zurücksetzen des Simulationsmodus und nach einer erfolgreich angenommenen Fahrstraßenanforderung werden Auswahl und Buttonzustand zurückgesetzt.

Beim Verarbeiten werden unabhängig von der Markierungsreihenfolge die projektierten Fahrstraßen ermittelt, deren Fahrstraßenstart und Fahrstraßenziel gemeinsam den beiden markierten Elementen entsprechen. Die Bezeichnung (`Fahrstrasse.Bezeichnung`) jeder passenden Fahrstraße wird vor der Anforderung als eigene Zeile in der Commandline ausgegeben. Das gilt auch bei mehrdeutiger Auswahl oder negativer Zulassungsprüfung. Keine passende Fahrstraße, mehrere Fahrstraßen mit derselben Start-/Zielkombination sowie eine von der zentralen Fahrstraßenlogik nicht annehmbare Anforderung führen anschließend zur Meldung `abgewiesen` in derselben Commandline. Die Markierungen bleiben in diesen Fällen zur Korrektur bestehen. Die Ausgabe ist in `LupeForm::showSimulationMessage()` gekapselt; hierfür wird keine Messagebox geöffnet.

Die Meldung enthält den von der zentralen Anforderungsprüfung gelieferten Grund. Bei einer elementbezogenen Ursache lautet sie beispielsweise `abgewiesen: S12 (ID 47) – Signalsperre eingelegt`. Auch bei mehreren verursachenden Elementen wird genau eines genannt: das erste in der Prüfreihenfolge. Allgemeine Ursachen werden ohne Element ausgegeben, beispielsweise `abgewiesen: Keine passende Fahrstraße` oder `abgewiesen: Fahrstraße bereits aktiv`. Die Oberfläche wertet die Zulassungsbedingungen hierfür nicht erneut aus; sie verwendet das zurückgegebene Ergebnis gemäß [fahrstrasse.md](fahrstrasse.md#diagnose-bei-abweisung).

Bei genau einer passenden und annehmbaren Fahrstraße werden beide Markierungen entfernt und der Button deaktiviert. Anschließend startet die zentrale Fahrstraßenlogik die Anschaltung mit der einmaligen Zulassungsprüfung; es wird kein separater Anschaltungsablauf in der Bedienoberfläche geführt.

### Auflöseelemente

Ein Linksklick auf ein **Auflöseelement** markiert es nicht und verändert eine bestehende Start-/Zielauswahl nicht. Stattdessen werden über die zentrale Fahrstraßenlogik alle diesem Element zugeordneten Fahrstraßen betrachtet. Die Auflösung wird nur für diejenigen passenden Fahrstraßen angefordert, deren aktueller Zustand die Auflösung zulässt; ohne passende aktive Fahrstraße bleibt der Klick ohne sichtbare Wirkung.

## Edit

Im Editmodus können Elemente neu hinzugefügt, bearbeitet, verschoben oder gelöscht werden.

Die genauen Vorgaben für diesen Modus sind in der Datei [editmodus.md](editmodus.md) beschrieben. Beachte diese Datei. Du hast lesenden Zugriff auf diese Datei. Änderungen an dieser Datei sind nicht erlaubt.

## Fahrstrasse

Im Fahrstraßenmodus können Elemente als Fahrstrassenelemente Fahrstrassen hinzugefügt, bearbeitet oder gelöscht werden.

Das Form LupeForm hat eine MenuBar.


## Menü Ansicht in LupeForm

In der MenuBar des Forms LupeForm gibt es das Menü Ansicht. Darin sind anhackbare MenuItems, die View-Optionen (true/false) steuern. z.B. Sind die ID's der Elemente in der Lupe-Ansicht sichtbar?

Das Menü soll folgende MenuItems haben:

- Element-ID
- Elementname
- Elementnamen (alle)
- Gitter
- Frame

**Elementname** zeigt ausschließlich die Namen von Signalen, Weichen und Blindzielen an. **Elementnamen (alle)** zeigt die Namen sämtlicher Elementtypen an und hat Vorrang vor der eingeschränkten Anzeige. Sind beide Punkte ausgeschaltet, werden keine Elementnamen angezeigt. Die Regel gilt auch für neu hinzugefügte oder im Editmodus ersetzte Elemente.

## Speicherung der View-Optionen

Die View-Optionen aus Abschnitt `Menü Ansicht in LupeForm` sollen im Key-Value-Store pro Projekt gespeichert werden, so dass bei wiederholtem Öffnen des Projektes (Anzeigen des LupeForm) die View-Optionen wiederhergestellt werden. Die Mechanik des Key-Value-Store ist analog der Verwendung zu den configData zu bauen. Wenn die entsprechenden Key-Werte nicht existieren, sollen die View-Optionen auf Standardwerte gesetzt werden. Die View-Optionen sollen beim Beenden von LupeForm im Key-Value-Store gespeichert werden.

Die eingeschränkte Option **Elementname** verwendet weiterhin den bestehenden Schlüssel `element_name_visible` (Standard: eingeschaltet). **Elementnamen (alle)** wird separat unter `all_element_names_visible` gespeichert (Standard: ausgeschaltet). Beide Schlüssel erhalten wie die übrigen View-Optionen das projektbezogene Präfix.

