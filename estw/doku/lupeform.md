# Form LupeForm

Das LupeForm kann in verschiedenen Modi geöffnet werden:

- Simulation
- Edit
- Fahrstasse

## Simulation

Im Simulationsmodus wird die "normale" ESTW-Logik ausgeführt.

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

