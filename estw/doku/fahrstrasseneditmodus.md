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

Es muss genau eine Fahrstrasse in der Liste ausgewählt sein, damit ihre FahrstrassenElemente bearbeitet werden können. Die ausgewählte Fahrstrasse ist die allein maßgebliche Fahrstrasse; alle Zuordnungen werden über die stabilen IDs der Fahrstrasse, der Elemente und der FahrstrassenElemente ermittelt. Bei einem Wechsel der ausgewählten Fahrstrasse werden eine eventuell vorhandene FahrstrassenElement-Selektion und die Typauswahl zurückgesetzt und die Markierungen im `LupeForm` anhand der Zuordnungen der neu ausgewählten Fahrstrasse neu aufgebaut.

Neben der Fahrstrassenliste enthält das `ToolForm` folgende Schaltflächen:

- **Neue Fahrstrasse:** Öffnet ein Eingabefenster, in dem die Daten der neuen Fahrstrasse erfasst werden. Wird das Eingabefenster mit `OK` geschlossen, wird in den Arbeitsdaten des aktuellen Projekts ein neuer Fahrstrassen-Datensatz angelegt und die Fahrstrassenliste entsprechend aktualisiert. Wird das Eingabefenster abgebrochen, werden keine Daten angelegt.
- **Fahrstrasse löschen:** Löscht die aktuell in der Liste selektierte Fahrstrasse. Vor dem Löschen muss eine Sicherheitsabfrage bestätigt werden. Nach der Bestätigung werden die Fahrstrasse und alle ihr zugeordneten FahrstrassenElemente aus den Arbeitsdaten entfernt. Die Fahrstrassenliste und die Markierungen im `LupeForm` werden anschließend aktualisiert. Wird die Sicherheitsabfrage abgebrochen, bleiben alle Daten unverändert.

Die Schaltfläche **Fahrstrasse löschen** ist nur verfügbar, wenn genau eine Fahrstrasse in der Liste selektiert ist. Das Löschen der zugehörigen FahrstrassenElemente erfolgt gemeinsam mit der Fahrstrasse, damit keine verwaisten Zuordnungen bestehen bleiben.

Am unteren Rand enthält das `ToolForm` im FahrstrassenEdit-Modus nebeneinander die Schaltflächen **Speichern**, **Verwerfen** und **Prüfung**. Die Schaltfläche **Prüfung** ist ausschließlich in diesem Modus vorhanden.

Unterhalb der Fahrstrassenliste enthält das `ToolForm` ein `SegmentControl` zur Auswahl der Bearbeitungsart mit den Segmenten **Neu**, **Edit** und **Löschen**. Darunter befindet sich das `SegmentControl` zur Auswahl des FahrstrassenElementtyps. Beide SegmentControls einschließlich ihrer Beschriftungen sind nur sichtbar und bedienbar, wenn genau eine Fahrstrasse ausgewählt ist.

Folgende acht FahrstrassenElementtypen stehen zur Verfügung:

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

Im `LupeForm` bleiben alle zur aktuell ausgewählten Fahrstrasse gehörenden FahrstrassenElemente als nicht ausgefüllte Kreise sichtbar. Diejenigen Zuordnungen, deren Typ der aktuellen Auswahl im Typ-SegmentControl entspricht, werden als ausgefüllte Kreise dargestellt. Die Filterung erfolgt anhand der stabilen ID des FahrstrassenElementtyps. Ist keine Fahrstrasse ausgewählt, wird keine fahrstrassenbezogene Elementmarkierung angezeigt.

Die zulässigen Kombinationen aus Elementtyp und FahrstrassenElementtyp sind verbindlich in [datenstrukturen.md](datenstrukturen.md#zulässige-zuordnungen-zu-elementtypen) festgelegt.

## Bearbeitungsart „Neu“

Für eine Zuweisung müssen im `ToolForm`

1. eine Fahrstrasse in der Fahrstrassenliste und
2. ein FahrstrassenElementtyp im `SegmentControl`

ausgewählt sein.

Anschließend klickt der Benutzer im `LupeForm` auf das gewünschte Element. Ist dieses Element der ausgewählten Fahrstrasse noch nicht zugeordnet und ist die Kombination aus seinem Elementtyp und dem ausgewählten FahrstrassenElementtyp zulässig, wird die Zuordnung mit der im Eingabefeld angegebenen Sollstellung angelegt. Das Eingabefeld für die Sollstellung ist nur in dieser Bearbeitungsart sichtbar. Nicht zulässige Kombinationen dürfen nicht zugewiesen werden. Ein Klick auf ein leeres Feld oder ein bereits zugeordnetes Element hat keine Wirkung. Ohne ausgewählten FahrstrassenElementtyp wird keine Zuordnung angelegt und der Benutzer zur Typauswahl aufgefordert.

Dabei wird

- in den Arbeitsdaten der ausgewählten Fahrstrasse ein neues FahrstrassenElement angelegt,
- beim Speichern ein entsprechender Datensatz in der Tabelle `FahrstrassenElemente` erzeugt und
- das Element im `LupeForm` markiert, solange sein FahrstrassenElementtyp im `SegmentControl` ausgewählt ist.

Die Markierung wird als ausgefüllter Kreis dargestellt.

## Bearbeitungsart „Edit“

Ein Klick auf ein bereits zugeordnetes Element selektiert genau dessen FahrstrassenElement-Datensatz anhand seiner ID. Ein Klick auf ein nicht zugeordnetes Element oder ein leeres Feld hat keine Wirkung. Nach der Selektion wird das Typ-SegmentControl auf den Typ des FahrstrassenElements eingestellt. Eine anschließende Typauswahl ändert den Typ dieses Datensatzes unmittelbar in den Arbeitsdaten.

Für das selektierte FahrstrassenElement wird ein PropertyGrid angezeigt. Es enthält mindestens:

| Eigenschaft | Verhalten |
|---|---|
| Name des Elements | wird angezeigt und kann im Grid nicht geändert werden |
| FahrstrassenElementtyp | wird angezeigt; Änderungen erfolgen ausschließlich über das Typ-SegmentControl |
| Sollstellung | wird angezeigt und kann im PropertyGrid geändert werden |

Eine geänderte Sollstellung wird sofort validiert und in die Arbeitsdaten übernommen. Danach wird die zentrale Aktualisierung der `LupeForm` ausgelöst, sodass insbesondere eine Weiche unmittelbar in ihrer neuen Sollstellung dargestellt wird. Ohne selektiertes FahrstrassenElement ist das PropertyGrid leer beziehungsweise ausgeblendet.

## Bearbeitungsart „Löschen“

Ein Klick auf ein bereits zur ausgewählten Fahrstrasse gehörendes FahrstrassenElement entfernt ausschließlich diesen Zuordnungsdatensatz aus den Arbeitsdaten. Das zugrunde liegende Element und die Fahrstrasse bleiben erhalten. Ein Klick auf ein nicht zugeordnetes Element oder ein leeres Feld hat keine Wirkung. Anschließend werden Selektion, PropertyGrid, Typauswahl und Markierungen konsistent aktualisiert.

## Wechsel und Zurücksetzen des Modus

Beim Verlassen des FahrstrassenEdit-Modus müssen nicht abgeschlossene Bedienzustände zurückgesetzt werden. Beim Wechsel der ausgewählten Fahrstrasse, der Bearbeitungsart oder beim Entfernen des selektierten FahrstrassenElements wird dessen Selektion zurückgesetzt und das PropertyGrid geleert beziehungsweise ausgeblendet. Die Darstellung im `LupeForm` muss stets der aktuell ausgewählten Fahrstrasse und der aktuellen Typauswahl entsprechen. Markierungen der zuvor ausgewählten Fahrstrasse oder eines zuvor angezeigten FahrstrassenElementtyps dürfen nicht bestehen bleiben.

## Speichern und fachliche Validierung

Beim Speichern wird der vollständige Arbeitsbestand weiterhin anhand aller vorhandenen fachlichen Gültigkeitsregeln für Fahrstrassen geprüft. Festgestellte Unvollständigkeiten oder fachliche Fehler werden dem Benutzer als Hinweise beziehungsweise Warnungen angezeigt. Ein negatives fachliches Prüfergebnis verhindert die Speicherung jedoch nicht: Alle bearbeiteten Fahrstrassen und ihre Fahrstrassenelemente werden im aktuell erreichten Arbeitsstand gespeichert.

Die Schaltfläche **Prüfung** führt dieselbe fachliche Prüfung für alle Fahrstrassen des aktuell bearbeiteten Projekts aus. Maßgeblich sind die aktuellen Arbeitsdaten einschließlich aller noch nicht gespeicherten Änderungen. Festgestellte Meldungen werden angezeigt; bei fehlerfreiem Ergebnis wird der Erfolg bestätigt. Die Prüfung führt keine Persistenzfunktion aus, verändert weder Arbeitsdaten noch Auswahl oder Markierungen und lässt den FahrstrassenEdit-Modus geöffnet.

Technische Fehler bleiben davon unberührt. Kann die Datenquelle die Änderungen nicht vollständig übernehmen, tritt ein Bearbeitungskonflikt auf oder sind interne Zuordnungsdaten nicht sicher speicherbar, wird der Vorgang weiterhin als Fehler behandelt. In diesem Fall bleibt der Editor mit seinen Arbeitsdaten geöffnet. Die für mrdb vorgesehene Rücksicherung bei einem technischen Schreibfehler bleibt erhalten.
