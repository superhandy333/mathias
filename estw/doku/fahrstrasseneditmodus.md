# FahrstrassenEdit-Modus

Der FahrstrassenEdit-Modus dient dazu, die Fahrstrassen eines Projekts zu bearbeiten. In diesem Modus können Fahrstrassen angelegt, geändert und gelöscht sowie den Fahrstrassen Elemente der Topologie mit einer bestimmten Funktion im Sinne der Fahrstrasse zugewiesen oder wieder entzogen werden.

Die Zuordnung eines Elements zu einer Fahrstrasse wird durch einen Datensatz in der Tabelle `FahrstrassenElemente` abgebildet. Der Datensatz verknüpft die Fahrstrasse mit dem Element und legt über den FahrstrassenElementtyp fest, welche Funktion das Element innerhalb dieser Fahrstrasse besitzt.

## ToolForm

Im FahrstrassenEdit-Modus zeigt das `ToolForm` eine Liste aller im aktuellen Projekt vorhandenen Fahrstrassen. Die Liste wird als Tabelle mit folgenden Spalten dargestellt:

| Spalte | Inhalt |
|---|---|
| ID | Eindeutige Kennung der Fahrstrasse |
| Bezeichnung | Bezeichnung der Fahrstrasse |
| Typ | Typ der Fahrstrasse `Rangier` oder `Zug` |

Es muss genau eine Fahrstrasse in der Liste ausgewählt sein, damit ihr Elemente zugewiesen oder entzogen werden können. Bei einem Wechsel der ausgewählten Fahrstrasse wird die Markierung der Elemente im `LupeForm` anhand der Zuordnungen der neu ausgewählten Fahrstrasse und des weiterhin ausgewählten FahrstrassenElementtyps neu aufgebaut.

Neben der Fahrstrassenliste enthält das `ToolForm` folgende Schaltflächen:

- **Neue Fahrstrasse:** Öffnet ein Eingabefenster, in dem die Daten der neuen Fahrstrasse erfasst werden. Wird das Eingabefenster mit `OK` geschlossen, wird in den Arbeitsdaten des aktuellen Projekts ein neuer Fahrstrassen-Datensatz angelegt und die Fahrstrassenliste entsprechend aktualisiert. Wird das Eingabefenster abgebrochen, werden keine Daten angelegt.
- **Fahrstrasse löschen:** Löscht die aktuell in der Liste selektierte Fahrstrasse. Vor dem Löschen muss eine Sicherheitsabfrage bestätigt werden. Nach der Bestätigung werden die Fahrstrasse und alle ihr zugeordneten FahrstrassenElemente aus den Arbeitsdaten entfernt. Die Fahrstrassenliste und die Markierungen im `LupeForm` werden anschließend aktualisiert. Wird die Sicherheitsabfrage abgebrochen, bleiben alle Daten unverändert.

Die Schaltfläche **Fahrstrasse löschen** ist nur verfügbar, wenn genau eine Fahrstrasse in der Liste selektiert ist. Das Löschen der zugehörigen FahrstrassenElemente erfolgt gemeinsam mit der Fahrstrasse, damit keine verwaisten Zuordnungen bestehen bleiben.

Zusätzlich enthält das `ToolForm` ein `SegmentControl` zur Auswahl des FahrstrassenElementtyps, der aktuell bearbeitet und angezeigt wird. Folgende acht FahrstrassenElementtypen stehen zur Verfügung:

| FahrstrassenElementtyp | Funktion innerhalb der Fahrstrasse |
|---|---|
| Fahrwegelement | Element, das im Sinne der Fahrstrecke direkt befahren wird |
| Fahrstrassenstart | Startpunkt der Fahrstrasse |
| Fahrstrassenziel | Zielpunkt der Fahrstrasse |
| Auflöseelement | Element zur Auflösung der Fahrstrasse |
| Unterwegselement | Unterwegs liegendes Element mit einer besonderen Funktion innerhalb der Fahrstrasse |
| Vorwegelement | Der Fahrstrasse zugeordnetes Vorwegelement |
| Flankenschutzelement | Element, das den Flankenschutz für die Fahrstrasse herstellt |
| Flankenschutztransportelement | Element im Flankenschutzraum, das frei sein muss |

Damit sind alle im Projekt vorgesehenen FahrstrassenElementtypen enthalten.

Im `LupeForm` werden ausschließlich diejenigen Elemente markiert, die der aktuell ausgewählten Fahrstrasse mit genau dem im `SegmentControl` ausgewählten FahrstrassenElementtyp zugeordnet sind. Die Filterung erfolgt anhand der ID des FahrstrassenElementtyps. Beim Wechsel des Segments werden zunächst die bisherigen fahrstrassenbezogenen Markierungen entfernt und anschließend die Zuordnungen des neu ausgewählten FahrstrassenElementtyps markiert. Existieren für die ausgewählte Kombination keine Zuordnungen, bleibt die fahrstrassenbezogene Markierung leer. Ist keine Fahrstrasse oder kein FahrstrassenElementtyp ausgewählt, wird keine fahrstrassenbezogene Elementmarkierung angezeigt.

Die zulässigen Kombinationen aus Elementtyp und FahrstrassenElementtyp sind verbindlich in [datenstrukturen.md](datenstrukturen.md#zulässige-zuordnungen-zu-elementtypen) festgelegt.

## Zuweisen eines FahrstrassenElements

Für eine Zuweisung müssen im `ToolForm`

1. eine Fahrstrasse in der Fahrstrassenliste und
2. ein FahrstrassenElementtyp im `SegmentControl`

ausgewählt sein.

Anschließend klickt der Benutzer im `LupeForm` auf das gewünschte Element. Ist dieses Element der ausgewählten Fahrstrasse noch nicht zugeordnet und ist die Kombination aus seinem Elementtyp und dem ausgewählten FahrstrassenElementtyp zulässig, wird die Zuordnung angelegt. Nicht zulässige Kombinationen dürfen nicht zugewiesen werden.

Dabei wird

- in den Arbeitsdaten der ausgewählten Fahrstrasse ein neues FahrstrassenElement angelegt,
- beim Speichern ein entsprechender Datensatz in der Tabelle `FahrstrassenElemente` erzeugt und
- das Element im `LupeForm` markiert, solange sein FahrstrassenElementtyp im `SegmentControl` ausgewählt ist.

Die Markierung wird als ausgefüllter Kreis dargestellt.

## Entziehen einer Zuweisung

Ist das angeklickte Element der ausgewählten Fahrstrasse bereits als FahrstrassenElement zugeordnet, wird die vorhandene Zuweisung entfernt. Die aktuell im `SegmentControl` ausgewählte Rolle ist für das Entfernen ohne Bedeutung. Das Element ist vor dem Entfernen nur dann fahrstrassenbezogen markiert, wenn seine vorhandene Zuordnung dem ausgewählten FahrstrassenElementtyp entspricht.

Dabei wird

- das FahrstrassenElement aus den Arbeitsdaten der ausgewählten Fahrstrasse entfernt,
- beim Speichern der zugehörige Datensatz aus der Tabelle `FahrstrassenElemente` gelöscht und
- die fahrstrassenbezogene Markierung im `LupeForm` anhand der verbleibenden Zuordnungen des ausgewählten FahrstrassenElementtyps aktualisiert.

Soll einem bereits zugeordneten Element ein anderer FahrstrassenElementtyp gegeben werden, wird die bestehende Zuweisung zunächst durch einen Klick entfernt. Danach wird im `SegmentControl` der neue FahrstrassenElementtyp ausgewählt und das Element erneut angeklickt.

Auf diese Weise kann der Benutzer der jeweils in der Liste ausgewählten Fahrstrasse nach und nach die zugehörigen Elemente der Topologie zuweisen oder vorhandene Zuordnungen wieder entfernen.

## Wechsel und Zurücksetzen des Modus

Beim Verlassen des FahrstrassenEdit-Modus müssen nicht abgeschlossene Bedienzustände zurückgesetzt werden. Beim Wechsel der ausgewählten Fahrstrasse bleibt der im `SegmentControl` ausgewählte FahrstrassenElementtyp erhalten. Die Darstellung im `LupeForm` muss stets der Kombination aus aktuell ausgewählter Fahrstrasse und aktuell ausgewähltem FahrstrassenElementtyp entsprechen. Markierungen der zuvor ausgewählten Fahrstrasse oder eines zuvor angezeigten FahrstrassenElementtyps dürfen nicht bestehen bleiben.
