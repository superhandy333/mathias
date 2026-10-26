Die Grundfolge **Beanspruchen → Fahrwegweichen stellen und verschließen → Flankenschutz herstellen → Signale freigeben** ist plausibel. Als eindeutig implementierbare Zustandsmaschine ist der Abschnitt aber noch nicht vollständig; insbesondere besteht ein Widerspruch bei den FÜM-Voraussetzungen.

| Status | Einschätzung |
|---|---|
| 0 | Plausibel. Die Zulassungsprüfung muss einmalig bleiben. |
| 1 | Beanspruchungen weitgehend nachvollziehbar; eine Formulierung widerspricht der Rollenabgrenzung. |
| 2 | Plausibel: erst bei tatsächlich vorhandenen Beanspruchungen stellen. |
| 3 | Plausibel: erst bei tatsächlich erreichter Sollstellung verschließen. Einmaligkeit des Verschlussbeitrags fehlt. |
| 4 | Als Zwischenstufe vor dem Flankenschutz nachvollziehbar. |
| 5 | Nachvollziehbar, aber tatsächliches Erreichen der Sollstellung und Signalflankenschutz müssen genauer beschrieben werden. |
| 6 | Widersprüchliche FÜM-Voraussetzung; Bedeutung des Dauerlichts noch klärungsbedürftig. |
| 7 | Freiprüfung und Vorsignalgeschwindigkeit sind unvollständig beschrieben. |

Folgende Punkte würde ich bearbeiten:

1. **Status 6 und 7 verlangen mittelbar gleichzeitig blinkenden und dauerleuchtenden FÜM.**

   Status 5 fordert einen blinkenden FÜM. Status 6 übernimmt sämtliche Voraussetzungen aus Status 5 und schaltet anschließend auf Dauerlicht. Damit verletzt seine eigene Aktion eine weiterhin geforderte Voraussetzung. Status 7 übernimmt diesen Widerspruch.

   **Empfehlung:** Einmalige Eintrittsbedingungen von dauerhaft zu überwachenden Sicherungsbedingungen unterscheiden. Das vorherige FÜM-Anzeigebild darf nicht als dauerhafte Voraussetzung in höhere Status übernommen werden. [Status 5–7](D:/csharp/CPP/estw/estw3/.agents/zugstrasse.md:122)

2. **Die Aktion zum Herstellen des Signalflankenschutzes fehlt.**

   Status 6 verlangt Flankenschutzsignale in Haltstellung. Vorher fordert aber kein Status diese Haltstellung an. Steht eines dieser Signale auf Fahrt, kann die Zustandsfolge deshalb dauerhaft warten.

   Zusätzlich ist noch offen, wie diese Signale während der aktiven Zugstraße in Halt gebunden werden und wie konkurrierende Fahrstraßen berücksichtigt werden. Eine momentane Haltstellung allein beschreibt diese Bindung nicht. Auch die Behandlung gegebenenfalls benötigter Flankenbeanspruchungen bleibt offen. [Flankenschutzprüfung](D:/csharp/CPP/estw/estw3/.agents/zugstrasse.md:131)

3. **Verschließen muss nach tatsächlicher Sollstellung und mit genau einem eigenen Beitrag erfolgen.**

   Bei Status 5 sollte „sofort verschlossen“ ausdrücklich bedeuten: **erst nachdem die jeweilige Weiche ihre Sollstellung tatsächlich erreicht hat**. Die bloße Stellanforderung genügt nicht. Jede Weiche kann dabei einzeln nach Erreichen ihrer Sollstellung verschlossen werden.

   Für Status 3 und 5 fehlt außerdem die Festlegung, dass der eigene Verschlussbeitrag nur einmal aufgebaut wird. Zyklische Ausführung oder Wiedererreichen eines Status dürfen den gemeinsamen Verschlusszähler nicht erneut erhöhen, solange der ursprüngliche Beitrag besteht. [Gemeinsamer Verschlusszähler](D:/csharp/CPP/estw/estw3/.agents/weiche.md:18)

4. **Status 1 enthält wieder eine unklare Rollenabgrenzung.**

   Der Einleitungstext schließt Flankenschutz- und Transportelemente von diesen Beanspruchungsaktionen aus. Der erste Aktionspunkt nennt anschließend jedoch pauschal „alle Fahrwegelemente, Blindziele und Auflöseelemente“.

   Ein Blindziel kann auch Transportelement sein. Der Aktionspunkt sollte deshalb ausdrücklich auf die zuvor genannten Rollen begrenzt werden. Für Gleise, Blindziele und Auflöseelemente als Fahrwegelement sollte außerdem der durchgehend beanspruchte Bereich A–B wieder ausdrücklich genannt werden. [Status 1](D:/csharp/CPP/estw/estw3/.agents/zugstrasse.md:82)

5. **Die Freiprüfung in Status 7 vermischt Rollen und Elementtypen.**

   „Blindelement“ ist im dokumentierten Modell keine definierte Rolle. „Blindziel“ bezeichnet einen Elementtyp; seine Fahrstraßenrolle kann beispielsweise Ziel oder Fahrweg sein.

   Außerdem fehlen **Auflöseelemente** und ein **Signal als Fahrstrassenziel** ausdrücklich in der Aufzählung. Hier muss festgelegt werden, welche beanspruchten Bereiche für die Fahrtfreigabe frei sein müssen.

   Beim Vorwegsignal ist zusätzlich zu klären: Muss das gesamte Signal frei sein oder nur sein beanspruchter Bereich B? Das vorhandene Laufzeitmodell besitzt lediglich einen gemeinsamen Besetztzustand je Element und kann diese beiden Fälle derzeit nicht unterscheiden. [Freiprüfung](D:/csharp/CPP/estw/estw3/.agents/zugstrasse.md:148), [Besetztzustand](D:/csharp/CPP/estw/estw3/estw/element.h:54)

6. **Die Vorsignalgeschwindigkeit in Status 7 ist noch nicht vollständig definiert.**

   Es fehlen folgende Fälle:

   - Ohne Vorwegelement ist laut `signale.md` das Zielelement heranzuziehen.
   - Bei einem Blindziel ohne Vorwegelement fehlt eine Hauptsignalgeschwindigkeit als Quelle; dieser Fall benötigt eine Festlegung.
   - Negative Hauptsignalgeschwindigkeiten dürfen nicht unverändert als Vorsignalgeschwindigkeit übernommen werden: Deren Wertebereich beginnt bei 0. Besonders die Behandlung eines nicht relevanten Signals muss geklärt werden.
   - Für Unterwegssignale wird zwar die Hauptsignalstellung gesetzt, ihre Vorsignalgeschwindigkeit bleibt unbeschrieben.

   Die Vorsignalgeschwindigkeit sollte außerdem bereits beim erstmaligen Freigeben stimmen und anschließend zyklisch aktualisiert werden. [Status 7](D:/csharp/CPP/estw/estw3/.agents/zugstrasse.md:150), [Signalvorgaben](D:/csharp/CPP/estw/estw3/.agents/signale.md:61)

7. **Rückfälle, Wiederanlauf und ZIF fehlen.**

   Die allgemeine Fahrstraßenlogik fordert bei weggefallenen Voraussetzungen Halt für Start- und Unterwegssignale. Im Zugstraßenmodell fehlt aber:

   - Auf welchen Status fällt die Zugstraße zurück?
   - Welches FÜM-Anzeigebild folgt daraus?
   - Welche Beanspruchungen und Verschlüsse bleiben erhalten?
   - Darf bei wiederhergestellten Voraussetzungen erneut Fahrt angefordert werden?
   - Wann wird der Ziel-ZIF gesetzt und zurückgenommen?

   Gerade die neue Freiprüfung ist dabei relevant: Eine Belegung nach Fahrtfreigabe kann durch die vorgesehene Zugfahrt entstehen. Die daraus folgende Reaktion muss ausdrücklich beschrieben werden. [Allgemeine Rückfallregel](D:/csharp/CPP/estw/estw3/.agents/fahrstrasse.md:41)

8. **Die Bedeutung von Status 6 und 7 muss abschließend festgelegt werden.**

   Nach jetzigem Text kann der FÜM bereits dauerhaft leuchten, obwohl der Fahrweg noch besetzt ist: Die Fahrwegfreiprüfung erfolgt erst vor Status 7. Das ist innerhalb dieses Modells möglich, wenn Dauerlicht bewusst nur das bis Status 6 erreichte Sicherungsniveau bezeichnet. Soll es zusätzlich den freien Fahrweg bestätigen, müsste die Prüfung früher erfolgen.

   Ebenso bleibt offen, ob Status 7 lediglich die Fahrtstellung **anfordert** oder die tatsächlich erreichte Fahrtstellung bestätigt. Eine positive Sollstellung am Startsignal sollte ausdrücklich vorausgesetzt werden.

Außerhalb des untersuchten Abschnitts bestehen noch Anschlusswidersprüche: Das Zustandsdiagramm beschreibt weiterhin die alte Folge bis Status 4; die Liste offener Punkte erklärt bereits nummerierte Flankenschutzschritte noch für unbestimmt. Außerdem nennt `fahrstrasse.md` weiterhin Status 4 als Zeitpunkt der Fahrtstellung. [Altes Diagramm](D:/csharp/CPP/estw/estw3/.agents/zugstrasse.md:172), [Allgemeine Statusregel](D:/csharp/CPP/estw/estw3/.agents/fahrstrasse.md:39)

Ich habe keine Dateien verändert.