# Allgemeine Fahrstraßenlogik

Dieses Dokument beschreibt die für **Rangierstraßen** und **Zugstraßen** gemeinsam gültige Fahrstraßenlogik. Die jeweiligen Besonderheiten und konkreten Statusfolgen sind in [rangierstrasse.md](rangierstrasse.md) und [zugstrasse.md](zugstrasse.md) beschrieben.

Die fachliche Bedeutung der Signalstellungen wird durch die [Hauptsignalgeschwindigkeit](signale.md#hauptsignalgeschwindigkeit) festgelegt. Für Zug- und Rangierstrassen ist ausschließlich `HSV = 0` die **Haltstellung**. Eine Haltanforderung fordert diesen Wert an; eine Haltbestätigung setzt den tatsächlich erreichten Zustand `HSV == 0` voraus. `HSV = -3` bedeutet **Kennlicht** und ist kein Haltbegriff. Die Bedeutung der Fahrtstellungen hängt von der Fahrstraßenart ab.

## Zustand einer Fahrstraße

Der **Status** einer Fahrstraße ist ein ganzzahliger Ablaufzustand, der den aktuellen Schritt innerhalb der Fahrstraßenmechanik beschreibt.

- **Status = 0:** Die Fahrstraße ist **ausgeschaltet** und befindet sich in Grundstellung.
- **Status > 0:** Die Fahrstraße gilt als **eingeschaltet**. Dazu zählen auch die Zwischenzustände während des Einstell- und Sicherungsablaufs.

Die Begriffe **eingeschaltet** und **ausgeschaltet** werden damit aus dem ganzzahligen Status abgeleitet und stellen keinen eigenständigen booleschen Zustand dar.

## Einschalten und Zulassungsprüfung

Das Einschalten wird durch die Auswahl eines geeigneten Start- und Zielelements angestoßen. Als Startelement kommt ein **Signal** infrage; als Zielelement ein **Signal** oder **Blindziel**.

Sind Start- und Zielelement ausgewählt, wird die zugehörige Fahrstraße ermittelt und ihre **Zulassungsprüfung** durchgeführt. Nur bei positiver Zulassungsprüfung darf die Fahrstraße von **Status 0** in einen Status **größer 0** übergehen.

Die Reihenfolge der Auswahl ist ohne Bedeutung. Entscheidend ist, dass eines der beiden ausgewählten Elemente als Fahrstraßenstart und das andere als Fahrstraßenziel derselben Fahrstraße projektiert ist. Existiert keine passende Fahrstraße oder existieren mehrere Fahrstraßen mit derselben Start-/Zielkombination, wird keine Fahrstraße willkürlich ausgewählt. Die Bedienung und Markierungsregeln im Simulationsmodus sind in [lupeform.md](lupeform.md#fahrstraßenauswahl) beschrieben.

Die Zulassungsprüfung ist eine **einmalige Prüfung ganz am Anfang des Einschaltvorgangs**. Sie wird nach dem Übergang aus Status 0 **nicht zyklisch wiederholt**. Der Grund dafür ist, dass im weiteren Ablauf, insbesondere durch das Setzen der Beanspruchungen in **Status 1**, Bedingungen verändert werden, die Bestandteil der ursprünglichen Zulassungsprüfung sind. Eine erneute Zulassungsprüfung würde deshalb nicht mehr den Zustand vor dem Einschalten der Fahrstraße prüfen.

Die allgemein gültige Beschreibung der Zulassungsprüfung sowie die Verweise auf die typabhängigen Bedingungen befinden sich in [estw.md](estw.md). Die speziellen Bedingungen für Rangier- und Zugstraßen sind in [rangierstrasse.md](rangierstrasse.md) beziehungsweise [zugstrasse.md](zugstrasse.md) beschrieben.

### Diagnose bei Abweisung

Eine abgewiesene Anforderung liefert einen Ablehnungsgrund. Ist die Ursache einem vorhandenen Element zuzuordnen, enthält die Diagnose dessen **Name und Element-ID** sowie die verletzte Bedingung. Bei mehreren Ursachen wird ausschließlich die **erste** Verletzung in der bestehenden Prüfreihenfolge gemeldet; innerhalb der Elementprüfung gilt die Reihenfolge der projektierten Fahrstrassenelemente. Die Prüfung endet unmittelbar mit diesem Ergebnis. Die Diagnose entsteht während der ursprünglichen Struktur- beziehungsweise Zulassungsprüfung und wird über `Fs::requestActivationWithResult()` und `Estw::requestFahrstrasseWithResult()` an die Commandline weitergereicht. Eine zusätzliche Zulassungsprüfung zur Ermittlung der Ursache findet nicht statt.

Die bestehenden booleschen Anforderungsfunktionen verwenden denselben Prüfpfad. Ablehnungen verändern keine Beanspruchungen, Verschlüsse, Signalstellungen oder Melder. Eine angenommene Anforderung enthält keine Ablehnungsdiagnose. Name, ID und Grund sind im zurückgegebenen Ergebnis als Werte festgehalten.

Allgemeine Ursachen wie eine fehlende oder mehrdeutige Fahrstrassenauswahl, eine bereits aktive Fahrstrasse oder eine unvollständige Pflichtrollen-Projektierung werden ohne Elementzuordnung gemeldet. Bei einer nicht auflösbaren Elementreferenz wird, soweit verfügbar, die projektierte Element-ID mit dem Ersatztext `Element` statt eines unbekannten Namens ausgegeben. Die Zulassungsbedingungen und die spätere Zustandsmechanik werden durch diese Diagnose nicht verändert.

## Zyklische Prüfung der Statusvoraussetzungen

Nach dem erfolgreichen Beginn des Einschaltvorgangs werden die Voraussetzungen für das **Erreichen beziehungsweise Beibehalten eines Status größer 1 zyklisch überprüft**.

Bei jeder zyklischen Überprüfung ist für die beteiligten Fahrstraßenelemente festzustellen, ob die für den jeweiligen Status erforderlichen Voraussetzungen weiterhin erfüllt sind. Die konkreten Voraussetzungen ergeben sich aus der Statusbeschreibung der jeweiligen Fahrstraßenart.

### Ausnahmen von der zyklischen Ausführung

Von der zyklischen Prüfung beziehungsweise Ausführung sind insbesondere folgende Vorgänge ausgenommen:

1. **Zulassungsprüfung:** Sie wird nur **einmal ganz am Anfang** durchgeführt und danach nicht erneut ausgeführt.
2. **Fahrtstellung des Startsignals:** Die Fahrtstellung des Startsignals wird einmalig beim Erreichen von **Status 4 bei Rangierstrassen** beziehungsweise **Status 6 bei Zugstrassen** angefordert. Diese Aktion wird bei späteren zyklischen Prüfungen innerhalb desselben Status nicht erneut ausgelöst. Die in [rangierstrasse.md](rangierstrasse.md) beschriebene erneute Fahrtanforderung beim Wiedererreichen von Status 4 nach einem Rückfall auf Status 3 bleibt davon unberührt. Für Zugstrassen gelten die Regeln zur einmaligen Fahrtanforderung und Wiederholungssperre aus [zugstrasse.md](zugstrasse.md).

## Reaktion auf weggefallene Voraussetzungen

Wird bei der zyklischen Überprüfung festgestellt, dass bei mindestens einem der beteiligten **Fahrstraßenelemente** die für den aktuellen Status erforderlichen Voraussetzungen nicht mehr erfüllt sind, muss die sichernde Signalstellung wiederhergestellt werden.

Dazu gilt:

- Das **Startsignal** ist in **Haltstellung** zu bringen.
- Sind **Unterwegssignale** vorhanden, sind auch diese in **Haltstellung** zu bringen.

Diese Reaktion erfolgt unabhängig davon, welcher der zyklisch überwachten Voraussetzungen weggefallen ist. Die weitere Behandlung des Fahrstraßenstatus richtet sich nach der für Rangier- beziehungsweise Zugstraßen beschriebenen Zustandslogik.

## Typabhängige Fahrstraßenlogik

Die gemeinsamen Regeln dieses Dokuments werden durch die typabhängigen Beschreibungen ergänzt:

- [rangierstrasse.md](rangierstrasse.md) – Ablauf, Statuswerte und Besonderheiten der Rangierstraße.
- [zugstrasse.md](zugstrasse.md) – Ablauf, Statuswerte und Besonderheiten der Zugstraße.

Spezielle Regeln dieser Dokumente haben Vorrang, soweit sie für den jeweiligen Fahrstraßentyp eine von der allgemeinen Beschreibung abweichende oder weitergehende Festlegung enthalten.
