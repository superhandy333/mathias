# Weiche – fachliche Zustände und Verschluss

Dieses Dokument beschreibt allgemeine fachliche Zustände und Regeln einer **Weiche** in `estw3`. Die hier beschriebenen Regeln gelten unabhängig davon, ob die Weiche im Zusammenhang mit einer Rangier- oder Zugstraße verwendet wird.

## Stränge

Eine Weiche hat 3 Stränge:

- Strang A: Die Spitze der Weiche
- Strang B: Der rechte Strang der Weiche. Von der Spitze der Weiche gesehen Fahrtrichtung rechts.
- Strang C: Der linke Strang der Weiche. Von der Spitze der Weiche gesehen Fahrtrichtung links.

## Zungenstellung

Die Zungenstellung einer Weiche beschreibt die aktuelle Lage der beweglichen Zunge, die den Fahrweg bestimmt. In der Simulation wird die Zungenstellung durch eine ganze Zahl im Bereich von −10 bis +10 einschließlich beider Grenzwerte dargestellt. Es gilt: -10 <= zungenstellung <= 10. 

Dabei gilt bildlich gesehen: Eine Person, die sich an der Weichenspitze befindet und in Richtung Herzstück der Weiche schaut, sieht die Stellung der Zungen:

- "links" bei Zungenstellung = -10, d.h. der Fahrweg ist nach "rechts" eingestellt (von Strang A nach Strang B).
- "rechts" bei Zungenstellung = 10, d.h. der Fahrweg ist nach "links" eingestellt (von Strang A nach Strang C).

## Stellung

Die Stellung einer Weiche beschreibt die aktuelle Lage der Weiche für die Auswertung durch die Stellwerkslogik, abhängig von der Zungenstellung.

Dabei gilt:

|Zungenstellung|Stellung|Bedeutung|
|-|-|-|
|-10|1|Ordnungsstellung/Endlage rechts|
|-9..9|0|keine Endlage|
|10|-1|Ordnungsstellung/Endlage links|
    

## Verschluss einer Weiche

Eine Weiche besitzt genau einen **Verschlusszähler**. Dieser Zähler wird gemeinsam für Verschlüsse durch Fahrstraßen und für Verschlüsse aufgrund von Flankenschutz verwendet. Einen separaten Flankenschutz-Verschlusszähler gibt es nicht.

Der Verschlusszähler ist eine nichtnegative ganze Zahl und hat mindestens den Wert `0`.

### Bedeutung des Verschlusszählers

Der Verschlusszähler hat folgende eindeutige Bedeutung:

- `0`: Die Weiche ist nicht verschlossen.
- `> 0`: Die Weiche ist verschlossen.

### Änderung des Verschlusszählers

Wenn eine Fahrstraße die Weiche als Fahrstraßenelement verschließt, wird der Verschlusszähler erhöht. Benötigt eine Fahrstraße die Weiche als Flankenschutz, wird derselbe Verschlusszähler ebenfalls erhöht.

Bei der jeweiligen Auflösung einer Fahrstraße oder der Aufhebung eines Flankenschutzes wird der Verschlusszähler entsprechend vermindert.

Der Verschluss einer Weiche wird wieder entfernt, wenn der Verschlusszähler den Wert `0` erreicht. 

Der Verschlusszähler darf dabei nicht kleiner als `0` werden.

### Gesamtregel

Durch den gemeinsamen Zähler kann eine Weiche gleichzeitig durch mehrere Fahrstraßen sowie durch mehrere Flankenschutzforderungen verschlossen sein. Das Entfernen eines einzelnen Verschlusses gibt die Weiche deshalb nur dann frei, wenn der Verschlusszähler anschließend den Wert `0` hat.

Jeder Zählerstand größer als `0` bedeutet, dass die Weiche verschlossen ist. Der Verschluss bezieht sich auf die aktuelle Lage der Weiche. Solange der Verschlusszähler größer als `0` ist, darf die Weiche nicht in eine andere Lage umgestellt werden.

## Umlaufmelder

Der Umlaufmelder einer Weiche ist der kleine rote Punkt im freien Bereich neben den Gleislinien. Er ist in `LupeWeiche` als `melder1` implementiert.

Der Umlaufmelder soll im sichtbaren Zustand entweder Ruhelicht zeigen oder schnell rot blinken. Das schnelle Blinken verwendet 250 ms an und 250 ms aus (2 Hz). Ein separater Anzeigetimer aktualisiert ausschließlich diese Melder alle 125 ms; der bisherige Simulations- und Aktualisierungstakt der übrigen Melder bleibt erhalten.

Sichtbar im Ruhelicht ist der Umlaufmelder, wenn die Weiche nicht in einer Endlage ist (stellung != -1 und stellung != 1) und der Motor nicht läuft (Motorlauf = None).

Sichtbar mit schnellem Blinken ist der Umlaufmelder, wenn die Weiche nicht in einer Endlage ist (stellung != -1 und stellung != 1) und der Motor läuft (Motorlauf != None).

Nicht sichtbar ist der Umlaufmelder, wenn sich die Weiche in einer Endlage befindet.

Maßgeblich ist die tatsächliche Weichenstellung aus dem Laufzeitmodell. Eine im Edit- oder FS-Edit-Modus abweichende Stellungsanzeige verändert den Umlaufmelder nicht.
