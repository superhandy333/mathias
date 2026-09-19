# Form LupeForm

Das LupeForm kann in verschiedenen Modi geöffnet werden:

- Simulation
- Edit
- Fahrstasse

## Simulation

Im Simulationsmodus wird die "normale" ESTW-Logik ausgeführt.

### Fahrstraßenauswahl

Mit der linken Maustaste können im Simulationsmodus ausschließlich **Signale** und **Blindziele** für eine Fahrstraße markiert werden. Die beiden markierten Elemente bilden gemeinsam die Start-/Zielkombination; ihre Auswahlreihenfolge ist ohne Bedeutung. Eines der Elemente muss als **Fahrstraßenstart** und das andere als **Fahrstraßenziel** derselben Fahrstraße projektiert sein. Die Auswahl enthält höchstens zwei Elemente.

- Ein Klick auf ein bereits markiertes Element wählt es ab. Die relative Reihenfolge der verbleibenden Auswahl bleibt erhalten.
- Ein Klick auf ein weiteres Signal oder Blindziel ergänzt eine Auswahl mit weniger als zwei Elementen.
- Sind bereits zwei Elemente markiert, beginnt der Klick auf ein drittes zulässiges Element eine neue Auswahl; die bisherigen Markierungen werden entfernt und nur das angeklickte Element bleibt markiert.
- Ein Linksklick auf ein Gleis, eine Weiche oder einen sonstigen nicht zulässigen Elementtyp verändert die bestehende Auswahl nicht.

Das `ToolForm` enthält nur im Simulationsmodus den Button **„Verarbeiten“**. Er ist bei keiner oder genau einer Markierung deaktiviert und ausschließlich bei genau zwei Markierungen aktiviert. Beim Verlassen oder Zurücksetzen des Simulationsmodus und nach einer erfolgreich angenommenen Fahrstraßenanforderung werden Auswahl und Buttonzustand zurückgesetzt.

Beim Verarbeiten wird unabhängig von der Markierungsreihenfolge nach genau einer projektierten Fahrstraße gesucht, deren Fahrstraßenstart und Fahrstraßenziel gemeinsam den beiden markierten Elementen entsprechen. Keine passende Fahrstraße, mehrere Fahrstraßen mit derselben Start-/Zielkombination sowie eine von der zentralen Fahrstraßenlogik nicht annehmbare Anforderung werden mit der vorläufigen Meldung `abgewiesen` abgewiesen. Die Markierungen bleiben in diesen Fällen zur Korrektur bestehen. Die Ausgabe ist in einer eigenen Simulationsmeldungsfunktion gekapselt, damit sie später durch die geplante kommandozeilenartige Anzeige ersetzt werden kann.

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
- Element-Name
- Gitter
- Frame

## Speicherung der View-Optionen

Die View-Optionen aus Abschnitt `Menü Ansicht in LupeForm` sollen im Key-Value-Store pro Projekt gespeichert werden, so dass bei wiederholtem Öffnen des Projektes (Anzeigen des LupeForm) die View-Optionen wiederhergestellt werden. Die Mechanik des Key-Value-Store ist analog der Verwendung zu den configData zu bauen. Wenn die entsprechenden Key-Werte nicht existieren, sollen die View-Optionen auf Standardwerte gesetzt werden. Die View-Optionen sollen beim Beenden von LupeForm im Key-Value-Store gespeichert werden.

