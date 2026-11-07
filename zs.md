# Machbarkeitsanalyse zur Implementierung der Zugstraßenmechanik

Stand: 7. November 2026. Untersucht wurde der lokale Arbeitsstand von `estw3` auf Basis von Git-Commit `213d3519`, nicht lediglich der historische Bericht.

## 1. Ziel und Grundlage der Analyse

Der Bericht bewertet, ob die Zugstraßenmechanik mit den vorhandenen Projektierungsdaten, Laufzeitobjekten und Bedienabläufen implementiert werden kann. Er unterscheidet vorhandene Funktionen, fehlende Implementierung und offene fachliche Festlegungen. Empfehlungen in diesem Bericht sind Vorschläge für einen späteren Auftrag und keine Ergänzungen der verbindlichen Spezifikation.

Die Quellenpriorität lautet: aktuelle [zugstrasse.md](zugstrasse.md), aktueller C++-Code, anschließend [zugstrasse_bericht.md](zugstrasse_bericht.md) als historische Einschätzung. Die Zugstraßenspezifikation und der historische Bericht wurden jeweils vollständig gelesen und miteinander abgeglichen. Weitere Quellen sind [fahrstrasse.md](fahrstrasse.md), [rangierstrasse.md](rangierstrasse.md), [signale.md](signale.md), [weiche.md](weiche.md), [datenstrukturen.md](datenstrukturen.md), [rules_isvalid_estwdata.md](rules_isvalid_estwdata.md), [estw.md](estw.md), [lupeform.md](lupeform.md), die übrigen im Projekt-AGENTS.md genannten Vorgaben sowie das [Rangierstraßen-Implementierungsprotokoll](../RANGIERSTRASSEN_IMPLEMENTIERUNG.md).

Vollständig untersucht wurden die Laufzeitklassen unter `estw/`, insbesondere `Fs`, `Estw`, `Signal`, `Weiche`, `Gleis`, `Blind` und `Aufloese`. Ergänzend wurden das Stammdatenmodell, die einschlägigen Validierungs- und Editorfunktionen, die Erzeugungs-, Bedienungs- und Aktualisierungspfade in `LupeForm`/`ToolForm`, die betroffenen Lupe-Darstellungen sowie die vorhandenen Fahrstraßen- und Auflösemeldertests untersucht. Fundstellen nennen Datei, Funktion beziehungsweise Klasse und bei zentralen Aussagen die Zeilennummer des untersuchten Standes. Zeilennummern können sich später verschieben.

Es handelt sich um eine statische Analyse. Es wurden weder ein Build noch Tests oder eine manuelle Simulation ausgeführt. Vorhandene Tests werden als überprüfbare Testgrundlage beschrieben, nicht als in diesem Auftrag erfolgreich ausgeführte Prüfung. Dieser Auftrag erstellt ausschließlich diesen Bericht.

## 2. Zusammenfassung der Einschätzung

**Die vollständige Zugstraßenmechanik kann auf Grundlage der aktuellen Dokumentation noch nicht ohne fachliche Vorentscheidungen begonnen werden.** Technische Grundlagen sind ausreichend vorhanden, um abgegrenzte Vorarbeiten sofort zu beginnen. Eine durchgängig spezifikationsgerechte Implementierung von Zulassung bis Auflösung wäre derzeit jedoch nur mit eigenmächtigen Fachentscheidungen möglich.

Die wesentlichen Blocker sind:

1. **Unvollständige Zustandsfolge:** `zugstrasse.md` beschreibt Status 0 bis 4 sowie 6 und 7. Status 5 fehlt vollständig, wird aber von Status 6 vorausgesetzt. Das Diagramm beschreibt weiterhin Fahrtstellung bei Status 4; der Fließtext verlangt dafür Status 7.
2. **Unvollständiger Signalflankenschutz:** Halt wird als Zustand verlangt, aber Herstellen, Haltebindung, gemeinsame Nutzung und Konfliktbehandlung sind nicht beschrieben. Das vorhandene Bool-Flag `BEAF` erzwingt diese Bindung nicht umfassend.
3. **Offene Auflösung:** Die Spezifikation erklärt die Bedingungen und Reihenfolge zur Aufhebung des Flankenschutzes ausdrücklich für noch festzulegen. Unterwegssignale, FÜM, ZIF und gemeinsam genutzte Schutzbeiträge brauchen ebenfalls einen vollständigen Lebenszyklus.
4. **Offene Rückfälle und Wiederanläufe:** Allgemein ist Halt bei Überwachungsverlust vorgegeben; Zielstatus, FÜM/ZIF und Wiederfreigabe für Zugstraßen fehlen. Dies betrifft besonders die regulär durch eine Zugfahrt entstehende Belegung.
5. **Widersprüchliche Zielzulassung:** Die Zugstraßen-ZP verlangt ausdrücklich `PRZ`, während das Datenmodell `PZZ` als Zugzielzulassung führt und `Blind::setZIF(Zug)` genau dieses Flag voraussetzt. Die höhere Quellenpriorität rechtfertigt keine stillschweigende Korrektur von `PRZ` zu `PZZ`.
6. **Unvollständige Signalsteuerung:** Zusammenspiel der Unterwegssignal-Aktionen in Status 4 und 6, tatsächliche Stellungsbestätigung, VSV-Sonderfälle und vorhandene Signalbedingungen einschließlich Wiederholungssperre sind nicht abschließend vereinbart.

Bereits nutzbar sind die acht Projektierungsrollen, ID-basierte Start-/Zielsuche, Signal-/Blindziel-Wrapper, Strangbeanspruchungen, gemeinsamer Weichenverschlusszähler, gezielte Weichenanforderung mit Istprüfung, zyklischer Simulationsaufruf, Auswahllogik, Auflöseklick und große Teile der Darstellung. Es ist weder ein neues Datenbankschema für die bereits beschriebenen Rollen noch ein zweiter Verschlusszähler erforderlich. Eine neue topologische Fahrwegsuche ist für die Auswahl einer bereits projektierten Fahrstraße nicht erforderlich.

**Abgrenzung für einen Arbeitsbeginn:** Strukturprüfung, Zugriff auf vorhandene Programmfälle, gemeinsame technische Hilfen und Testdaten lassen sich vorbereiten. Das bloße Entfernen der Zugstraßensperre aus `Fs::requestActivation()` wäre keine tragfähige Implementierung und würde Rangierlogik unter dem falschen Fahrstraßentyp ausführen.

## 3. Aktueller Stand im Code

### 3.1 Stammdaten, Projektierung und Validierung

| Bestandteil | Tatsächlicher Stand und Fundstelle | Bewertung |
|---|---|---|
| Fahrstraßentyp und Rollen | [structs.h](../structs.h): `FahrstrassenTypId` mit Rangier/Zug sowie `FahrstrassenElementRole` mit Fahrweg=1, Start=2, Ziel=3, Aufloese=4, Flanke=5, Vorweg=6, Unterwegs=7, Transport=8 | IDs sind im aktuellen Code ausdrücklich definiert; keine bloße Annahme. |
| Zuordnung und Sollstellung | `FahrstrassenElement` enthält `FahrstrasseId`, `ElementId`, `TypId`, `SollStellung`; `FullProjekt` enthält Elemente und Fahrstraßen | Die für die beschriebenen Rollen benötigten Referenzen und Sollwerte sind vorhanden. |
| Programmfälle | `Element` enthält `PR`, `PZ`, `PRS`, `PRZ`, `PZS`, `PZZ`, `PSSL`, `PSSR`, `PZSP` | Keine zusätzlichen Bool-Felder für diese Anforderungen nötig. |
| Strukturelle Prüfung | [validate.cpp](../validate.cpp), `isvalid_estwdata()`, insbesondere Zeilen 395–489 und 493–621 | Referenzen, Projektdifferenzen, doppelte Elementzuordnung, erlaubte Element-/Rollenkombinationen, genau ein Start/Ziel/Auflöser und höchstens ein Vorweg sind geprüft. |
| Sollwertprüfung | `validate.cpp:467`: Gleis/Blind/Auflöser nur 0, Weiche -1/+1, Signal -3 bis 16; Rolle Unterwegs ausschließlich -1/-2/-3 | Allgemeine Datenvalidität ist vorhanden. Positive Sollstellung eines Zugstartsignals wird noch nicht gesondert verlangt. |
| Editor | [fahrstrasseneditor.cpp](../fahrstrasseneditor.cpp), `validateMemberRole()`, `validateSollStellung()`, `addElement()`, `updateSelectedSollStellung()` | Alle acht Rollen sind projektierbar. Die lokale Sollprüfung ist elementtypbezogen und kennt die strengere Unterwegsrollen-Prüfung des Gesamtvalidators nicht. |
| Laden und Speichern | [aktionen.cpp](../aktionen.cpp), `generate_fullprojekte()` ab Zeile 666 und `generate_estw_data()`; [fahrstrassenpersistence.cpp](../fahrstrassenpersistence.cpp), `replaceFahrstrassenTables()` und `validateFahrstrassenChanges()` | Projektionen und Sollstellungen erreichen die Projektkopien. Speichern fachlich unvollständiger Fahrstraßen ist bewusst möglich. |

Das Speichern ist kein Nachweis der Laufzeitverwendbarkeit: `validateFahrstrassenChanges()` setzt bei fachlichen Fehlern eine Warnmeldung und lässt technisch gültige Speicherung zu. `persistFahrstrassenChanges()` in `aktionen.cpp:1406` übernimmt diese Trennung. Auch `generate_estw_data()` ruft die Validierung auf und gibt den Datenbestand zurück. Daher muss die Aktivierungsseite eine unvollständige oder ungültige Zugprojektion zuverlässig abweisen; auf eine vermeintlich immer fehlerfreie Persistenz darf sie sich nicht verlassen.

### 3.2 Laufzeitaufbau und Auswahl

[estw/estw.cpp](../estw/estw.cpp), `Estw::createFahrstrasse()`, löst alle acht Rollen anhand der Element-ID auf und befüllt `FsInitData`. [estw/fs.h](../estw/fs.h) enthält dafür Einzelreferenzen für Start, Ziel, Auflöser und Vorweg sowie Listen für Fahrweg, Unterwegs, Flanke und Transport. `Fs::Member` kombiniert die jeweilige Projektierungszeile einschließlich Sollstellung mit dem Laufzeitelement. Die Sollstellung geht durch die rollenbezogenen Elementlisten somit nicht verloren.

`ZielElement` kapselt Signal oder Blindziel und stellt `getElement()`, `getSignal()`, `getPRZ()`, `getZIF()` und `setZIF()` bereit. Der Setter reicht den übergebenen ZIF inzwischen unverändert weiter (`fs.cpp:348`). Ein gemeinsamer `getPZZ()`-Zugriff ist nicht vorhanden; die konkreten Signal-/Blindklassen besitzen die Einzelzugriffe bereits.

`Fs::matchesSelection()` (`fs.cpp:96`) sucht Start und Ziel unabhängig von der Auswahlreihenfolge. `Estw::requestFahrstrasse()` weist keinen Treffer und mehrere Treffer zurück. Die Suche umfasst beide Fahrstraßentypen. Eine Rangier- und eine Zugstraße mit derselben Start-/Zielkombination werden deshalb gemeinsam als mehrdeutig abgewiesen; das entspricht zunächst der allgemeinen Auswahlregel und darf nicht heimlich durch Typpriorisierung verändert werden.

`Fs::isValid()` ist eine **teilweise** Laufzeit-Strukturprüfung: Aufgelöste Referenzen, eindeutige Element-IDs, passende `FahrstrasseId`, Pflichtrollen, einige Fahrwegtypen und Weichen-Sollstellungen werden geprüft. Blind/Auflöser als Fahrweg werden über ihre Ableitung von `Gleis` mit akzeptiert. Unbekannte Rollen werden bereits mittelbar abgewiesen, weil `resolveElement()` dafür keine Referenz liefert. Auch mehrere Vorwegelemente scheitern am aktuellen Aufbau: Es gibt nur eine Vorwegreferenz; bei unterschiedlichen IDs ist mindestens eine Zuordnung nicht auflösbar, bei gleicher ID greift die Duplikatprüfung. Eine ausdrückliche Vorwegzählung fehlt, die Ablehnung ist damit aber nicht grundsätzlich ungesichert. Für Zugstraßen fehlen insbesondere vollständige Typprüfungen für Flanke/Unterwegs/Transport und vollständige rollenspezifische Sollwertprüfungen. Beispielsweise kann ein Gleis in der Flankenliste aufgelöst werden, ohne dass `isValid()` den unzulässigen Typ beanstandet. Der Konstruktoraufbau ersetzt die Gesamtvalidierung nicht.

### 3.3 Tatsächlich implementierte Fahrstraßenmechanik

`FsStatus` enthält in `fs.h:55` ausschließlich `Aus=0`, `Beanspruchungen=1`, `Stellen=2`, `Verschliessen=3`, `Eingestellt=4`. `getAn()` wird aus dem Status abgeleitet; ein unabhängiger Aktiv-Bool existiert nicht mehr.

`Fs::requestActivation()` (`fs.cpp:174`) und `requestAufloesung()` (`fs.cpp:182`) akzeptieren ausdrücklich nur `FahrstrassenTypId::Rangier`. Eine Zugprojektion kann als Laufzeitobjekt existieren, wird aber nicht aktiviert. `setAn()` delegiert an diese Anforderungen und umgeht die Typbeschränkung nicht. Der Test `testZulassung()` in [tests/rangierstrassen_tests.cpp](../tests/rangierstrassen_tests.cpp) prüft die Ablehnung einer Zugaktivierung ausdrücklich.

Die vorhandene ZP prüft Rangierfälle (`PRS`, `PR`, Rangier-Halt und Start-Sollwert -1/-2), nicht Zugfälle. `setBeanspruchungen()` und `beanspruchungenVorhanden()` verwenden fest `BEA_t::Rangier`. `requestWeichenStellungen()`, `weichenInSollstellung()` und `weichenVerschlossen()` betrachten jede in `members` vorhandene Weiche ohne Zugrollenfilter. Das ist für gültige Rangierprojektionen passend, weil dort nur Fahrwegweichen vorkommen dürfen; für Zugprojektionen würde es Flanken- und Transportweichen zu früh beziehungsweise unberechtigt einbeziehen.

`Fs::work()` (`fs.cpp:278`) implementiert Rangierstatus 1–4, Rückfall 4→3 und Wiederkehr 3→4. Die Zulassungsprüfung wird dabei nicht erneut ausgeführt. Beim ersten Verschließen wird jede Weiche in `verschlosseneWeichen` aufgenommen; der Rückfall verschließt sie nicht erneut. Auflösung wartet auf Rangier-Halt, nimmt die eigenen Verschlussbeiträge und Beanspruchungen zurück und setzt erst danach Status 0. `Fs::reset()` ist dagegen ein sofortiger Simulationsreset und wartet nicht auf eine verzögerte Haltbestätigung.

### 3.4 Vorhandene Elementzustände

| Klasse / Datei | Vorhandene Grundlage | Grenze |
|---|---|---|
| `EstwElement`, [estw/element.h](../estw/element.h) | `BEA_t::Zug`, `ZIF_t::Zug`, `FUEM_t::Zug_B/Zug_R`; Besetztzustand und Befahrbarkeitssperre | Belegung ist ein Bool pro Element, auch bei Signal und Weiche. Keine getrennte A-/B-Belegung. |
| `Gleis`, [estw/gleis.h](../estw/gleis.h) | Durchgehender BEA-Zustand; `GleisInitData` enthält PR und PZ | Es gibt `getPR()`, aber keinen lesenden PZ-Zugriff für `Fs`. |
| `Blind`, [estw/blind.cpp](../estw/blind.cpp) | BEA von `Gleis`, `getPRZ()`, `getPZZ()`, ZIF | Zug-ZIF wird ohne PZZ abgewiesen. Das steht in Spannung zur ZP-Forderung PRZ. |
| `Aufloese`, [estw/aufloese.cpp](../estw/aufloese.cpp) | Durchgehender BEA-Zustand und Reset; zusätzlich FAUF | Der angezeigte Auflösemelder wird über aktive Fahrstraßen gesteuert, nicht über FAUF allein. |
| `Weiche`, [estw/weiche.cpp](../estw/weiche.cpp) | BEA auf A/B/C, Istlage, Motorlauf, Auffahren, PSSL/PSSR, WUS und gemeinsamer Verschlusszähler | `HatFS()` ist ein Platzhalter mit `false`; daraus folgt kein wirksamer Flankenschutz. |
| `Signal`, [estw/signal.cpp](../estw/signal.cpp) | Getrennte BEA A/B, BEAF, FÜM, ZIF, HSV/VSV, Signalsperre, Ersatzsignal, Wiederholungssperre, Haltanforderung | Flag-Speicherung und Signalbedingungen bilden noch keinen zentralen Zugstraßenablauf. |

### 3.5 Zyklus, Oberfläche und vorhandene Tests

`Estw::work()` führt zuerst die Elementzyklen, danach die Fahrstraßenzyklen aus. `LupeForm` verwendet einen [WindowTimer](../windowtimer.h) im Fensterthread (`lupeform.cpp:238`): im Simulationsmodus zuerst `estwWork()`, dann `lupeWork()`. Der Weichenmotor wird in `Weiche::work()` über verstrichene Zeit fortgeschrieben. Eine neue Hintergrundthread-Infrastruktur ist nicht erforderlich.

Die Simulationsauswahl wird in [simulationselection.h](../simulationselection.h) gehalten; `LupeForm::button_verarbeiten_click()` (`lupeform.cpp:1070`) ruft die zentrale Fahrstraßenanforderung auf. Erfolg entfernt die Markierungen, Ablehnung behält sie und meldet über `showSimulationMessage()`/`showMessage()` vorläufig `abgewiesen`. `ToolForm::createLupeButtons()` (`toolform.cpp:431`) erstellt Verarbeiten und Reset; `setVerarbeitenEnabled()` synchronisiert den Button. Ein eigener Zuganschaltablauf in der Oberfläche fehlt und wird auch nicht benötigt.

Der Auflöseklick delegiert in `lupeform.cpp:1378` an `Estw::requestAufloesung()`. Diese Methode betrachtet alle passenden Fahrstraßen. Die Auflösemeldersichtbarkeit fragt `Estw::isAufloeseElementActive()` ab und funktioniert bereits während der Zwischenzustände mit Status >0.

Vorhandene Testgrundlagen sind `tests/rangierstrassen_tests.cpp` (ZP, Beanspruchungen, Weichenlauf, eigene Verschlussbeiträge, Rückfall, verzögertes Halt, frühe Auflösung und Reset), `tests/aufloese_melder_tests.cpp` (Auswahl, Mehrdeutigkeit, Auflösezuordnung), `tests/fahrstrassen_tests.cpp` (Editor, Sollwerte, Auswahlmodell, Verarbeiten, Fenstertimer) und `tests/validate_tests.cpp` (Stammdatenregeln). Die CMake-Ziele für Rangierstraßen und Auflösemelder stehen in `CMakeLists.txt:215` beziehungsweise `:196`. Es gibt noch keine eigenständige Testreihe für einen Zugstraßenautomaten. Der aktive Zugflankenfall wird im Rangiertest über eine eingesetzte Abfragefunktion simuliert; das ist kein Nachweis einer funktionsfähigen Zugflankensicherung.

## 4. Aktueller fachlicher Stand aus zugstrasse.md

### 4.1 Klar und ausreichend beschriebene Regeln

Die Rollen und ihre Kardinalitäten sind festgelegt: genau ein Signalstart, ein Signal-/Blindziel und ein Auflöser; höchstens ein Signal als Vorweg; beliebig viele Fahrweg-, Unterwegs-, Flanken- und Transportelemente innerhalb der jeweils erlaubten Elementtypen. Jedes Element darf gemäß Datenmodell nur einmal derselben Fahrstraße zugeordnet sein. Unterschiedliche Fahrstraßen dürfen dasselbe Element referenzieren.

Die ZP erfolgt vor der ersten Zustandsänderung. Sie prüft Ziel-ZIF, Startzulassung PZS, die im Text genannte Zielzulassung PRZ, PZ an echten Gleisen, vorhandene Beanspruchungen, Weichensperren/-verschlüsse/Auffahren, Start-Halt, Signalsperre, aktive Zugflankennutzung bei PZSP sowie Befahrbarkeitssperren. Vorhandene Beanspruchung wird unabhängig von ihrem Verursacher bewertet; auch eine eigene vorhandene Beanspruchung führt zur Ablehnung. Eine Eigentümer- oder Referenzzählung ist **für diese Zulassungsprüfung nicht gefordert** (`zugstrasse.md:62`). ZP und spätere Überwachung sind getrennte Vorgänge.

Die konkrete Strangzuordnung in Status 1 ist weitgehend bestimmt:

| Rolle / Elementtyp | Durch diese Zugstraße geforderte Beanspruchung |
|---|---|
| Startsignal | A; B unverändert |
| Zielsignal | B; A unverändert |
| Blindziel als Ziel | Durchgehend A–B |
| Auflöser als Auflöserrolle | Durchgehend A–B |
| Signal als Fahrweg, Gegenrichtung | A und B |
| Signal als Unterwegs | A und B |
| Signal als Vorweg | Nur B, ausdrücklich auch bei Signalziel |
| Weiche als Fahrweg | A und B bei Soll=+1/rechts; A und C bei Soll=-1/links; anderer Strang unverändert |
| Flanke und Transport | In Status 1 keine Beanspruchung nach diesen Regeln |

Gleis, Blind und Auflöser besitzen nach `estw.md`, Abschnitt „Stränge und Gleismelder“, jeweils einen durchgehenden A–B-Melder. Die pauschale Aufzählung von Blindzielen/Auflösern in Status 1 bleibt hinsichtlich Transportrollen redaktionell mehrdeutig; daraus darf keine zusätzliche Transportbeanspruchung erfunden werden.

Status 2 darf erst nach tatsächlich vorhandenen Beanspruchungen stellen; er betrifft Fahrwegweichen. Status 3 verlangt deren tatsächlich erreichte Sollstellung und verschließt sie. Status 4 verlangt Fahrwegverschluss, schaltet FÜM blinkend, fordert Unterwegs-Sollstellungen -1/-2/-3 an und stellt Flankenweichen; diese werden mit Erreichen ihrer Sollstellung verschlossen. Damit ist die alte Unklarheit „Stellanforderung oder Istlage vor Flankenverschluss“ im aktuellen Text grundsätzlich beseitigt.

Status 6 nennt Flankenweichen in Sollstellung und verschlossen, Flankensignale in Halt, die Freiprüfung und eine weitere Unterwegssignal-Aktion; FÜM wird Ruhelicht. Die Freiprüfung enthält inzwischen ausdrücklich Startsignal, Zielsignal, Fahrweg, Blindziel, Auflöser, Unterwegs, Vorweg und Transport. Der Wortlaut verlangt freie **Elemente**, nicht ausdrücklich nur beanspruchte Teilstränge. Das aktuelle elementweite Besetztmodell kann diese wörtliche Anforderung abbilden.

Status 7 verlangt die Voraussetzungen von Status 6 und FÜM Ruhelicht, fordert die projektierte Fahrtstellung des Startsignals an und aktualisiert zyklisch dessen VSV aus dem Vorwegsignal. `signale.md` ergänzt verbindlich die Zielquelle bei fehlendem Vorweg. Zugfahrt bedeutet HSV >0, also im vorhandenen Wertebereich 1–16. HSV <=0 gilt für Zugstraßen als Halt, wobei -3 zugleich als „nicht relevant“ definiert ist; seine Verwendung als Sicherungsnachweis verlangt deshalb besondere Klarstellung im Signalflankenschutz.

Die Auflösung wird per Auflöseklick ausgelöst. Nachgewiesenes Start-Halt muss dem Entfernen von Weichenverschluss und Beanspruchungen vorausgehen; Status 0 folgt zuletzt. Die Aufhebung des Flankenschutzes ist ausdrücklich noch offen.

### 4.2 Nicht als vollständige Spezifikation verwendbare Teile

Der Verweis auf einen fehlenden Status 5, das alte Diagramm, die alte pauschale Liste noch festzulegender Flankenschritte und die allgemeine Fahrtaktion bei Status 4 sind nicht konsistent mit den aktuellen Einzelbeschreibungen. Diese Textteile dürfen weder durch den historischen Bericht noch durch den vorhandenen Rangierautomaten automatisch ausgefüllt werden. Abschnitt 7 formuliert die erforderlichen Entscheidungen.

## 5. Abgleich mit zugstrasse_bericht.md

Die folgende Tabelle umfasst die Eingangseinschätzung, die Statusbewertung, alle acht nummerierten Kritikpunkte sowie die Anschlusswidersprüche. Teilweise erledigte Aussagen sind ausdrücklich aufgeteilt.

| Historische Aussage | Aktuelle Bewertung | Begründung und Konsequenz |
|---|---|---|
| Grundfolge plausibel, Zustandsautomat noch unvollständig | **Weiterhin gültig** | Der Automat ist weiterhin fachlich unvollständig; der präzisere aktuelle Hauptbefund ist der fehlende Status 5. |
| Statustabelle 0–3: einmalige ZP, Beanspruchung, Istlage vor Verschluss | **Weiterhin gültig; technische Grundlagen vorhanden** | `zugstrasse.md:68–113`; Rangierablauf in `Fs::requestActivation()`/`work()` trennt diese Schritte bereits. Das belegt noch keine Zugimplementierung. |
| Statustabelle 4 als Zwischenstufe | **Weiterhin gültig** | Status 4 ist jetzt ausdrücklich FÜM blinkend und beginnt auch die Flankenweichenaktionen. |
| Statustabelle 5 als vorhandener, nachvollziehbarer Schritt | **Überholt** | Der aktuelle Text enthält keinen Status 5. Aussagen über dessen Voraussetzungen sind derzeit nicht belegbar. |
| Punkt 1: Status 5 verlange blinkenden FÜM; Status 6/7 verlangten dadurch gleichzeitig Blink- und Ruhelicht | **In dieser konkreten Form überholt** | Der genannte Status 5 fehlt. Aktuell kann diese genaue Gleichzeitigkeit nicht aus dem Text nachgewiesen werden. Weiterhin relevant ist die Trennung von Eintrittsbedingungen und dauerhaften Sicherungsbedingungen bei der noch festzulegenden Zustandsfolge. |
| Punkt 2: Signalflankenschutz wird nicht hergestellt und nicht wirksam gebunden | **Weiterhin gültig** | Status 6 prüft Halt; eine vorangehende Aktion und Konflikt-/Haltebindung fehlen weiterhin. `Signal::setBEAF()` ist nur ein Setter. |
| Punkt 3: „Sofort verschließen“ müsse ausdrücklich erst nach tatsächlich erreichter Flankenweichenlage erfolgen | **Inzwischen fachlich erledigt** | `zugstrasse.md:126` sagt nun „Mit Erreichen dieser Stellung“. Die Aktion steht heute in Status 4, nicht in Status 5. Motorstillstand und idempotente Umsetzung bleiben technische Prüfpunkte. |
| Punkt 3: genau ein eigener Verschlussbeitrag, keine Erhöhung bei jedem Zyklus/Rückfall | **Für Rangierstraßen technisch erledigt; für Zugflanken weiterhin umzusetzen** | `verschlosseneWeichen`, erster Eintritt 2→3 und `releaseVerschluesse()` verhindern im Rangierablauf wiederholten Aufbau. Der gemeinsame Zähler existiert bereits. Zugrollen und ihre Freigabe fehlen. |
| Punkt 4: Status 1 vermischt ausgeschlossene Rollen mit pauschalen Blind-/Auflösetypen | **Weiterhin redaktionell relevant** | Rollenbegrenzung in Zeile 82 und pauschaler Aktionspunkt in Zeile 86 stehen weiterhin nebeneinander. Durchgehender Bereich für die Ziel-/Auflöserrollen ist nun ausdrücklich beschrieben; für Fahrwegtypen ergibt er sich zusätzlich aus der Melderdefinition. |
| Punkt 5: „Blindelement“ sei undefiniert; Zielsignal und Auflöser fehlten in der Freiprüfung | **Überholt beziehungsweise erledigt** | Der aktuelle Status 6 nennt „Blindziel“, „Zielsignal“ und „Auflöseelement“ ausdrücklich. Die Prüfung steht nicht mehr erst in Status 7. Rollen und Typen werden noch gemeinsam aufgezählt; eine eindeutige Rollentabelle bleibt hilfreich. |
| Punkt 5: getrennte A-/B-Belegung fehle im Code | **Technisch weiterhin richtig, als zwingende Vorarbeit nicht belegt** | `EstwElement` besitzt nur einen Besetzt-Bool. Der aktuelle Text prüft freie Elemente; dafür reicht er. Eine Verfeinerung auf Teilstränge wäre eine neue Festlegung und darf nicht vorausgesetzt werden. |
| Punkt 6: VSV-Zielersatzquelle, Blindziel ohne Vorweg, negative HSV, Unterwegssignal-VSV und Initialisierung unvollständig | **Weiterhin teilweise gültig** | Die Zielersatzquelle steht bereits verbindlich in `signale.md`, fehlt aber in der Zugbeschreibung und im Ablaufcode. Blind-/Negativfälle sowie Unterwegsbedarf und Zeitpunkt müssen geklärt werden. „VSV beim ersten Freigeben korrekt“ war eine Empfehlung, keine schon vorhandene Detailregel. |
| Punkt 7: Rückfall, Wiederanlauf, FÜM und Zug-ZIF fehlen | **Für Zugstraßen weiterhin gültig** | Rangierregeln und deren Implementierung sind konkret vorhanden, gelten aber nicht automatisch für Zugstraßen. Die aktuelle Zugbeschreibung ergänzt keine entsprechende Übergangstabelle. |
| Punkt 8: FÜM Ruhelicht könne vor Fahrwegfreiprüfung gesetzt werden, da diese erst vor Status 7 erfolge | **Überholt** | Freiprüfung ist jetzt Voraussetzung von Status 6 vor der Ruhelichtaktion. Ein besetzter genannter Bereich erfüllt diese Voraussetzung nicht. |
| Punkt 8: Status 7 als Anforderung oder Bestätigung; positive Start-Sollstellung | **Teilweise offen, teilweise ableitbar** | Anfordern versus bestätigtes Fahrtbild ist offen. Ein positiver Sollwert folgt bereits aus „Fahrtstellung“ und `signale.md`; es fehlt die dazu passende Laufzeitprüfung, während die allgemeine Datenvalidierung auch nichtpositive Signalwerte zulässt. |
| Anschlusswiderspruch: Diagramm endet mit Fahrtstellung bei Status 4; offene Liste ist nicht nachgeführt; `fahrstrasse.md` nennt Status 4 | **Weiterhin gültig** | `zugstrasse.md:172–221` und `fahrstrasse.md`, Ausnahme von zyklischer Ausführung, widersprechen der Fahrtaktion in Zugstatus 7. Die spezielle Zugbeschreibung hat Vorrang, die Gesamtdokumentation braucht trotzdem eine konsistente Festlegung. |
| Abschlusssatz „Ich habe keine Dateien verändert“ | **Historische Ausführungsangabe** | Keine fachliche oder technische Voraussetzung; dieser Bericht leitet daraus keine Aussage zum aktuellen Code ab. |

Weitere veraltete Angaben in der ergänzenden Wissensgrundlage `estw.md` sind für die technische Planung zu beachten: Rollen-IDs werden dort noch als Annahme bezeichnet; `Fs::work()` wird als zyklischer `isValid()`-Aufruf mit eigenständigem `an` beschrieben; für die Simulation werden zwei Timer genannt. Der aktuelle Code enthält benannte IDs, Aktivität aus Status und einen gemeinsamen Fenstertimer. Diese Aussagen wurden nicht als aktuelle technische Defizite übernommen. Die bestehenden Dokumente werden durch diesen Analyseauftrag nicht geändert.

## 6. Bewertung der einzelnen Funktionsbereiche

### 6.1 Auswahl und Anschalten

**Technisch weitgehend vorbereitet.** `SimulationSelection`, `ToolForm`, `LupeForm::button_verarbeiten_click()`, `Fs::matchesSelection()` und `Estw::requestFahrstrasse()` sind unmittelbar wiederverwendbar. Das fachliche Ermitteln bedeutet hier Auswahl einer vorhandenen Projektion, nicht Berechnung eines neuen Fahrwegs aus Rasterkoordinaten.

Vor Zugaktivierung sind die unvollständige Laufzeit-Strukturprüfung und die typabhängige Weiterleitung zu ergänzen. `FsStatus::Eingestellt=4` darf nicht zugleich ungeprüft als Zugfahrtstatus interpretiert werden. Für gemischte Typen mit gleicher Start-/Zielkombination bleibt die dokumentierte Mehrdeutigkeitsablehnung bestehen, solange keine andere Bedienung beauftragt wird.

### 6.2 Zulassungsprüfung

Die folgende Zuordnung betrachtet sämtliche 16 Zug-ZP-Bedingungen. „Baustein vorhanden“ bedeutet keine bereits ausgeführte Zug-ZP.

| Nr. | Anforderung | Vorhandene Grundlage / erforderliche Ergänzung |
|---|---|---|
| 1 | Kein Ziel-ZIF | `ZielElement::getZIF()` vorhanden. Rangier-ZP fragt ihn bewusst nicht ab; für Zug getrennt ergänzen. |
| 2 | PZS am Start | `Signal::GetPZS()` vorhanden; Rangier-ZP verwendet PRS. |
| 3 | PRZ am Ziel laut aktuellem Zugtext | Wrapper vorhanden; Widerspruch zu PZZ vor fachlicher Umsetzung entscheiden. |
| 4 | PZ an jedem zugeordneten echten Gleis | PZ wird in `LupeForm` geladen, Getter fehlt. Blind/Auflöser trotz C++-Ableitung nicht als echte Gleise behandeln. Auch Transport-Gleise fallen unter „jedes zugeordnete Fahrstrassenelement“. |
| 5 | Keine vorhandene Beanspruchung an Fahrweg/Auflöser/Blind | `getBeansprucht()` vorhanden; beim Signal umfasst es auch BEAF. Rangier-Vorbild vorhanden; Rollen-/Typabgrenzung für Transport-Blind/Auflöser präzisieren. |
| 6 | Vorweg B frei | `Signal::getBEAB()` vorhanden; Zugrollenprüfung ergänzen, A nicht pauschal mit ablehnen. |
| 7 | Unterwegs A und B frei | Beide Getter vorhanden; neue Rollenprüfung nötig. |
| 8 | Keine widersprechende Weichensperrlage | PSSL/PSSR, WUS und Istlage vorhanden. Rangierprüfung wiederverwendbar; Reichweite umfasst laut Zug-ZP Weichen als Fahrstrassenelement, also nicht nur Fahrwegweichen. |
| 9 | Kein widersprechender Weichenverschluss | Zähler und Istlage vorhanden; gleiche Lage bei fremdem Verschluss ist grundsätzlich nicht allein deshalb verboten. |
| 10 | Nicht aufgefahren | `Is_Aufgefahren()` vorhanden; im Rangierprüfpfad schon verwendet. |
| 11 | Start A frei | `getBEAA()` vorhanden; Start B bleibt unabhängig. |
| 12 | Zug-Halt am Start | Fachlich HSV <=0 gemäß `signale.md`; `isRangierHalt()` prüft dagegen HSV >=0 und ist ungeeignet. Behandlung von -3 als Sicherungszustand ausdrücklich beachten. |
| 13 | Keine Signalsperre am Start | `GetFSS()` vorhanden; entspricht dem dokumentierten SS-Zustand. |
| 14 | Zielsignal B frei | Getter und Rangierprüfbaustein vorhanden; bei Blindziel nicht anwenden. |
| 15 | PZSP mit aktiver Zugflankenrolle | `Estw::isActiveZugFlankenschutz()` und `Fs::usesActiveZugFlankenschutz()` sind passend auf Typ Zug, Status >0 und Rolle Flanke vorbereitet. Aktuell kann keine Zugstraße diesen Aktivzustand erreichen. |
| 16 | Keine Befahrbarkeitssperre | `EstwElement::getBefahrbarkeitssperre()` vorhanden; alle Rollen prüfen. |

Negative ZP muss ohne Beanspruchungs-, Verschluss-, Stellungs- oder Melderänderung enden. Nach erfolgreichem Übergang aus 0 darf dieselbe ZP nicht erneut angewandt werden: Schon die eigene Zugbeanspruchung würde sonst zu einer Ablehnung führen. Eine Besetztprüfung ist im aktuellen Text erst für das spätere Sicherungsniveau festgelegt; sie darf nicht ungefragt als zusätzliche ZP-Bedingung ergänzt werden.

### 6.3 Beanspruchung

**Zustandsfelder vorhanden, Rollensteuerung fehlt.** `Gleis::setBEA()` kann den gesamten A–B-Bereich von Gleis/Blind/Auflöser abbilden. Signal und Weiche besitzen die benötigten einzelnen Stränge. Für Vorweg und Unterwegs sind rollenbezogene Zugaktionen und deren Istprüfung neu erforderlich. Die vorhandenen Rangierfunktionen berücksichtigen diese Signalrollen nicht; ihre Weichenbehandlung greift dagegen zu breit.

Für die Zulassung braucht es keine Eigentümerzählung. Für die Rücknahme ist dennoch festzuhalten, welche Zustände die konkrete Fahrstraße tatsächlich gesetzt hat. Die bestehende Rangierlogik löscht eigene beanspruchte Signal-/Weichenstränge nur beim passenden BEA-Typ und verwendet für Gleisderivate zusätzlich eine Abfrage anderer aktiver Straßen (`Fs::usesGleisBeanspruchung()`, `Estw::addFahrstrasse()`). Diese Abfrage ist ausdrücklich auf Rangierstraßen begrenzt und lässt sich nicht unverändert für Zugbeiträge verwenden.

Das Skalarmodell kann Rangier- und Zugbeanspruchung auf demselben Strang nicht gleichzeitig ausdrücken. Die ZP verhindert viele solche Überschneidungen bereits. Daraus folgt **kein** allgemeiner Bedarf, für jeden Fahrweg einen Referenzzähler einzuführen. Geteilte Flankennutzung und eventuelle Transportbindung brauchen jedoch eigene Regeln und eine dazu passende technische Beitragsverwaltung. Fremde Start-B-/Ziel-A-Beanspruchungen und der nicht verwendete Weichenstrang dürfen beim Rücknehmen nicht verändert werden.

### 6.4 Weichen und Verschlüsse

**Die wesentliche Mechanik ist vorhanden.** `Weiche::requestStellung()` (`weiche.cpp:177`) fordert gezielt links/rechts an, bestätigt bereits erreichte Lage ohne Motorlauf und weist unerlaubtes Stellen bei Auffahren, Verschluss, Belegung, WUS oder projektierten Sperren zurück. Das Ergebnis einer angenommenen Stellanforderung ersetzt nicht die Istlagenprüfung. `Fs::weichenInSollstellung()` prüft Lage, Motorstillstand und Auffahrzustand bereits.

Der gemeinsame Zähler wird von `setVerschluss()` erhöht und von `unSetVerschluss()` vermindert, mit Untergrenze 0 (`weiche.cpp:78–91`). Stellanforderung, manueller WU und AWU berücksichtigen Verschluss; ein separater Flankenverschlusszähler wäre entgegen `weiche.md`. `verschlosseneWeichen` ist als Muster für genau einen eigenen Beitrag geeignet. Es muss auf getrennte fachliche Erwerbs-/Freigabezeitpunkte von Fahrweg und Flanke erweitert werden, ohne die Summe in zwei elementseitige Zähler aufzuteilen.

Die gleiche Weiche kann wegen der eindeutigen Elementzuordnung innerhalb einer Fahrstraße nicht zugleich Fahrweg- und Flankenrolle haben, sehr wohl aber in mehreren Fahrstraßen unterschiedlich vorkommen. Fahrwegstellung in Status 2 und Fahrwegverschluss in Status 3 müssen deshalb rollenbezogen bleiben. Eine Transportweiche darf aus diesen Schritten keinen Stell- oder Verschlussauftrag erhalten, nur weil sie technisch eine Weiche ist.

### 6.5 Signale und Signalstellungen

**Speicherung und Darstellung sind vorhanden; der Steuerungsvertrag ist unvollständig.** Eine direkte Übernahme von `requestRangierFahrt()` scheidet aus: Die Methode erlaubt nur -1/-2, setzt unmittelbar HSV/VSV und umgeht bewusst die manuelle Wiederholungssperre. Zugfahrt ist positiv.

Der allgemeine Pfad `Signal::SetHSV()` ist ebenfalls nicht ungeprüft als Zugstraßenanforderung geeignet:

- `condition4()` (`signal.cpp:383`) fordert PZS, Zugbeanspruchung A und unter anderem keine Wiederholungs-, Signal- oder Ersatzsignalbedingungen. Bei Fahrt setzt `SetHSV()` die Wiederholungssperre WSP. `HaGT()` nimmt diese Sperre nicht zurück; eine spätere erneute positive Anforderung kann deshalb abgewiesen werden. Eine Freigaberegel für diesen Zugstraßenfall fehlt.
- `condition4()` prüft BEAF nicht, obwohl andere Signalpfade Flankenschutz berücksichtigen. Auch `requestRangierFahrt()` bindet das Signal nicht über BEAF. Ein gesetztes Flankenflag allein verhindert daher nicht alle späteren Fahrtanforderungen.
- Unterwegssignale erhalten Zugbeanspruchung A/B. `condition3()` für -1 weist Zugbeanspruchung ab, `condition2()` für -2 weist Zugbeanspruchung B ab; -3 über `condition1()` verlangt wiederum PZS und weitere Bedingungen. Diese Pfade erfüllen die ausdrücklich verlangte Unterwegs-Sollstellung in einer Zugstraße nicht allgemein.
- `setBEAA()` ruft bei jeder Änderung `HaGT()` auf, `setBEAB()` tut dies nicht (`signal.cpp:186`). Der Aufbau einer Zugbeanspruchung kann also bereits eine Signalstellung ändern. Besonders bei Unterwegssignalen sind Reihenfolge und Wiederfreigabe zu berücksichtigen.
- `SetVSV()` speichert ungeprüft jeden Integer. `SetHSV()` protokolliert einen ungültigen Wertebereich, führt aber keinen unmittelbaren generellen Abbruch allein aufgrund dieser Meldung aus. Die Fahrstraßensteuerung braucht deshalb geprüfte Werte und eine geeignete Ergebnis-/Istkontrolle.

`SignalInitData` trägt keine `UnterElementArt`; diese gelangt derzeit nur in die Lupe-Darstellung. Falls Haupt-/Rangiersignalart die Zugstartzulassung oder Unterwegssteuerung begrenzen soll, muss diese Regel ausdrücklich festgelegt und im Laufzeitmodell verfügbar gemacht werden. Der aktuelle Rollentext fordert nur „Signal“; eine zusätzliche Hauptsignalpflicht wird hier nicht unterstellt.

Ein eigener zentral kontrollierter Signal-Anforderungspfad ist eine technische Option. Seine Regeln zu WSP, BEAF, Unterwegs-Sollwerten und Bestätigung müssen vor dem produktiven Zugablauf fachlich feststehen; die Rangierausnahme darf nicht einfach verallgemeinert werden.

### 6.6 Flankenschutz

**Datenreferenzen und Teilzustände vorhanden, Fachmechanik nicht vollständig.** Flankenweichen werden laut aktuellem Text ab Status 4 gestellt und nach Erreichen der Sollstellung verschlossen. Ihr Schutzbeitrag kann denselben Weichenzähler nutzen. Die Transportliste kann für die Freiprüfung ausgewertet werden; eine Transportbeanspruchung folgt daraus nicht.

Für Signale fehlen Haltanforderung, Bindung und Verhalten bei Konflikten. `BEAF` ist ein einzelner Bool ohne Herkunft oder Beitragszahl; `Weiche::HatFS()` ist keine wirksame Schutzabfrage. Bei zwei zulässigen Schutzforderungen darf das Auflösen einer Fahrstraße nicht das letzte Flag der anderen löschen. Ob solche gemeinsame Signalflankennutzung zulässig ist und wie sie freigegeben wird, muss festgelegt werden; erst daraus folgt die konkrete Verwaltung, etwa durch eigene Beiträge oder Abfrage aktiver Anforderungen.

Die vorhandene PZSP-Abfrage schützt eine **neu anzufordernde Zielverwendung** gegen ein bereits aktives Zugflankensignal. Sie beantwortet nicht den umgekehrten Ablauf: Eine bestehende Rangier-/Zugfahrt verwendet das Signal bereits, anschließend fordert eine neue Zugstraße dort Flankenschutz. Priorität, Ablehnung oder Warten sind derzeit nicht spezifiziert. Beide Anforderungsreihenfolgen müssen in die spätere Konfliktprüfung eingehen.

### 6.7 Statusüberwachung

**Zyklus vorhanden, Zugprädikate und Zugrückfälle fehlen.** Der vorhandene Automat prüft Rangierbeanspruchungen, Weichenlage und Verschluss; weder Flankenschutz, Fahrwegfreiheit, Unterwegssignale noch Zug-FÜM/VSV werden von ihm überwacht. Während des Einstellens kann er auf fehlende Istzustände warten, ab Rangierstatus 4 fällt er nach 3 zurück. Diese konkrete Rückfallnummer ist keine Zugregel.

Die neue Zugüberwachung muss Eintrittsaktionen, wiederholbare Stellanforderungen und dauerhafte Sicherungsprüfungen unterscheiden. Eine blinkende Anzeige ist nicht automatisch eine dauerhaft zu erfüllende Vorbedingung des Ruhelichtzustands. Eine einmalige Fahrtanforderung ist außerdem von zyklischer VSV-Aktualisierung zu trennen. Die Reihenfolge `Element::work()` vor `Fs::work()` erlaubt, angeforderte und bestätigte Zustände über mehrere Zyklen abzubilden.

Belegung nach Fahrtfreigabe fällt unter die allgemeine Haltregel bei weggefallenen Voraussetzungen. Ob die Zugstraße danach aktiv auf einem niedrigeren Sicherungsniveau bleibt und ob nach Freifahren wieder Fahrt angefordert werden darf, ist offen. Eine automatische Übernahme des Rangier-Wiederanlaufs wäre eine fachliche Entscheidung.

### 6.8 Auflösung und Rücknahme

**Der Halt-vor-Freigabe-Baustein ist wiederverwendbar.** `requestAufloesung()` stellt die Anforderung, `work()` wartet und gibt dann frei. Die vorhandene Haltprüfung ist rangierspezifisch und muss für Zug durch eine fachlich passende Prüfung ersetzt werden. Die heutige konkrete Signalimplementierung erreicht Halt unmittelbar; `DelayedSignal` in den Rangiertests zeigt, wie die zeitliche Trennung geprüft werden kann.

Zu ergänzen sind die fachlich festgelegte Behandlung von Unterwegssignalen, Flankensignalen, Fahrweg- und Flankenverschlussbeiträgen, Vorweg-/Unterwegsbeanspruchungen, FÜM, Ziel-ZIF und VSV. Fremde Vorweg-A-Beiträge oder weiter aktive Schutzforderungen dürfen nicht als Teil einer pauschalen Elementrücksetzung verschwinden. Befahrbarkeitssperren und Besetztmeldungen sind ebenfalls keine von der Zugstraße erworbenen Sicherungsbeiträge; ihre Rücknahme folgt nicht aus der normalen Fahrstraßenauflösung.

`Estw::reset()` setzt erst die Fahrstraßen, dann sämtliche Elemente zurück. Das ist ein vollständiger Simulationsreset einschließlich Besetztzuständen und Sperren. Er muss von normaler Auflösung getrennt bleiben. Ein Auflöseversuch während unvollständigen Einstellens benötigt ebenfalls einen definierten Weg, der bereits erworbene Beiträge genau einmal abbaut.

### 6.9 Anzeige und UI-Synchronisation

**Großteils vorbereitet.** [lupe/lupegleis.cpp](../lupe/lupegleis.cpp) und [lupe/lupeweiche.cpp](../lupe/lupeweiche.cpp) werten Zug-BEA bereits aus. [lupe/lupesignal.cpp](../lupe/lupesignal.cpp), `LupeSignal::work()`, zeigt getrennte A-/B-Beanspruchung, BEAF, Zug-ZIF und FÜM blinkend/Ruhelicht. HSV >0 mit VSV=0 beziehungsweise >0 hat bereits unterschiedliche KS-Darstellung. [lupe/lupeblind.cpp](../lupe/lupeblind.cpp) kennt Zug-ZIF und Zugziel-Programmfall; [lupe/lupeaufloese.cpp](../lupe/lupeaufloese.cpp) kennt Zug-BEA und die aktive Auflösezuordnung.

Das Vorhandensein der Darstellungszweige bedeutet nicht, dass der zentrale Automat diese Zustände bereits setzt. Im Simulations-ToolForm existiert keine laufende Fahrstraßenstatusliste; die wesentliche Rückmeldung erfolgt über Lupenmelder. Eine solche Liste wäre eine zusätzliche Produktanforderung, keine notwendige Voraussetzung des beschriebenen Ablaufs.

Zu prüfen sind vor allem Zustandsrücknahme und Aktualisierungsdetails: `LupeSignal::work()` invertiert `blinkState` bei jedem Darstellungsaufruf, also auch nach Bedienaktionen außerhalb des Timertakts. `LupeBlind::work()` kann das Dreieck bei Zugbeanspruchung ohne PZZ ausblenden, stellt dessen Sichtbarkeit in den regulären ZIF-Zweigen aber nicht ausdrücklich wieder her. `LupeWeiche` zeigt Motorlauf, Auffahren und Strangbeanspruchung; ein vollständiger eigener Flanken-/Verschlussmelder folgt daraus nicht. Diese vorhandenen Besonderheiten sind gezielte UI-Prüfpunkte und keine Begründung für einen neuen Simulationsloop.

### 6.10 Sonderfall Blindziel / nachfolgendes Zielsignal

Der aktuelle Datenvertrag besitzt genau ein Fahrstrassenziel. Ist dies ein Blindziel, ist ein nachfolgendes Signal im Zugstraßenmodell nur dann eindeutig verfügbar, wenn es als Vorwegelement projektiert ist. `FsInitData::vorwegElement` und die entsprechende Projektierungszeile bieten diese Referenz bereits. Ein automatisches Ermitteln eines nachfolgenden Signals aus der gezeichneten Topologie ist im untersuchten Aufbau nicht implementiert und wird von der aktuellen Auswahlregel nicht gefordert.

Für diesen Fall sind die wesentlichen Beanspruchungsgrenzen ausdrücklich bestimmt: Blindziel durchgehend A–B, Vorwegsignal ausschließlich B; Vorweg A bleibt unverändert. Dadurch kann die Start-A-Verwendung desselben Signals durch eine anschließende Fahrstraße grundsätzlich getrennt abgebildet werden. Die Reihenfolge der Betätigung und alle übrigen ZP-/Signalbedingungen müssen dennoch passen. Beim Auflösen der ersten Zugstraße darf Vorweg A der anschließenden Straße nicht gelöscht oder deren Signal pauschal zurückgesetzt werden.

Mit Vorwegsignal ist die Quelle für die Start-VSV vorhanden; ohne Vorweg verweist `signale.md` auf das Zielelement, ein Blindziel besitzt aber keine HSV. Hier fehlt eine verbindliche Regel: unzulässige Projektierung, definierter Ersatzwert oder anderer ausdrücklich projektierter Bezug. Der Bericht wählt keine davon. Auch negative HSV des Vorwegs dürfen ohne festgelegte Abbildung nicht unverändert in den VSV-Bereich 0–16 übernommen werden.

## 7. Offene fachliche Fragen und erforderliche Ergänzungen

### 7.1 Vor Umsetzung des vollständigen Zugablaufs zwingend zu entscheiden

| ID | Konkrete Frage / benötigte Festlegung | Betroffener Implementierungsteil |
|---|---|---|
| F1 | Existiert ein Status 5, und wenn ja: Name, Eintritt, Aktionen, dauerhafte Bedingungen und Folgezustand? Oder ist der Verweis aus Status 6 zu entfernen? Welche vollständige Statusfolge ersetzt das alte Diagramm und die allgemeine Fahrtaktion bei Status 4? | Statusmodell, FÜM, Überwachung, Tests |
| F2 | Soll Zugzielzulassung tatsächlich PRZ sein, PZZ oder eine ausdrücklich bestimmte Kombination? Wie müssen dann Blind-ZIF und Darstellung reagieren? Bitte die vier PRZ/PZZ-Kombinationen für Signal- und Blindziel eindeutig festlegen. | ZP, Zielwrapper, Blind-ZIF, Validierung |
| F3 | Wann wird an Flankensignalen Halt angefordert, welche HSV gelten dafür als bestätigter Schutz und wie wird die Haltbindung gegen weitere Zug-/Rangieranforderungen und Ersatzsignalbedienung erhalten? Bedeutet -3 „nicht relevant“ auch hier einen zulässigen Schutz? | Signalflankenschutz und Signalvertrag |
| F4 | Sind mehrere gleichzeitige Flankenanforderungen an dasselbe Signal zulässig? Was geschieht bei bestehender Fahrt-/Zielverwendung vor neuer Flankenanforderung und bei widersprechenden Weichenanforderungen? Ablehnen, warten oder andere Reaktion? | Konfliktlogik in beiden Anforderungsreihenfolgen |
| F5 | Welche genauen Bedingungen und welche Reihenfolge gelten beim Aufheben des Flankenschutzes? Müssen neben Start-Halt auch Unterwegssignale Halt bestätigt haben? Wann werden Fahrweg- und Flankenverschlüsse, BEAF, Beanspruchungen, FÜM und ZIF entfernt? | Normale und vorzeitige Auflösung |
| F6 | Auf welchen Status fällt die Zugstraße bei Verlust welcher Sicherungsbedingung zurück? Welche Beiträge bleiben bestehen, welches FÜM/ZIF-Bild folgt und wann ist Wiederfreigabe zulässig? Bitte reguläre Belegung durch die Zugfahrt und technische Störung unterscheiden, soweit unterschiedliche Reaktionen beabsichtigt sind. | Zyklische Überwachung und Wiederanlauf |
| F7 | Wann wird Zug-ZIF gesetzt, wann zurückgenommen und wann wieder gesetzt? Wie wird verhindert, dass die ZIF-Rücknahme einer Rangierstraße die Zielbindung einer anderen aktiven Straße entfernt, falls eine solche gemeinsame Nutzung überhaupt zulässig sein soll? | Zielreservierung und gemischte Fahrstraßen |
| F8 | Welche zusätzliche Funktion haben Unterwegssignal-Aktionen in Status 6 nach der Sollanforderung -1/-2/-3 in Status 4? Ist die tatsächlich erreichte Sollstellung Übergangsbedingung oder nur die Anforderung? Dürfen positive Unterwegs-Sollwerte vorkommen, obwohl der aktuelle Validator sie ausschließt? | Unterwegssteuerung und Freigabeprädikate |
| F9 | Was bedeutet Status 7: Fahrt angefordert oder Fahrt tatsächlich bestätigt? Wie wirken WSP, SS, ZS1 und BEAF bei erster Freigabe, Rückfall, Wiederfreigabe und neuer Einstellung nach Auflösung? Welche dieser Regeln gehören zum Signal, welche zur zentralen Fahrstraße? | Zentraler Signal-Anforderungspfad |
| F10 | Wie wird VSV bei Blindziel ohne Vorweg und bei Vorweg-/Ziel-HSV -1/-2/-3 ermittelt? Wann muss die erste VSV gültig sein? Benötigen Unterwegssignale VSV, und falls ja, aus welcher eindeutig projektierten Quelle? | Geschwindigkeitsführung und KS-Anzeige |
| F11 | Soll der pauschale Status-1-Aktionspunkt ausschließlich die zuvor genannten Rollen meinen? Werden Blind-/Auflösetypen in Transportrolle nur gemäß Transportregeln behandelt? Gilt ZP 5 für solche Typen unabhängig von ihrer Rolle? | Beanspruchungs-/Zulassungsmatrix |
| F12 | Sollen Transportelemente nur frei überwacht werden oder zusätzlich gegen andere Nutzung gebunden sein? Falls Bindung: wann erwerben, wie Konflikte erkennen, wann freigeben? Was geschieht, wenn Flankenschutz nicht herstellbar ist: dauerhaft warten, abbrechen oder anderer festgelegter Zustand? | Flankenschutzraum, Fortschritt, Abbruch |

Die Fragen blockieren jeweils die davon abhängigen Fachschritte; sie verhindern nicht jede technische Vorarbeit. Eine später ausdrücklich begrenzte erste Ausbaustufe könnte offene Sonderfälle ausschließen. Ein solcher Ausschluss wäre jedoch gesondert zu beauftragen und dürfte nicht als vollständige Zugstraßenimplementierung bezeichnet werden.

### 7.2 Präzisierungen und Entscheidungen während der Umsetzung

Die Freiprüfung kann nach dem derzeitigen Wortlaut elementweit über `getBesetzt()` umgesetzt werden. Soll stattdessen bei Ziel-/Vorwegsignalen nur B geprüft werden, muss **vor diesem abweichenden Verhalten** ein genauer Strangvertrag festgelegt und das Belegungsmodell erweitert werden. Diese Änderung ist derzeit keine belegte Pflichtvorarbeit.

Eine positive Zugstart-Sollstellung ist aus der Fahrtdefinition bereits ableitbar. Umsetzungsort und Fehlermeldung sind technische Entscheidungen. Ebenso sind genau einmal erworbene und genau einmal freigegebene Verschlussbeiträge notwendige technische Umsetzung der gemeinsamen Zählerregel; dazu braucht es keinen neuen zweiten Zähler und keine freie Wahl der Freigabereihenfolge.

Während der Implementierung entscheidbar sind die interne Aufteilung in Hilfsfunktionen oder getrennte Ablaufstrategien, private Statusrepräsentation, Beitragsdatenstruktur, Diagnoseausgabe über die vorhandene Simulationsmeldungsfunktion und Testaufbau. Falls dabei öffentliche Schnittstellen geändert werden müssen, ist dies in einem späteren Auftrag ausdrücklich abzudecken. Ob nur Hauptsignale Zugstart sein dürfen, ist dagegen eine fachliche Ergänzung, falls eine solche zusätzliche Beschränkung gewünscht wird; aus der gegenwärtigen Rollenregel wird sie hier nicht abgeleitet.

## 8. Technische Vorarbeiten

### 8.1 Gemeinsame Infrastruktur und fachliche Trennung

| Gemeinsam sinnvoll nutzbar | Zugstraßenspezifisch zu implementieren |
|---|---|
| Projektionsauflösung, stabile IDs, Pflichtrollen und sichere Referenzprüfung | Zug-ZP mit PZS/PZ/ZIF und den entschiedenen Zielregeln |
| Auswahl, Mehrdeutigkeitsablehnung, zentrale Anforderung, Auflöserzuordnung | Statusfolge einschließlich Flanke, FÜM und Zugfahrt |
| Zugriff auf BEA-Stränge, Istlage und Motorzustand | Rollenmatrix für Zug-BEA, insbesondere Vorweg/Unterwegs/Transport |
| Gezielter Weichenauftrag und gemeinsamer Verschlusszähler | Unterschiedliche Zeitpunkte für Fahrweg- und Flankenweichen |
| Pro Fahrstraße erfasste eigene Verschlussbeiträge | Signalflankenbindung, Konflikte und Schutzfreigabe |
| Muster „Halt anfordern – Istzustand abwarten – eigene Beiträge freigeben“ | Zug-Haltbegriff, Unterwegssignale, Zug-ZIF, FÜM und VSV |
| Simulationszyklus im UI-Thread, Darstellung nach mechanischem Schritt | Zugüberwachungsprädikate und festgelegter Wiederanlauf |

Ein zweiter vollständiger Bedien- oder Simulationspfad ist nicht nötig. Gemeinsame technische Hilfen dürfen nicht automatisch gleiche fachliche Bedingungen bedeuten. Insbesondere bleiben PR/PRS statt PZ/PZS, ZIF ohne Einfluss in der Rangier-ZP, Rangier-Halt, Fahrt bei Rangierstatus 4, fehlende Rangier-Freiprüfung und Rangier-Rückfall 4→3 typabhängig.

### 8.2 Konkrete Ergänzungen vor Freigabe einer Zugaktivierung

1. Laufzeit-Strukturprüfung vervollständigen: alle acht Rollen, Typkombinationen, unbekannte Rollen, Vorweganzahl, Pflichtreferenzen und Sollwerte. Vorhandene Prüfregeln möglichst gemeinsam nutzbar machen, ohne Speichern unvollständiger Projektionen zu verbieten. Betroffen: `Fs::isValid()`, `Estw::createFahrstrasse()`, gegebenenfalls `validate.cpp` und Editorprüfungen.
2. Vorhandenes PZ lesbar machen; nach F2 gegebenenfalls PZZ im Zielwrapper verfügbar machen. Echte Gleise von den technischen Gleisunterklassen unterscheiden. Betroffen: `Gleis`, `ZielElement`, `Fs`.
3. Rollenbezogene Zustandszugriffe und Istprüfungen vorbereiten. Eine gemeinsame Hilfe darf den Beanspruchungstyp als Parameter kennen, aber nicht alle Rangierrollenregeln zu Zugregeln erklären. Erwerb und Rücknahme müssen dieselben Stranggrenzen verwenden.
4. Eigene Sicherungsbeiträge für Fahrweg und Flanke getrennt nachverfolgbar machen; Weichen behalten genau einen gemeinsamen Zähler. Änderungen der bestehenden Abfrage anderer Gleisbeanspruchungen auf gemischte Nutzung prüfen. Eine neue generelle BEA-Referenzzählung ist daraus nicht vorgegeben.
5. Nach F3–F10 einen zentralen Signalvertrag implementieren: Anforderung, Istnachweis, Wiederholungsbedingungen, Unterwegs-Sollwerte und VSV-Bereich. Bestehende manuelle Signalbedingungen und Rangierfahrt müssen gezielt auf Verträglichkeit geprüft werden.
6. Statuswerte und Eintrittsaktionen typsicher trennen. `getAn()` bleibt aus Status >0 ableitbar; Rangierstatus 4 behält seine bisherige Bedeutung. Keine zyklische Wiederholung der ZP oder erneute Zählererhöhung beim Wiedererreichen eines Niveaus.
7. Zugtestdaten mit Signalziel, Blindziel plus Vorweg, Null-Fahrwegelementen, Flanke und Transport ergänzen. Bestehende Regressionen bewahren; nach Zugimplementierung echte gemischte Konflikttests statt ausschließlich simulierter Zugflankenabfrage hinzufügen.

Besonders zu vermeiden sind das Überschreiben fremder Stränge, pauschales `reset()` bei normaler Auflösung, Fahrt trotz abgewiesener Signalstellung, Flankenflag-Freigabe bei weiter bestehender Anforderung und versteckte Abhängigkeit von der Reihenfolge der Fahrstraßen im `Estw::work()`-Container.

## 9. Empfohlene Implementierungsreihenfolge

Die Schritte sind bewusst getrennt abnehmbar. Fachfragen werden vor dem jeweils abhängigen Schritt beantwortet; eine ausführbare vollständige Zugaktivierung wird erst mit definierter Überwachung und Auflösung freigegeben.

### Schritt 1 – Fachvertrag und Abnahmefälle festlegen

- **Ziel:** F1–F12 beantworten und eine vollständige Übergangs-, Rollen- und Freigabetabelle erstellen.
- **Fachbereiche:** Status, Zielzulassung, Flanke, Signale, VSV, Rückfall und Auflösung.
- **Dateien/Klassen:** Später autorisierte Aktualisierung von `zugstrasse.md`, `fahrstrasse.md`, gegebenenfalls `signale.md` und Validierungsvorgaben; noch kein produktiver Zugautomat.
- **Abhängigkeiten:** Keine technischen Vorgänger; Entscheidung durch den fachlich Verantwortlichen.
- **Prüfkriterien:** Kein Verweis auf einen unbeschriebenen Status; jede Anforderung besitzt Istnachweis und Freigaberegel; alle PRZ/PZZ- und VSV-Sonderfälle haben erwartete Ergebnisse. Eintrittsbedingungen und Dauerüberwachung sind getrennt.

### Schritt 2 – Projektionen sicher auflösen und prüfen

- **Ziel:** Gültige Zugprojektionen vollständig abbilden und ungültige vor jeder Zustandsänderung abweisen.
- **Fachbereiche:** Rollen, Kardinalitäten, Sollwerte, Start-/Zielermittlung.
- **Dateien/Klassen:** `structs.h`, `estw/fs.h/.cpp`, `estw/estw.cpp`, `validate.cpp`, bei Bedarf `fahrstrasseneditor.cpp`; vorhandene Validierungs-/Fahrstraßentests.
- **Abhängigkeiten:** Strukturteil von Schritt 1; bereits vorab technisch vorbereitbar.
- **Prüfkriterien:** Alle erlaubten Rollen-/Elementkombinationen einschließlich Blind/Auflöser als Fahrweg; falsche Typen, unbekannte Rolle, zwei Vorwegsignale, doppelte Elemente, fehlende Referenzen und falsche Sollwerte werden abgewiesen. Null Fahrwegelemente bleiben zulässig. Mehrdeutige Auswahl bleibt ohne Seiteneffekt.

### Schritt 3 – Gemeinsame Elementzugriffe und Beitragsverwaltung

- **Ziel:** PZ-/Zielzugriff, Rollenfilter und genau einmal verwaltete Weichenbeiträge bereitstellen.
- **Fachbereiche:** Beanspruchungsgrenzen, Istlage, gemeinsamer Verschluss, eigene/fremde Beiträge.
- **Dateien/Klassen:** `Gleis`, `ZielElement`, `Fs`, `Estw`, gegebenenfalls interne Hilfstypen; `tests/rangierstrassen_tests.cpp` und neue Zugtests.
- **Abhängigkeiten:** Schritt 2, F2/F4/F5 für gemeinsame Schutzbeiträge.
- **Prüfkriterien:** Fahrweg-/Flanken-/Transportweichen werden gezielt getrennt; N Zyklen ergeben einen eigenen Zählerbeitrag; fremder Beitrag bleibt nach eigener Freigabe erhalten; Zähler unterschreitet 0 nicht. Rangierstatus, Fremdstränge und bisherige sichere Auflösung behalten ihr Verhalten.

### Schritt 4 – Zug-Zulassung isoliert ergänzen

- **Ziel:** Alle 16 Bedingungen einmalig und ohne Nebenwirkungen prüfen.
- **Fachbereiche:** PZS/PZ, entschiedene Zielzulassung, ZIF, Beanspruchung, Sperren, Weichen und aktive Flanken.
- **Dateien/Klassen:** `Fs`, `Estw::isActiveZugFlankenschutz()`, `Gleis`, `Signal`, `Blind`, `ZielElement`; neue Zugtestreihe.
- **Abhängigkeiten:** Schritte 1–3, insbesondere F2 und F11.
- **Prüfkriterien:** Jede Bedingung einzeln negativ und positiv; Rangier-/Zugbeanspruchung jeweils auf relevanten und unabhängigen Strängen; PZ nur an echten Gleisen einschließlich Transport; PZSP nur bei aktiver Zugflankenrolle; negative Prüfung verändert keinen Simulationszustand. Keine erneute ZP nach Beanspruchung.

### Schritt 5 – Zugbeanspruchungen und Fahrwegweichen

- **Ziel:** Status-1-Beanspruchungen sowie Stellen und Verschließen der Fahrwegweichen als testbare Teilfolge umsetzen.
- **Fachbereiche:** Start-A, Ziel-B, Blind/Auflöser A–B, Unterwegs A/B, Vorweg B, Weiche A+Sollstrang.
- **Dateien/Klassen:** `Fs`, `Signal`, `Weiche`, `Gleis`, `Blind`, `Aufloese`; neue Zugtests.
- **Abhängigkeiten:** Schritte 3–4, F11; noch keine Freigabe eines unvollständigen Gesamtpfads in der Oberfläche.
- **Prüfkriterien:** Erst nach tatsächlich vorhandenen BEA weiter; erst nach Istlage/Motorstillstand verschließen; passende Fremdsperre zulässig, Gegenlage/Auffahren abgewiesen; Flanke/Transport bleiben von Fahrwegaktionen unberührt; unverwendete Stränge unverändert.

### Schritt 6 – Signalvertrag und Flankenschutz

- **Ziel:** Flankenweichen und -signale kontrolliert sichern, Transportbedingungen prüfen und Unterwegssignale gemäß beschlossenem Ablauf stellen.
- **Fachbereiche:** Signal-Haltbindung, gemeinsame Schutzanforderungen, Konflikte, Unterwegs-Sollwerte, Flankenverschluss.
- **Dateien/Klassen:** `Signal`, `Weiche`, `Fs`, `Estw`; neue Signal-/Zugtests.
- **Abhängigkeiten:** Schritte 1–5; F3–F5, F8/F9/F12 entschieden.
- **Prüfkriterien:** Verzögerte Flankenweiche wird erst nach Istlage verschlossen; Signal-Halt wird tatsächlich erreicht und gegen unzulässige Fahrt gebunden; nicht herstellbarer Schutz folgt der vereinbarten Reaktion; beide Reihenfolgen konkurrierender Anforderung geprüft; Auflösung einer von mehreren erlaubten Schutzforderungen erhält die übrigen.

### Schritt 7 – Sicherungsniveaus, Fahrt, VSV und Überwachung

- **Ziel:** Vollständige Zugstatusfolge mit FÜM, Freiprüfung, Startfreigabe, VSV und definiertem Rückfall/Wiederanlauf.
- **Fachbereiche:** Zustandsmodell, Melder, Signalstellungen, Belegung und Geschwindigkeitsführung.
- **Dateien/Klassen:** `FsStatus` beziehungsweise interne typabhängige Zustände, `Fs::work()`, `Signal`, `ZielElement`; Zugtests.
- **Abhängigkeiten:** Schritte 1–6; F1/F6–F10 vollständig entschieden.
- **Prüfkriterien:** Jeder Übergang wartet auf seine wirklichen Voraussetzungen; Besetzung in jeder genannten Rolle verhindert das vereinbarte Niveau; kein Fahrtstatus allein nach abgewiesenem Signalauftrag; VSV-Quellen und Grenzwerte stimmen; eine Quelländerung wird zyklisch übernommen; Rückfall/Wiederkehr erzeugt keine neue ZP und keine doppelten Verschlüsse. Belegung nach Fahrt folgt dem beschlossenen Verhalten.

### Schritt 8 – Vollständige Auflösung und Simulationsreset

- **Ziel:** Aus jeder aktiven Zugphase sicher und vollständig zurückkehren.
- **Fachbereiche:** Haltbestätigung, Flankenschutzfreigabe, eigene Beanspruchungen, Melder und Restzustände.
- **Dateien/Klassen:** `Fs::requestAufloesung()`, `Fs::work()`, `Fs::reset()`, `Estw::reset()`, Signal-/Weichenklassen; Zug- und Auflösemeldertests.
- **Abhängigkeiten:** Schritte 1–7, vor allem F5/F7; produktive Aktivierung erst zusammen mit diesem vollständigen Rückweg freigeben.
- **Prüfkriterien:** Verzögertes tatsächliches Halt blockiert Freigabe; Auflösung während Einstellen und bei Überwachungsverlust funktioniert; wiederholter Auflöseklick baut Beiträge nicht doppelt ab; fremde Stränge/Schutzbeiträge bleiben; alle eigenen BEA, FÜM, ZIF und Schutzbindungen werden zum vereinbarten Zeitpunkt entfernt. Normalauflösung verändert keine fremden Besetzt-/Sperrzustände; Reset ist getrennt vollständig.

### Schritt 9 – Bedienung, Darstellung und gemischte Regression

- **Ziel:** Fertigen zentralen Zugablauf über die vorhandene Oberfläche verwenden und Anzeigen konsistent halten.
- **Fachbereiche:** Auswahl, Abweisung, Auflösen, FÜM/ZIF, Blindziel/Vorweg und Überschneidung mit Rangierstraßen.
- **Dateien/Klassen:** `LupeForm`, `ToolForm`, `SimulationSelection`, `LupeSignal`, `LupeBlind`, `LupeWeiche`, `LupeAufloese`, `CMakeLists.txt`, bestehende und neue Tests.
- **Abhängigkeiten:** Schritte 1–8.
- **Prüfkriterien:** Auswahl in beiden Reihenfolgen; Erfolg löscht und Ablehnung erhält Markierung; Mehrdeutigkeit beider Typen wird abgewiesen; Zustandsänderung wird nach dem mechanischen Schritt angezeigt. FÜM blinkt beziehungsweise leuchtet korrekt, Melder verschwinden nach letzter aktiver Zuordnung. Blindziel plus Vorweg lässt eine unabhängige Start-A-Nutzung bestehen. Rangier-ZP, Status 0–4, Wiederanlauf und Halt-vor-Auflösung bleiben erhalten. Echte aktive Zugflanken werden nun gemeinsam mit Rangierstraßen geprüft.

## 10. Konkreter Vorschlag für einen späteren Implementierungsauftrag

Der folgende Auftrag ist erst nach dokumentierter Beantwortung der Fragen F1–F12 vollständig ausführbar. Er setzt diese Antworten voraus und ersetzt sie nicht:

> Implementiere im Projekt `estw3` die Zugstraßenmechanik gemäß der anschließend konsolidierten `zugstrasse.md` und der daraus abgenommenen Übergangs-, Rollen- und Freigabetabelle. Verwende die vorhandene zentrale Auswahl und Laufzeitauflösung in `Estw`/`Fs`. Erhalte das spezifizierte Verhalten der Rangierstraßen.
>
> Vervollständige vor einer Zugaktivierung die Laufzeit-Strukturprüfung und implementiere die einmalige Zug-ZP ohne Nebenwirkungen bei Ablehnung. Ergänze den Zugriff auf vorhandene Programmfälle und die rollenbezogene Beanspruchung einschließlich Unterwegs A/B und Vorweg B. Setze Fahrweg- und Flankenweichen zu den festgelegten Zeitpunkten, warte jeweils auf die tatsächliche Sollstellung und verwalte genau die eigenen Beiträge im gemeinsamen Weichenverschlusszähler.
>
> Implementiere den festgelegten Signalvertrag einschließlich Unterwegssignalen, Signalflankenschutz, Wiederholungssperren, Ziel-ZIF, FÜM und VSV. Die zyklische Überwachung muss die beschlossenen Rückfälle und Wiederanläufe ausführen, ohne ZP oder Verschlussbeiträge zyklisch neu aufzubauen. Eine nicht angenommene Signalstellung darf keinen bestätigten Fahrtzustand erzeugen.
>
> Implementiere Auflösung aus jeder zulässigen aktiven Phase mit nachgewiesenem Halt vor Freigabe, der vereinbarten Flankenschutzreihenfolge und vollständiger Rücknahme ausschließlich eigener Zustände. Erhalte fremde Signalstränge, Weichenverschlussbeiträge und weiterhin aktive Schutzanforderungen. Trenne normale Auflösung von vollständigem Simulationsreset.
>
> Binde den fertigen Ablauf an Verarbeiten und Auflöseklick an; führe keine parallele Fachmechanik in `LupeForm` oder `ToolForm` ein. Prüfe Zuganzeigen, Blindziel mit Vorweg, Signalziel, unvollständige Projektionen, gemeinsame Schutzfälle und die Wechselwirkungen mit Rangierstraßen. Arbeite in den kleinen Schritten dieses Berichts und ergänze passende Regressionstests. Benötigte Änderungen öffentlicher Schnittstellen sind im Umfang dieses Implementierungsauftrags ausdrücklich festzulegen. Ein Build bleibt gemäß Projektvorgabe dem Benutzer vorbehalten, solange er ihn nicht ausdrücklich gesondert beauftragt.

Als sofort ausführbarer, kleinerer Vorauftrag eignen sich ausschließlich die fachlich unabhängigen Teile von Schritten 2 und 3: sichere Rollenauflösung, Abgleich der Laufzeit-Strukturprüfung mit bestehenden Regeln sowie lesender PZ-Zugriff und technische Beitragsverwaltung unter Erhalt der Rangierlogik. Dieser Vorauftrag würde Zugstraßen noch nicht produktiv aktivieren.

## 11. Abschlussprüfung dieser Analyse

- Alle geforderten Funktionsbereiche und die zehn vorgegebenen Hauptabschnitte sind enthalten; Fundstellen verknüpfen die technischen Bewertungen mit dem aktuellen Code.
- Die historische Analyse wurde vollständig abgeglichen. Überholte Status-5-, FÜM- und Freiprüfungsannahmen sowie erledigte Istlagen-/Verschlussgrundlagen sind ausdrücklich gekennzeichnet.
- Fachliche Blocker sind als konkrete Fragen formuliert. Empfehlungen, bereits geltende Regeln und nur optional benötigte Modellverfeinerungen sind getrennt.
- Die Implementierungsfolge nennt pro Schritt Ziel, Fachbereiche, Klassen/Dateien, Abhängigkeiten und Prüfkriterien. Der spätere Auftrag setzt fachliche Entscheidungen ausdrücklich voraus.
- Für diesen Auftrag wurde ausschließlich `.agents/zugstrasse_bericht2.md` erstellt. C++-Dateien und bestehende Dokumentation wurden nicht geändert. Die bereits zu Beginn vorhandene Änderung an `../../tools/win32form/win32form.cpp` gehört nicht zu diesem Auftrag und wurde nicht bearbeitet.
- Es wurden keine Builds oder Tests gestartet. Die technische Einschätzung beruht auf dem untersuchten Quelltext; eine spätere Implementierung muss die beschriebenen Prüffälle tatsächlich validieren.
