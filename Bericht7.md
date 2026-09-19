# Analyse der Rangierstraßenlogik in `estw3`

Stand der Analyse: 19. September 2026  
Untersuchungsgegenstand: aktueller Arbeitsstand des Projekts `estw3`

## 1. Auftrag, Abgrenzung und Bewertungsmethode

Dieser Bericht untersucht, ob die Rangierstraßenlogik anhand der derzeitigen Dokumentation und des vorhandenen Codes vollständig und fachlich konsistent implementiert werden kann. Er ist als fachlich-technische Planungsgrundlage für eine spätere Umsetzung gedacht.

Für die Aussagen gelten folgende Kennzeichnungen:

- **Belegt:** unmittelbar durch eine Dokumentations- oder Codefundstelle nachweisbar.
- **Schlussfolgerung:** aus mehreren Fundstellen technisch oder logisch abgeleitet, aber nicht selbst als Fachregel dokumentiert.
- **Offen:** durch die untersuchten Quellen nicht eindeutig beantwortet.
- **Annahme:** mögliche Auslegung, die vor einer Implementierung fachlich bestätigt werden müsste. Annahmen werden in diesem Bericht nicht als verbindliche Regeln verwendet.

Die Dokumentation der Rangierstraße wird als primäre fachliche Grundlage behandelt. Code wird nicht allein deshalb als fachlich richtig gewertet, weil eine Funktion oder ein Zustand bereits vorhanden ist. Die Zugstraßendokumentation wird nur zur Erkennung gemeinsamer Grundlagen und möglicher späterer Konflikte herangezogen; ihre ausdrücklich offenen Regeln werden nicht ergänzt oder vorweggenommen.

## 2. Gesamturteil

**Die Rangierstraßenlogik kann mit den derzeitigen Informationen nur teilweise und nicht als belastbare, vollständige Fachfunktion implementiert werden. Vor der vollständigen Implementierung müssen mehrere fachliche Regeln und technische Zustandsgrundlagen geklärt beziehungsweise ergänzt werden.**

Die positive Seite: Auswahl von Start und Ziel, strukturelle Rollen, zehn Zulassungsbedingungen, die grobe Zustandsfolge 0 bis 4, Weichen-Iststellung, Signalbegriffe, ein Verschlusszähler sowie die Reihenfolge der Auflösung sind weitgehend beschrieben. Persistente Daten für Fahrstraßentyp, Rolle und `SollStellung` existieren ebenfalls.

Die entscheidenden Hindernisse sind:

1. Beanspruchungen enthalten nur die Art `Rangier` oder `Zug`, aber weder Besitzer noch Referenzzählung. Damit lassen sich „durch eine andere Fahrstraße“, konfliktfreie gemeinsame Nutzung und die gezielte Rücknahme der **eigenen** Beanspruchung nicht zuverlässig ausdrücken.
2. Die Laufzeitprojektion verwirft `FahrstrassenElement::SollStellung` und mehrere Rollen. Ohne diese Daten sind Zulassungsprüfung, gezieltes Weichenstellen, Signalstellung und symmetrische Auflösung nicht umsetzbar.
3. Der dokumentierte Zustandsautomat ist nur als Gerüst vorhanden. Der Code realisiert lediglich einen Teil der Zulassungsprüfung und den Übergang 0 → 1; die Zustände 1 bis 4 und eine mehrstufige Auflösung fehlen.
4. Mehrfach ausgeführte zyklische Aktionen sind nicht abgesichert. Insbesondere erhöht `Weiche::setVerschluss()` bei jedem Aufruf erneut.
5. Für Fahrwegsignale fehlen eindeutige Rollen und vollständige Regeln zur Weiß-/Dunkelschaltung, zur Rücknahme sowie zur Fehlerreaktion.
6. Nach Verlust einer zyklischen Voraussetzung ist zwar „Signale auf Halt“ dokumentiert, nicht aber der anschließende Status, die Wiederanlaufregel oder die Rückabwicklung eines teilweise hergestellten Zustands.
7. Mehrere fachlich relevante Punkte sind offen, darunter die Bedeutung von Belegung bei Rangierstraßen, die Nutzung des Projektierungsmerkmals `PR`, die Behandlung des ZIF bei Auflösung und das Verhalten bei dauerhaft ausbleibender Rückmeldung.
8. Die Zugstraßenregeln zu Zulassung und Flankenschutz sind ausdrücklich unvollständig. Eine zu eng auf Rangierstraßen zugeschnittene Besitz-, Phasen- oder Signalarchitektur würde voraussichtlich spätere Umbauten erzwingen.

**Technische Schlussfolgerung:** Eine spätere Umsetzung ist grundsätzlich möglich, erfordert aber nicht nur das Ausfüllen vorhandener `TODO`-Blöcke, sondern eine belastbare Laufzeitprojektion, besitzersichere Elementoperationen, einen expliziten Zustandsautomaten und definierte Fehler-/Auflösungszustände.

## 3. Untersuchte Quellen

### 3.1 Fach- und Projektdokumentation

| Datei | Bedeutung für die Analyse |
|---|---|
| `.agents/fahrstrasse.md` | Gemeinsame Statusbedeutung, ungeordnete Start-/Zielauswahl, einmalige Zulassungsprüfung, zyklische Prüfungen und Sicherheitsreaktion. |
| `.agents/rangierstrasse.md` | Primäre Fachquelle: Struktur, zehn Zulassungsbedingungen, Zustände 0 bis 4 und Auflösereihenfolge. |
| `.agents/zugstrasse.md` | Vergleichsbasis für gemeinsame Mechanik; dokumentiert zugleich die offenen Zugstraßen- und Flankenschutzregeln. |
| `.agents/signale.md` | Verbindliche Halt- und Fahrtstellungen; insbesondere `-2` und `-1` als Rangierfahrtstellungen. |
| `.agents/weiche.md` | Bedeutung und gemeinsame Nutzung des nichtnegativen Verschlusszählers. |
| `.agents/lupeform.md` | Bedienregeln für Auswahl, Anforderung, Abweisung und Auflöseelement. |
| `.agents/datenstrukturen.md` | Projektierungsmerkmale (`PR`, `PRS`, `PRZ`, `PSSL`, `PSSR` usw.), Rollen und Bedeutung von `SollStellung`. |
| `.agents/rules_isvalid_estwdata.md` | Verbindliche strukturelle Konsistenzregeln je Fahrstraßentyp. |
| `.agents/estw.md` | Gesamtmodell, Laden in Laufzeitobjekte und Simulationszyklus; enthält eine zum aktuellen Code abweichende Beschreibung von `Fs::work()`. |
| `.agents/fahrstrasseneditmodus.md` | Erfassung von Fahrstraßenrollen und Sollstellungen im Editor. |
| `.agents/lademechanismus_daten.md` | Kontext der Datenquellen und des Ladeablaufs. |

Es existieren keine eigenständigen, als verbindlich erkennbaren Fachdokumente für Gleis-/Abschnittslogik, Auflöseelementlogik oder Flankenschutzmechanik. Die hierzu verfügbaren Aussagen verteilen sich auf die oben genannten allgemeinen Dokumente und den Code. Das ist selbst eine Dokumentationslücke.

### 3.2 Datenmodell, Laufzeit und Bedienung

| Datei | Bedeutung für die Analyse |
|---|---|
| `structs.h:41-93` | Persistente Strukturen `Element`, `FahrstrassenElement` und `Fahrstrasse`; enthält Projektierungsmerkmale, Rolle und `SollStellung`. |
| `estw/fs.h:12-88`, `estw/fs.cpp:19-185` | Fahrstraßen-Laufzeitobjekt `Fs`, Zielabstraktion, aktueller Statusansatz, Auswahl, Zulassung und Auflösung. |
| `estw/estw.h`, `estw/estw.cpp:14-85` | Zentraler Simulationszyklus, eindeutige Auswahl sowie Weiterleitung von Anschalt- und Auflöseanforderungen. |
| `estw/element.h:10-66`, `estw/element.cpp:14-44` | Gemeinsame Zustände für Belegung, Befahrbarkeitssperre und Beanspruchungsart. |
| `estw/gleis.h`, `estw/gleis.cpp:23-39` | Einfache, überschreibbare Gleisbeanspruchung ohne Besitzer. |
| `estw/weiche.h:8-87`, `estw/weiche.cpp:40-87,146-198` | Weichenstellung, Motorlauf, Sperren, Beanspruchungen B/C und Verschlusszähler. |
| `estw/signal.h:7-78`, `estw/signal.cpp:35-93,160-227,278-345` | Projektierungsmerkmale, Signalbegriffe, Strangbeanspruchungen, Signalsperre und ZIF. |
| `estw/blind.h`, `estw/blind.cpp:23-63` | `PRZ`/`PZZ`, Gleisbeanspruchung und ZIF eines Blindziels. |
| `estw/aufloese.h`, `estw/aufloese.cpp:19-35` | Auflöseelement als Gleis mit Anzeige `FAUF`; keine eigene Auflöseablauflogik. |
| `lupeform.cpp:804-949,977-1021,1181-1193` | Simulationsbedienung und Umwandlung persistenter Projektierung in Laufzeitobjekte. |
| `validate.cpp:320-530` | Allgemeine und typabhängige Strukturvalidierung. |
| `fahrstrasseneditor.cpp:90-105` | Erfassung einer Weichen-Sollstellung und Erzeugung von Fahrstraßenelementen. |

### 3.3 Laden, Speichern und Tests

| Datei | Bedeutung für die Analyse |
|---|---|
| `aktionen.cpp:450-512,618-699,1187-1301` | CSV-/MariaDB-Lesen, Aufbau von `FullProjekt` und Speichern von Rolle und `SollStellung`. |
| `mrdbstore.cpp:107-109,474-481,561-589` | mrdb-Abbildung und Speicherung der Fahrstraßenelemente. |
| `fahrstrassenpersistence.h` | Schnittstelle für persistente Fahrstraßenänderungen. |
| `tests/aufloese_melder_tests.cpp:13-135` | Tests für Auswahlreihenfolge, Mehrdeutigkeit, `an`-Zustand, Auflöseweiterleitung und Melder. |
| `tests/fahrstrassen_tests.cpp:46-231` | Editor-, Validierungs- und Persistenztests; keine Prüfung der Zustände 1 bis 4. |
| `CMakeLists.txt` | Vorhandene Testziele; kein eigenständiger fachlicher Zustandsautomat-Test. |

Der vorhandene unversionierte Bericht `.agents/bericht6.md` wurde nicht als normative Quelle verwendet. Maßgeblich für dieses Ergebnis sind die Fachdateien, Datenstrukturen und der aktuelle Code selbst.

## 4. Dokumentierter Lebenszyklus einer Rangierstraße

### 4.1 Struktur und Auswahl

Eine Rangierstraße besitzt beliebig viele Fahrwegelemente, genau ein Startsignal, genau ein Zielsignal oder Blindziel und genau ein Auflöseelement. Die Rollen 5 bis 8 sind für eine Rangierstraße verboten (`.agents/rangierstrasse.md:11-32`). Start- und Zielelement dürfen in beliebiger Reihenfolge ausgewählt werden. Genau eine Fahrstraße muss zu der ungeordneten Kombination passen; bei keiner oder mehreren passenden Fahrstraßen erfolgt keine willkürliche Auswahl (`.agents/fahrstrasse.md:18-24`, `.agents/lupeform.md:17-30`).

### 4.2 Einmalige Zulassungsprüfung

Die Zulassungsprüfung wird einmal bei der Wahl in Status 0 ausgeführt und nach dem erfolgreichen Übergang nicht zyklisch wiederholt (`.agents/fahrstrasse.md:20-24,36-39`). Für die Rangierstraße müssen gleichzeitig gelten (`.agents/rangierstrasse.md:36-49`):

1. Startsignal hat `PRS = 1`.
2. Zielsignal oder Blindziel hat `PRZ = 1`.
3. Kein Fahrwegelement ist durch eine andere Zug- oder Rangierstraße beansprucht.
4. Keine Weiche besitzt eine der benötigten `SollStellung` widersprechende Sperrlage.
5. Keine Weiche ist in einer der benötigten `SollStellung` widersprechenden Lage verschlossen.
6. Strang A des Startsignals ist nicht anderweitig beansprucht.
7. Das Startsignal zeigt Halt.
8. Am Startsignal ist keine Signalsperre aktiv.
9. Bei einem Zielsignal ist dessen Strang B nicht anderweitig beansprucht.
10. An keinem Fahrstraßenelement ist eine Befahrbarkeitssperre aktiv.

Nur bei positivem Gesamtergebnis darf die Fahrstraße Status 0 verlassen.

### 4.3 Einstellfolge

1. **Status 1 – Beanspruchungen anfordern:** Alle Fahrwegelemente erhalten eine Rangierbeanspruchung. Ein Signal als Fahrwegelement soll A und B erhalten, das Startsignal A, ein Zielsignal B und eine Weiche A plus den zur `SollStellung` gehörenden Zweig B oder C. Erst nach Rückmeldung aller geforderten Beanspruchungen darf der Ablauf fortfahren (`.agents/rangierstrasse.md:63-75`).
2. **Status 2 – Weichen stellen:** Nach bestätigten Beanspruchungen werden alle Weichen in ihre projektierte `SollStellung` gebracht (`.agents/rangierstrasse.md:77-81`). Der dokumentierte Ansatz trennt damit Befehl und tatsächliche Rückmeldung.
3. **Status 3 – Weichen verschließen:** Nur solange die Bedingungen aus Status 2 weiter gelten und alle Weichen ihre Ist-Soll-Gleichheit melden, werden sie verschlossen (`.agents/rangierstrasse.md:83-90`).
4. **Status 4 – eingestellt:** Erst nach bestätigtem Verschluss aller Weichen wird das Startsignal einmalig in die projektierte Rangierfahrtstellung gebracht und am Ziel ein Rangier-ZIF angezeigt (`.agents/rangierstrasse.md:92-106`; `.agents/fahrstrasse.md:36-39`).

Signale, die als Fahrwegelement projektiert sind, sollen je nach Funktion weißgeschaltet werden. Die Dokumentation nennt als mögliche Funktion „Signal der Gegenrichtung“ oder „Unterwegsignal“, spezifiziert aber weder die Unterscheidung in den Daten noch Zeitpunkt, konkreten Signalbegriff und Rücknahme (`.agents/rangierstrasse.md:69-70`).

### 4.4 Zyklische Überwachung und Verlust einer Voraussetzung

Voraussetzungen zum Erreichen beziehungsweise Beibehalten von Statuswerten größer 1 sind zyklisch zu prüfen. Die Zulassungsprüfung und das einmalige Setzen der Fahrtstellung beim Eintritt in Status 4 sind davon ausgenommen (`.agents/fahrstrasse.md:28-39`). Fällt eine zyklisch überwachte Voraussetzung weg, müssen Startsignal und vorhandene Unterwegssignale auf Halt gebracht werden (`.agents/fahrstrasse.md:41-50`).

**Offen:** Die rangierstraßenspezifische Dokumentation bestimmt danach weder Folgestatus noch Sperrzustand, Wiederanlauf, Rückabwicklung oder Bedienmeldung. Auch ist nicht eindeutig festgelegt, welche der zuvor hergestellten Zustände nach einem Fehler erhalten bleiben müssen.

### 4.5 Auflösung

Ein Klick auf das projektierte Auflöseelement startet die Auflösung. Die verbindliche Reihenfolge lautet (`.agents/rangierstrasse.md:108-120`):

1. Halt am Startsignal anfordern.
2. Tatsächliche Haltstellung abwarten.
3. Erst danach jeden eigenen Weichenverschluss genau einmal entfernen.
4. Danach die durch diese Fahrstraße gesetzten Beanspruchungen entfernen.
5. Abschließend Status 0 herstellen.

**Offen:** Eigene nummerierte Auflösestatus, die Rücknahme des ZIF, die Rücknahme der Weißschaltung, das Verhalten bei ausbleibender Halt-Rückmeldung und der Umgang mit einer nur teilweise eingestellten Fahrstraße sind nicht beschrieben.

## 5. Abgleich der fachlichen Anforderungen mit dem Code

Die Bewertung „vollständig vorbereitet“ bedeutet nur, dass die nötige Grundlage im aktuellen Code vorhanden ist; sie bedeutet nicht automatisch, dass der gesamte Ablauf bereits implementiert ist.

| Fachliche Anforderung | Dokumentationsquelle | Vorhandene Codegrundlage | Umsetzbarkeit | Fehlende Information oder Funktion |
|---|---|---|---|---|
| Rangierstraße als eigener Typ | `rangierstrasse.md:7-34`; `estw.md:111-119` | `Fahrstrasse::TypId` in `structs.h:85-93`; Validator akzeptiert 1/2 in `validate.cpp:328-340` | vollständig vorbereitet | `Fs` verzweigt im Ablauf nicht nach `TypId`; typsichere Repräsentation fehlt. |
| Genau ein Start-, Ziel- und Auflöseelement | `rangierstrasse.md:16-24` | Persistente Rollen; `validate.cpp:450-496`; Teilprüfung in `Fs::isValid()` | teilweise vorbereitet | `Fs::isValid()` prüft nicht alle Rangierverbote und Elementtypen. Laufzeitaufbau kann fehlende Rollen nur über Nullzeiger erkennen. |
| Rollen 5–8 bei Rangierstraße verboten | `rangierstrasse.md:25-32` | `validate.cpp:516-521` | mit überschaubarer Erweiterung möglich | `Fs::isValid()` prüft dies nicht; Laufzeitaufbau ignoriert Rollen 5, 7 und 8. |
| Start-/Zielauswahl unabhängig von Klickreihenfolge | `fahrstrasse.md:18-22`; `lupeform.md:17-26` | `Fs::matchesSelection()` in `estw/fs.cpp:46-60` | vollständig vorbereitet | Keine fachliche Lücke für die Zuordnung selbst. |
| Keine willkürliche Auswahl bei Mehrdeutigkeit | `fahrstrasse.md:22` | `Estw::requestFahrstrasse()` bricht beim zweiten Treffer ab (`estw/estw.cpp:61-69`) | vollständig vorbereitet | Projektvalidierung verhindert doppelte Start-/Zielkombinationen nicht; Abweisung erfolgt erst zur Laufzeit. |
| ZP genau einmal vor Status > 0 | `fahrstrasse.md:20-24,36-39` | `Fs::requestActivation()`, `Fs::work()` | widersprüchlich / derzeit nicht korrekt | `requestActivation()` setzt zuerst `an=true`; `work()` prüft `condition1()` bei jedem Zyklus in Status 0. Eine negative ZP bleibt dadurch als aktive, wiederholt prüfende Anforderung bestehen. |
| ZP 1: `PRS` am Start | `rangierstrasse.md:40` | `Signal::GetPRS()`; Flag wird in `lupeform.cpp:995-1003` geladen | mit überschaubarer Erweiterung möglich | `Fs::condition1()` prüft `PRS` nicht. |
| ZP 2: `PRZ` am Signal-/Blindziel | `rangierstrasse.md:41` | `Signal::GetPRZ()`, `Blind::getPRZ()`; Flags werden geladen | teilweise vorbereitet | `ZielElement` bietet keinen `PRZ`-Zugriff. `Fs::condition1()` prüft nur ZIF, nicht `PRZ`. |
| ZP 3: kein Fahrwegelement durch **andere** Fahrstraße beansprucht | `rangierstrasse.md:42` | `EstwElement::getBeansprucht()` und `BEA_t`; Teilprüfung in `Fs::condition1()` | derzeit nicht fachlich sicher implementierbar | Beanspruchung besitzt keine Fahrstraßen-ID/Ownership. „Eigene“ und „andere“ Beanspruchung sowie mehrere gleichartige Besitzer sind nicht unterscheidbar. |
| ZP 4: keine widersprechende Weichensperrlage | `rangierstrasse.md:43` | `WeicheInitData::PSSL/PSSR`, Getter, `FahrstrassenElement::SollStellung` | derzeit nicht implementierbar im Laufzeitmodell | `SollStellung` gelangt nicht in `FsInitData`; `lupeform.cpp:977-983` überträgt `PSSL/PSSR` nicht in `WeicheInitData`; Zuordnung Links/Rechts zur Fachbedeutung muss eindeutig bestätigt sein. |
| ZP 5: kein widersprechender Verschluss | `rangierstrasse.md:44`; `weiche.md:7-32` | `Weiche::getVerschluss()` und `getStellung()` | technisch und fachlich teilweise ungeklärt | Verschluss speichert nur einen Zähler, nicht die verschlossene Lage oder Besitzer. Die aktuelle Istlage kann als Lage dienen, doch Besitzer und konsistente Mehrfachnutzung fehlen. |
| ZP 6: Startsignal A frei von anderer Fahrstraße | `rangierstrasse.md:45` | `Signal::getBEAA()`; aktuelle Teilprüfung in `Fs::condition1()` | teilweise vorbereitet | Kein Besitzerbezug; Code lehnt jede Beanspruchung ab, kann „andere“ nicht erkennen. |
| ZP 7: Startsignal Halt | `rangierstrasse.md:46`; `signale.md:26-42` | `Signal::GetHSV()`, `HaGT()`, `SetHSV()` | mit überschaubarer Erweiterung möglich | `Fs::condition1()` prüft HSV nicht. Genaue Haltdefinition ist für HSV dokumentiert, sollte zentral typisiert werden. |
| ZP 8: keine Signalsperre | `rangierstrasse.md:47` | `Signal::GetFSS()` | mit überschaubarer Erweiterung möglich | In `Fs::condition1()` nicht geprüft. |
| ZP 9: Zielsignal B frei von anderer Fahrstraße | `rangierstrasse.md:48` | `Signal::getBEAB()` vorhanden | teilweise vorbereitet | `ZielElement` legt den konkreten Typ nach außen nicht ausreichend offen; Ownership fehlt. Blindziel darf diese Signalprüfung nicht erhalten. |
| ZP 10: keine Befahrbarkeitssperre an irgendeinem Fahrstraßenelement | `rangierstrasse.md:49` | `EstwElement::getBefahrbarkeitssperre()` | derzeit im `Fs`-Modell unvollständig | `Fs::condition1()` prüft sie nicht; `FsInitData` enthält nicht alle Rollen zuverlässig, sodass „jedes Fahrstraßenelement“ nicht vollständig iterierbar ist. |
| ZIF muss vor Anforderung frei sein | Aus aktuellem Code ableitbar, nicht Teil der zehn dokumentierten ZP-Bedingungen | `Fs::condition1()` prüft `zielElement->getZIF()` | fachlich ungeklärt | **Schlussfolgerung:** Code fügt eine nicht dokumentierte Zulassungsbedingung hinzu. Fachliche Bestätigung nötig. |
| Beanspruchungen in Status 1 setzen | `rangierstrasse.md:63-75` | Setter in `Gleis`, `Signal`, `Weiche`; `BEA_t::Rangier` | derzeit nicht sicher implementierbar | Keine transaktionale/atomare Reservierung, kein Besitzer, keine Rücknahme nur der eigenen Anforderung, keine Teilfehlerstrategie. |
| Signal als Fahrwegelement: A und B | `rangierstrasse.md:69-70` | `Signal::setBEAA()`/`setBEAB()` | teilweise vorbereitet | Rolle/Funktion Gegenrichtung vs. Unterwegsignal nicht repräsentiert; Weißschaltregel unvollständig. |
| Startsignal A, Zielsignal B | `rangierstrasse.md:71-72` | Signalsetter vorhanden | teilweise vorbereitet | Besitzerlos; beim Überschreiben fremder Werte kein Schutz. |
| Weiche A plus B/C entsprechend Sollstellung | `rangierstrasse.md:73` | `Weiche` hat nur `FBEAB`/`FBEAC` | derzeit nicht vollständig ausdrückbar | Ein Weichen-Strang A fehlt im Modell. `SollStellung` wird nicht in die Laufzeitprojektion übernommen. |
| Tatsächliche Beanspruchungen rückmelden, erst dann weiter | `rangierstrasse.md:75` | Getter der Elemente | teilweise vorbereitet | Keine vollständige, rollenspezifische Sollmenge; keine Ownership. `condition2()` ist nur `return true`. |
| Weichen gezielt in Sollstellung bringen | `rangierstrasse.md:77-81` | `Weiche::WU()`, Motorlauf und `getStellung()` | derzeit technisch unzureichend | `WU()` ist ein Umschalt-/Richtungswechselbefehl, kein idempotenter Befehl „fahre nach Links/Rechts“. `SollStellung` fehlt im Laufzeitobjekt. |
| Weichenlauf und Ist-Soll-Rückmeldung abwarten | `rangierstrasse.md:81-88` | `Is_Motorlauf()` und `getStellung()` | mit Erweiterung möglich | Zustand 2 fehlt; kein per Route gespeicherter Sollwert. Verhalten bei Endlagenfehler/Timeout offen. |
| Weichen in Status 3 genau einmal verschließen | `rangierstrasse.md:83-90`; `weiche.md:18-32` | `setVerschluss()` erhöht Zähler | derzeit nicht zyklussicher | Jeder erneute Aufruf erhöht erneut; es gibt kein fahrstraßenbezogenes „bereits verschlossen“ oder Besitz-Token. |
| Mehrfach genutzte Weiche erst nach letzter Freigabe frei | `weiche.md:20-32` | Gemeinsamer ganzzahliger Zähler | teilweise vorbereitet | Zähler allein kann gleiche Lage unterstützen, aber nicht Besitzer, Lagekonflikte oder doppelte Anforderung derselben Route verhindern. |
| Startsignal beim Eintritt in Status 4 einmal auf projektierte Fahrtstellung | `rangierstrasse.md:92-102`; `fahrstrasse.md:39`; `signale.md:33-42` | `FahrstrassenElement::SollStellung`; `Signal::SetHSV()` | teilweise vorbereitet | Start-Sollstellung wird im `FsInitData` verworfen. Status 4/Eintrittsaktion fehlt. Zulässigkeit von `-1` gegenüber `-2` je konkreter Projektierung muss validiert werden. |
| Rangier-ZIF am Ziel setzen | `rangierstrasse.md:102-104` | Zieltypen können ZIF speichern; `ZielElement::setZIF()` vorhanden | fehlerhaft vorbereitet | `ZielElement::setZIF(ZIF_t)` ignoriert den Parameter und setzt immer `None` (`estw/fs.cpp:178-185`). |
| Zyklische Statusvoraussetzungen > 1 prüfen | `fahrstrasse.md:28-32` | Zentraler `Estw::work()`-Zyklus | mit erheblicher Erweiterung möglich | `Fs::work()` implementiert die Zustände nicht. Zu prüfende Invarianten und Fehlerfolgestatus sind nur teilweise bestimmt. |
| Wegfall einer Voraussetzung: Start/Unterwegssignale Halt | `fahrstrasse.md:41-50` | `Signal::HaGT()` | teilweise vorbereitet | Rangierstraße verbietet Rolle 7, nennt aber Signale im Fahrweg als mögliche Unterwegssignale. Betroffene Signale und weiterer Status sind unklar. |
| Auflösung durch zugeordnetes Auflöseelement | `rangierstrasse.md:108-110`; `lupeform.md:30` | Klickweiterleitung und Rollenmatch in `lupeform.cpp:1189-1192`, `Estw::requestAufloesung()` | teilweise vorbereitet | Bedienung ist vorhanden, der fachliche Ablauf nicht. Alle passenden aktiven Routen werden verarbeitet; das entspricht der aktuellen Bedienungsdokumentation. |
| Zuerst Halt anfordern und tatsächliches Halt abwarten | `rangierstrasse.md:112-120` | `HaGT()` und `GetHSV()` | derzeit nicht umgesetzt | `Fs::requestAufloesung()` ruft sofort `setAn(false)` auf; keine Wartephase. |
| Danach eigenen Weichenverschluss einmal entfernen | `rangierstrasse.md:116` | `unSetVerschluss()` | derzeit nicht sicher umsetzbar | Keine Liste der tatsächlich durch diese Route gesetzten Verschlüsse; kein Besitzer; keine Teilauflösestatus. |
| Danach eigene Beanspruchungen entfernen | `rangierstrasse.md:117` | Setter können `None` setzen | derzeit nicht sicher umsetzbar | Besitzerloses Löschen könnte Ansprüche anderer Routen entfernen. Keine Aktionshistorie für teilweise eingestellte Routen. |
| Abschließend Status 0 | `rangierstrasse.md:118` | `status=0`, `an=false` | widersprüchlich | Code setzt sofort Status 0; Dokumentation verlangt Status 0 erst nach sicherer Auflösung. Bool `an` widerspricht der dokumentierten alleinigen Ableitung aus `status`. |
| Bedienabweisung/Diagnose | `lupeform.md:26` | `showSimulationMessage("abgewiesen")`; Element-Logging | teilweise vorbereitet | `requestActivation()` meldet trotz negativer `condition1()` Erfolg. Es fehlen strukturierte Ablehnungsgründe und Fehlerzustände. |

## 6. Zustandsmodell

### 6.1 Dokumentiertes Sollmodell

Der allgemeine Status ist laut `.agents/fahrstrasse.md:7-14` die einzige Wahrheitsquelle: 0 bedeutet ausgeschaltet, jeder Wert größer 0 eingeschaltet. Die Rangierstraße verwendet beim Einstellen die Statuswerte 0 bis 4 (`.agents/rangierstrasse.md:51-106`). Die Auflösung besitzt eine verbindliche Schrittreihenfolge, aber keine zugeordneten Statuswerte.

Damit einmalige Aktionen in einem zyklisch aufgerufenen System nicht mehrfach wirken, ist konzeptionell zwischen drei Dingen zu unterscheiden:

1. **Eintrittsaktion:** genau einmal beim Eintritt in eine Phase ausführen, etwa Beanspruchungen anfordern oder den Verschlusszähler erhöhen.
2. **Rückmelde-/Invariantenprüfung:** beliebig oft ohne Zustandsänderung ausführen, etwa „alle Beanspruchungen vorhanden“ oder „Weiche in Sollstellung“.
3. **Übergang:** erst nach positiver Prüfung atomar in die nächste Phase wechseln.

Diese Trennung ist aus der Dokumentation ableitbar, aber noch nicht technisch umgesetzt. Eine Eintrittsaktion darf nicht einfach in jedem `work()`-Aufruf des gleichen Status wiederholt werden.

### 6.2 Zustandsübergangstabelle

„Nicht festgelegt“ in der Tabelle bezeichnet eine echte Dokumentationslücke und keinen Implementierungsvorschlag.

| Ausgangszustand | Voraussetzung | Einmalige Aktion | Zyklische Prüfung | Folgezustand | Fehlerreaktion |
|---|---|---|---|---|---|
| Status 0, keine Anforderung | Gültige, eindeutige Start-/Zielkombination | Rangier-ZP genau einmal ausführen | keine Wiederholung der ZP | bei ZP positiv Status 1; sonst Status 0 | Keine fachlich detaillierte Meldung festgelegt; Bedienung kennt „abgewiesen“. |
| Eintritt Status 1 | ZP war positiv | Alle rollenspezifischen Rangierbeanspruchungen genau einmal anfordern | Prüfen, ob jede geforderte Beanspruchung tatsächlich und weiterhin vorliegt | Status 2, sobald vollständig | Teilanforderungsfehler, Rollback und Wartezeit nicht festgelegt. Bei Verlust nach Status > 1 Signale Halt. |
| Eintritt Status 2 | Alle Beanspruchungen bestätigt | Jede noch nicht richtig stehende Weiche gezielt in ihre Sollstellung anfordern | Beanspruchungen weiter prüfen; Motorlauf/Weichen-Iststellung prüfen | Status 3, sobald alle Weichen Sollstellung erreicht haben | Signale auf Halt bei weggefallener zyklischer Voraussetzung; weiterer Status nicht festgelegt. |
| Eintritt Status 3 | Beanspruchungen vorhanden; alle Weichen in Sollstellung | Eigenen Verschluss jeder Weiche genau einmal hinzufügen | Beanspruchungen, Ist-Soll-Gleichheit und vollständigen Verschluss prüfen | Status 4, sobald alle Weichen verschlossen | Signale Halt; Behandlung teilweise gesetzter Verschlüsse nicht festgelegt. |
| Eintritt Status 4 | Bedingungen aus Status 3 gelten; Verschlüsse bestätigt | Startsignal genau einmal auf projektierte Rangierfahrtstellung; Ziel-ZIF setzen; dokumentierte Fahrwegsignale entsprechend Funktion schalten | Für Status 4 erforderliche Bedingungen zyklisch beibehalten | Status 4 | Start und Unterwegssignale Halt; Status-/Wiederanlauf-/Rückabwicklungsregel nicht festgelegt. |
| Status 1–4, Auflösung angefordert | Zugeordnetes Auflöseelement bedient; aktueller Zustand lässt Auflösung zu | Startsignal Halt anfordern | Tatsächliche Haltstellung prüfen | Auflöseschritt „Freigabe“ nach bestätigtem Halt | Verhalten bei niemals erreichter Haltstellung nicht festgelegt. |
| Auflösung, Halt bestätigt | Startsignal tatsächlich Halt | Jeden von dieser Route gesetzten Weichenverschluss genau einmal entfernen | Prüfen, ob die eigene Verschlussanforderung entfernt ist | nächster Auflöseschritt | Teilfehlerbehandlung nicht festgelegt. |
| Auflösung, Verschlüsse entfernt | Eigene Verschlüsse sicher entfernt | Eigene Beanspruchungen entfernen | Prüfen, ob die eigenen Ansprüche entfernt sind | Status 0 | Teilfehlerbehandlung nicht festgelegt. |
| Übergang nach Status 0 | Freigaben vollständig | Nicht mehr benötigte Anzeigen/Signalzustände zurücknehmen | keine | Status 0 | Rücknahme von ZIF und Weißschaltungen nicht dokumentiert. |

### 6.3 Tatsächlicher Codezustand

`Fs` besitzt parallel `bool an` und `int status` (`estw/fs.h:80-87`). Das widerspricht der dokumentierten Regel, dass Ein/Aus ausschließlich aus dem Status abgeleitet wird. Der Status ist privat und besitzt keinen diagnostischen Getter.

`Fs::requestActivation()` setzt `an` vor der Zulassungsprüfung und ruft einmal `work()` auf (`estw/fs.cpp:66-71`). Bleibt die Teilprüfung negativ, bleibt `an=true`, `status=0`, und jeder spätere Zyklus versucht sie erneut (`estw/fs.cpp:90-102`). Das ist sowohl eine doppelte Wahrheitsquelle als auch ein Verstoß gegen die einmalige ZP.

Nur der Übergang 0 → 1 ist ausführbar. Die vorgesehenen Teile für Status 1 bis 3 sind auskommentiert; `condition2()` und `condition3()` liefern bedingungslos `true` (`estw/fs.cpp:104-123,155-162`). Status 4 existiert nicht als funktionsfähiger Ablauf.

Die Auflösung setzt unmittelbar Status 0, fordert Halt an und setzt anschließend `an=false` (`estw/fs.cpp:73-89`). Sie wartet nicht auf die tatsächliche Haltstellung und entfernt weder Verschlüsse noch Beanspruchungen. Der aktuelle Code erfüllt daher gerade die dokumentierte sicherheitsrelevante Reihenfolge nicht.

### 6.4 Wiederholbarkeit und Idempotenz

Die aktuellen Setter sind überwiegend einfache Zuweisungen. Das macht eine Wiederholung nicht automatisch fachlich sicher:

- `Gleis::setBEA()` überschreibt ohne Besitzer- oder Konfliktprüfung (`estw/gleis.cpp:27-34`).
- `Signal::setBEAA()` setzt bei jeder Änderung das Signal auf Halt, überschreibt danach aber ebenfalls den Besitzer-losen Typ (`estw/signal.cpp:172-190`).
- `Weiche::setBEAB()` und `setBEAC()` überschreiben vorhandene Werte (`estw/weiche.cpp:47-62`).
- `Weiche::setVerschluss()` erhöht bei jedem Aufruf (`estw/weiche.cpp:73-86`). Eine zyklische Wiederholung produziert deshalb einen falschen Zähler.
- `Weiche::unSetVerschluss()` klemmt negative Werte auf 0. Dadurch wird eine zu häufige Freigabe verdeckt statt einem Besitzerfehler zugeordnet.

Für eine belastbare spätere Umsetzung muss der Automat deshalb nachweisbar festhalten, welche Ressource durch genau diese Fahrstraße bereits angefordert, bestätigt, verschlossen und wieder freigegeben wurde. Nur der numerische Phasenstatus reicht bei Teilfortschritt mehrerer Elemente nicht immer aus.

## 7. Fahrstraßenelemente, Rollen und ausdrückbare Eigenschaften

### 7.1 Persistentes Modell

`FahrstrassenElement` speichert Fahrstraße, Element, Rolle und `SollStellung` (`structs.h:74-82`). `Fahrstrasse` speichert zusätzlich den Fahrstraßentyp (`structs.h:85-93`). Laden und Speichern erhalten diese Angaben; unter anderem lesen CSV und MariaDB `SollStellung` (`aktionen.cpp:506-509,657-660`), und die MariaDB-Schreiblogik schreibt sie wieder (`aktionen.cpp:1278-1291`). Die persistente Grundstruktur ist daher grundsätzlich geeignet, eine Weichen- oder Startsignal-Sollstellung zu transportieren.

### 7.2 Rollenbewertung

| Benötigte Rolle/Eigenschaft | Im Datenmodell ausdrückbar? | Bewertung |
|---|---:|---|
| Startsignal | ja, Rolle 2 | Eindeutig; genau eines vorgeschrieben. |
| Zielsignal oder Blindziel | ja, Rolle 3 | Eindeutig; genau eines vorgeschrieben. |
| Fahrwegelement | ja, Rolle 1 | Elementart ist über `Element::TypId` erkennbar. |
| Weiche mit Sollstellung | ja, Rolle 1 + `SollStellung` | Persistierbar, aber im Laufzeitaufbau derzeit verloren. |
| Startsignal mit Fahrtstellung | ja, Rolle 2 + `SollStellung` | Persistierbar, aber im Laufzeitaufbau derzeit verloren; Wertevalidierung fehlt. |
| Signal im Fahrweg | ja, Rolle 1 | Beanspruchung A/B ist dokumentiert. |
| Unterwegssignal einer Rangierstraße | nicht eindeutig | Rolle 7 ist für Rangierstraßen verboten, während Rolle-1-Signale laut Text Unterwegsignale sein können. Es fehlt ein eindeutiger Funktionsindikator. |
| Signal der Gegenrichtung | nicht eindeutig | Ebenfalls nur als Rolle-1-Signal vorhanden; keine gespeicherte Unterscheidung zum Unterwegsignal. |
| Dunkel-/Weißschaltfunktion | nein | Keine explizite Rolle/Eigenschaft und keine vollständige Fachregel. |
| Durchrutschweg | nicht dokumentiert | Für Rangierstraßen ist keine entsprechende Rolle oder Regel angegeben. Es darf nicht angenommen werden, dass er erforderlich oder irrelevant ist. |
| Flankenschutzelement/-transport | strukturell vorhanden, bei Rangierstraße verboten | Für Rangierstraßen muss der Validator solche Zuordnungen abweisen; keine Rangier-Ablauflogik vorsehen. |
| Auflöseelement | ja, Rolle 4 | Genau eines vorgeschrieben; Klickweiterleitung vorhanden. |
| Strang A/B/C je Element | nur indirekt | Aus Rolle, Elementart und Sollstellung ableitbar; bei Weichen fehlt im Laufzeittyp Strang A. |
| Muss verschlossen/freigegeben werden | nur indirekt | Derzeit aus „Fahrwegelement ist Weiche“ abzuleiten; kein Aktionsprotokoll. |
| Besitzer einer Beanspruchung/eines Verschlusses | nein | Nur Beanspruchungsart beziehungsweise Zähler, keine Fahrstraßen-ID. |
| Teilfortschritt pro Element | nein | Weder persistent nötig noch aktuell als Laufzeitzustand vorhanden; für sichere Wiederholung jedoch erforderlich. |

### 7.3 Laufzeitprojektion

`FsInitData` hält Start, Ziel, Auflöseelement und mehrere untypisierte Elementvektoren (`estw/fs.h:39-54`). Beim Aufbau in `LupeForm::estwInit()` werden nur Rollen 1, 2, 3, 4 und 6 befüllt (`lupeform.cpp:884-944`). Das konkrete `FahrstrassenElement` und damit seine `SollStellung` werden nicht in den rollenspezifischen Laufzeitdaten behalten.

Diese Projektion reicht nicht aus, um später zu beantworten:

- welche Weiche welche Stellung benötigt,
- welcher Weichenzweig beansprucht werden muss,
- welche Signalstellung am Start projektiert ist,
- welche konkrete Anforderung durch diese Route bereits gesetzt wurde,
- welche Anforderung bei Teilauflösung zurückzunehmen ist,
- welche Fahrwegsignale welche Funktion besitzen.

**Schlussfolgerung:** Die persistenten Daten sind näher am fachlichen Bedarf als die derzeitige Laufzeitstruktur. Ein späterer Umbau sollte die projektierte Mitgliedschaft als typisierte, unveränderliche Laufzeitbeschreibung erhalten und nicht nur Elementzeiger nach Rolle kopieren.

## 8. Elementbezogene Voraussetzungen

| Elementtyp | Vorhandene Zustände/Operationen | Eignung für Rangierstraße | Kritische Lücken |
|---|---|---|---|
| Signal | `PRS`, `PRZ`, HSV/VSV, SS, WSP, ZS1, Beanspruchung A/B, ZIF, `HaGT()`, `SetHSV()` | Viele Einzelzustände vorhanden; Halt und Rangierfahrt sind darstellbar. | Kein Beanspruchungsbesitzer; Zielabstraktion zu schmal; Weiß-/Dunkelschaltregeln fehlen; `ZielElement::setZIF()` ist fehlerhaft; Start-Sollstellung fehlt in `Fs`. |
| Weiche | Iststellung, Motorlauf, PSSL/PSSR, Umstellsperre, Beanspruchung B/C, Verschlusszähler, `WU()` | Rückmeldung und mechanischer Lauf sind grundsätzlich simulierbar. | Strang A fehlt; kein gezielter idempotenter Stellbefehl; `SollStellung` fehlt in `Fs`; PSSL/PSSR werden beim Laufzeitaufbau nicht geladen; kein Besitzer/Lagebezug des Verschlusses. |
| Gleis/Gleisabschnitt | Belegung, Befahrbarkeitssperre, eine Beanspruchungsart | Minimaler Zustand vorhanden. | Kein Besitzer oder Mehrfachanspruch; `PR` wird beim Laufzeitaufbau nicht gehalten; fachliche Bedeutung von `besetzt` in der Rangier-ZP nicht bestimmt. |
| Blindziel | Gleiszustände, `PRZ`, ZIF mit eigener Zulässigkeitsprüfung | Ziel kann grundsätzlich abgebildet werden. | Ownership der Gleisbeanspruchung fehlt; ZIF-Ansteuerung über `ZielElement` setzt aktuell immer `None`. |
| Auflöseelement | Gleiszustände und Anzeige `FAUF`; Klick wird zentral ausgewertet | Als Bedienziel nutzbar. | Keine eigene Ablaufsteuerung; Anzeige folgt `Fs::getAn()` und damit aktuell dem separaten Bool statt dem dokumentierten Status. |
| Fahrwegsignal | Signaloperationen wie oben | A/B beanspruchbar. | Keine eindeutige Funktionsrolle und keine vollständige Schalt-/Rücknahmeregel. |

### 8.1 Projektierungsdaten gehen teilweise verloren

Signale und Blindziele erhalten ihre relevanten Zulassungsmerkmale beim Laufzeitaufbau (`lupeform.cpp:995-1021`). Bei Weichen werden dagegen nur ID und Name in `WeicheInitData` kopiert; `PSSL` und `PSSR` bleiben auf ihren Standardwerten `false` (`lupeform.cpp:977-983`, `estw/weiche.h:22-26`). Das macht Zulassungsbedingung 4 im aktuellen Simulationsobjekt nicht zuverlässig prüfbar.

Das in `.agents/datenstrukturen.md:54-60` beschriebene `PR` für Rangierfahrwege existiert in `Element`, wird aber weder in die `Gleis`-/Weichenlaufzeitdaten übertragen noch von der dokumentierten Rangier-ZP verlangt. Ob jedes Fahrwegelement `PR=1` benötigt, ist deshalb eine offene Fachfrage und darf nicht aus dem bloßen Vorhandensein des Felds erfunden werden.

### 8.2 Belegung

`EstwElement` stellt `besetzt` bereit (`estw/element.h:53-63`), und `Weiche::WU()` weist das Umstellen einer belegten Weiche ab (`estw/weiche.cpp:146-151`). Die zehn dokumentierten Zulassungsbedingungen nennen Belegung jedoch nicht. Das verlangte Testszenario „besetztes Fahrwegelement“ hat daher noch kein fachlich belegtes Soll-Ergebnis.

**Offen:** Ist eine besetzte Rangierstraße grundsätzlich abzuweisen, nur unter bestimmten Bedingungen zulässig, oder darf sie eingestellt werden, wobei lediglich Weichen nicht unter dem Fahrzeug umgestellt werden? Diese Frage blockiert einen vollständigen Abnahmetest.

### 8.3 Gemeinsame Nutzung und symmetrische Rücknahme

Der gemeinsame Weichenverschlusszähler ist fachlich ausdrücklich gewollt (`.agents/weiche.md:7-32`). Für korrekte Symmetrie muss jedoch jede Fahrstraße genau einmal erhöhen und genau ihre eigene Anforderung einmal entfernen. Der aktuelle Zähler kann nicht erkennen, welcher Teilnehmer ihn erhöht hat.

Noch kritischer sind Beanspruchungen: Ein einzelner `BEA_t`-Wert kann nur `None`, `Rangier` oder `Zug` speichern (`estw/element.h:10-15`). Zwei Fahrstraßen gleicher Art, eine Rangier- und eine Zugstraße oder eine fehlerhafte doppelte Setzung lassen sich nicht als getrennte Ansprüche darstellen. Das direkte Setzen auf `None` kann deshalb fremde Sicherung verlieren lassen.

## 9. Überschneidungen mit Zugstraßen

Die Zugstraßendokumentation bezeichnet ihre Zulassungsbedingungen als noch nicht abschließend festgelegt und lässt Position und Inhalt des Flankenschutzzustands offen (`.agents/zugstrasse.md:40-44,86-90,158-165`). Daraus folgt: Gemeinsame Mechanik kann vorbereitet werden, fachliche Zugregeln dürfen aber nicht aus der Rangierlogik extrapoliert werden.

| Funktionalität | Gemeinsam nutzbar | Rangierstraßenspezifisch | Zugstraßenspezifisch oder noch ungeklärt |
|---|---:|---:|---:|
| Ungeordnete Start-/Zielauswahl | ja | nein | nein |
| Eindeutige Routenauflösung aus der Auswahl | ja | nein | nein |
| Strukturprüfung von Start, Ziel, Auflöser | ja | nein | Typspezifische Zusatzrollen bleiben getrennt. |
| Einmalige Zulassungsprüfung als Ablaufkonzept | ja | Bedingungen 1–10 für Rangier | Zugbedingungen noch offen. |
| Zustandsautomat mit Eintrittsaktion und zyklischer Rückmeldung | ja | konkrete Phasen-/Signalaktionen | Zusätzlicher Flankenschutzzustand und mögliche weitere Phasen offen. |
| Besitzersichere Beanspruchung | ja | Beanspruchungsart `Rangier` | Beanspruchungsart `Zug`; zusätzliche Konfliktregeln offen. |
| Weichen-Sollstellung und Rückmeldung | ja | nein | nein, soweit bisher dokumentiert. |
| Weichenverschluss mit gemeinsamer Zählung | ja | nein | Flankenschutz nutzt denselben Zähler. |
| Flankenschutz | Infrastruktur möglicherweise gemeinsam | für Rangierstraße ausdrücklich verboten | Zugmechanik vollständig offen. |
| Signal auf Halt als Sicherheitsreaktion | ja | beteiligte Rangiersignale | Zug-Unterwegssignale/Vorsignal-/Flankenschutzdetails offen. |
| Startsignal in Fahrt | gemeinsamer Befehl/Rückmelde-Rahmen | HSV `-1`/`-2` nach Projektierung | positive Geschwindigkeit und VSV-Regeln. |
| Fahrwegsignale | Elementoperationen gemeinsam | Weißschaltung nach noch unvollständiger Rangierregel | Zug-Unterwegssignale und Kennlichtmechanik teilweise/offen. |
| Zielanzeige/ZIF | gemeinsame Zielabstraktion möglich | Rangier-ZIF als projektspezifische Besonderheit | Zug-ZIF-Regeln separat. |
| Auflösesequenz „Halt vor Freigabe“ | ja | konkrete eigenen Ressourcen | Zeitpunkt des Flankenschutzabbaus bei Zug offen. |
| Fehler-/Diagnosemodell | ja | rangierspezifische Gründe | zusätzliche Zuggründe offen. |
| Bedienmeldung | ja | nein | nein |
| Persistenz von Route, Rolle, Sollstellung | ja | Typ 1 | Typ 2 und Zusatzrollen. |

### 9.1 Konzeptioneller Architekturvorschlag

Der Vorschlag beschreibt Verantwortlichkeiten, keinen zu implementierenden Code:

1. **Unveränderliche Laufzeitprojektion:** Aus `Fahrstrasse` und allen `FahrstrassenElement`-Datensätzen wird einmal eine validierte, typisierte Routenbeschreibung erzeugt. Sie bewahrt Element, Rolle, Sollstellung und Fahrstraßentyp. Fehlerhafte Projektierung wird vor Aktivierung abgelehnt.
2. **Gemeinsamer Ablaufkern:** Eine einzige Status-/Phasenwahrheit steuert Anforderung, Eintrittsaktionen, zyklische Rückmeldung, Sicherheitsreaktion und Auflösung. `an` wird aus der Phase abgeleitet, nicht separat gespeichert.
3. **Typspezifische Regelstrategie:** Rangier- und Zugstraße liefern getrennte Zulassungsbedingungen, Phasenbeiträge und Signalaktionen. Der gemeinsame Kern kennt keine verstreuten `TypId == 1`-Sonderfälle.
4. **Besitzersichere Elementoperationen:** Beanspruchung und Verschluss werden mit stabiler Fahrstraßenidentität angefordert/freigegeben. Eine Anforderung derselben Route ist idempotent; fremde Ansprüche werden weder überschrieben noch gelöscht. Für Weichen muss zusätzlich die geforderte Lage konfliktfähig repräsentiert sein.
5. **Befehl und Rückmeldung trennen:** „Stellen anfordern“ und „Iststellung erreicht“ sowie „Signalbegriff anfordern“ und „tatsächlich erreicht“ sind getrennte Schritte. Das unterstützt den vorhandenen zyklischen Simulationsansatz.
6. **Explizite Eintrittsaktionen:** Jede irreversible oder zählende Aktion wird nur beim Phaseneintritt oder über eine idempotente Besitzoperation ausgelöst. Zyklisch werden ausschließlich Rückmeldungen und Invarianten geprüft.
7. **Explizite Auflösungsphasen:** Auflösung ist kein sofortiges Ausschalten, sondern ein sicherer Teil des Automaten: Halt anfordern → Halt bestätigen → eigene Verschlüsse lösen → eigene Beanspruchungen lösen → Anzeigen bereinigen → Status 0.
8. **Strukturierte Diagnose:** Zulassungs- und Laufzeitfehler liefern maschinenlesbaren Grund plus Anzeige-/Logtext. Dadurch werden Tests und Bedienmeldungen nicht von freien Logstrings abhängig.

Diese Aufteilung hält den gemeinsamen Sicherheits- und Lifecycle-Rahmen stabil, ohne noch unbekannte Zugstraßenregeln festzuschreiben.

## 10. Risiken der unvollständigen Zugstraßendokumentation

### 10.1 Bereits fachlich gesicherte gemeinsame Grundlagen

- Start-/Zielauswahl und Eindeutigkeitsprüfung können fahrstraßenartunabhängig behandelt werden.
- Fahrstraßen brauchen eine unveränderliche Projektionsbeschreibung und einen zyklischen Ablauf.
- Einmalige Befehle müssen von wiederholbaren Rückmeldeprüfungen getrennt sein.
- Beanspruchungen und Verschlüsse müssen besitzersicher und symmetrisch sein.
- Befehl und tatsächlicher Elementzustand müssen getrennt ausgewertet werden.
- Die Auflösung darf sichernde Ressourcen erst nach nachgewiesener Haltstellung freigeben.
- Persistenz von Fahrstraßentyp, Rolle und Sollstellung ist gemeinsam nutzbar.

### 10.2 Sicher rangierstraßenspezifisch

- Die zehn Bedingungen in `.agents/rangierstrasse.md:40-49` gelten ausdrücklich für Rangierstraßen.
- Rangierfahrtstellungen sind `-1` und `-2`; positive HSV-Werte sind für Rangierstraßen keine gültige Fahrtstellung (`.agents/signale.md:30-42`).
- Rollen 5 bis 8 sind in der Rangierprojektierung verboten.
- Status 4 setzt im Projekt `estw3` trotz des Hinweises auf das reale ESTW einen Rangier-ZIF.
- Flankenschutz ist für Rangierstraßen laut aktueller Struktur nicht herzustellen.

### 10.3 Wahrscheinlich gemeinsam, aber noch nicht abschließend festlegbar

Die groben Phasen „beanspruchen – Weichen stellen – verschließen – Signal freigeben – sicher auflösen“ stehen in beiden Typdokumenten. Ihre technischen Schnittstellen können gemeinsam sein. Nicht sicher ist jedoch, ob Zugstraßen zusätzliche Zwischenphasen, andere Überwachungsbedingungen, andere Besitzkonflikte oder abweichende Freigabereihenfolgen benötigen.

### 10.4 Entscheidungen, die zurückgestellt werden sollten

- Eine endgültige, lückenlose numerische Status-Enumeration für **alle** Fahrstraßenarten. Der Zug-Flankenschutzzustand hat bewusst noch keine Nummer.
- Eine feste Basisklassenmethode mit hart verdrahteter Folge 0–4, in die Flankenschutz später nur durch Sonderfälle eingeschoben werden könnte.
- Ein gemeinsamer Zulassungsprüfer mit einer großen Liste typabhängiger `if`-Abfragen.
- Eine Beanspruchung als einzelnes Typfeld ohne Besitzer; sie würde spätestens bei Wechselwirkungen von Rangier- und Zugstraßen scheitern.
- Die Annahme, jede Weiche habe nur einen unqualifizierten Verschlussgrund. Zug-Flankenschutz verwendet denselben Zähler, braucht aber getrennt nachvollziehbare Besitzer und Gründe.
- Eine Signalbehandlung, die nur ein Startsignal kennt. Zugstraßen enthalten Unterwegs-, Vorweg- und Flankenschutzsignale.
- Eine Auflösung, die generell alle Ressourcen gleichzeitig freigibt. Für Zugstraßen ist der Zeitpunkt der Flankenschutzfreigabe ausdrücklich offen.

### 10.5 Konkrete Sackgassenrisiken

| Vorschnelle Festlegung | Spätere Folge |
|---|---|
| Statuszahl direkt mit einer einzigen Aktion gleichsetzen | Zusätzliche Zugphasen erzwingen Umnummerierung oder verstreute Ausnahmen. |
| `BEA_t` als alleinige Besitzinformation beibehalten | Fremde Ansprüche können überschrieben/gelöscht werden; Rangier-Zug-Konflikte bleiben unprüfbar. |
| `FsInitData` nur als Listen roher Basisklassenzeiger erweitern | Jede Fachaktion benötigt Laufzeit-Typabfragen und verliert Rolle/Sollstellung. |
| `Weiche::WU()` als Fahrstraßen-Stellbefehl verwenden | Wiederholte Zyklen können Richtung wechseln, statt idempotent eine Sollstellung anzufordern. |
| Rangier-Fahrwegsignale nur anhand „Signal in Rolle 1“ gleich behandeln | Gegenrichtung, Unterwegsignal und spätere Zugfunktionen sind nicht trennbar. |
| Auflösung mit `an=false` gleichsetzen | Sichere mehrzyklische Halt-/Freigabefolge und Zug-Flankenschutzabbau sind nicht modellierbar. |

## 11. Priorisierte Lücken, Widersprüche und offene Fragen

Prioritäten:

- **P0 – blockiert vollständige Implementierung:** Vor Umsetzung fachlich oder strukturell klären.
- **P1 – vor Implementierung klären:** Ein Teil könnte beginnen, eine belastbare Gesamtlösung sollte die Antwort jedoch kennen.
- **P2 – während Umsetzung entscheidbar:** Technische Ausgestaltung innerhalb geklärter Regeln.
- **P3 – spätere Verbesserung:** Nicht für die Kernfunktion blockierend.

### 11.1 Fachliche Dokumentationslücken

| Prio | Lücke/offene Frage | Bedeutung | Konkrete Rückfrage |
|---|---|---|---|
| P0 | Fahrwegsignal ist entweder Gegenrichtung oder Unterwegsignal, aber Schaltregel und Unterscheidung fehlen. | Blockiert korrekte Weiß-/Dunkelschaltung und Haltreaktion. | Wie wird die Funktion eines Rolle-1-Signals eindeutig projektiert, welcher HSV/Sichtzustand wird in welcher Phase gesetzt und wann zurückgenommen? |
| P0 | Verhalten nach Verlust einer zyklischen Voraussetzung ist nach „Signale Halt“ offen. | Blockiert sicheren Fehlerzustand und Wiederanlauf. | Bleiben Beanspruchungen/Verschlüsse bestehen? In welchen Status wechselt die Route? Darf sie automatisch fortsetzen oder nur aufgelöst/neu angefordert werden? |
| P0 | Teilfehler in Status 1 oder 3 ist nicht geregelt. | Blockiert atomare oder kompensierende Ressourcenbehandlung. | Was geschieht, wenn einige, aber nicht alle Beanspruchungen beziehungsweise Verschlüsse gesetzt werden konnten? |
| P0 | Auflösung unvollständig spezifiziert. | ZIF und Fahrwegsignale könnten nach Status 0 fälschlich aktiv bleiben. | Wann und in welcher Reihenfolge werden ZIF, Weißschaltungen und weitere Anzeigen/Signalbegriffe zurückgenommen? |
| P0 | Belegung eines Fahrwegelements ist keine dokumentierte ZP-Bedingung. | Das geforderte Testszenario hat kein belegbares Ergebnis. | Darf eine Rangierstraße über besetzte Elemente eingestellt werden? Falls ja, unter welchen Bedingungen und mit welchen Weichenrestriktionen? |
| P1 | Bedeutung von `PR` gegenüber der ZP ist offen. | Projektierungsmerkmal existiert, wird aber nicht gefordert. | Muss jedes rangiermäßig befahrene Gleis/Fahrwegelement `PR=1` besitzen? Gilt dies nur für Gleise oder weitere Elementtypen? |
| P1 | Verhältnis von `PSSL`/`PSSR` zur dynamischen `umstellsperre` ist nicht vollständig beschrieben. | ZP 4 verlangt eine lagebezogene Sperrprüfung, im Code existieren sowohl Projektierungsmerkmale als auch eine laufzeitliche, nicht richtungsqualifizierte Sperre. | Welche Zustände bilden die fachliche „Sperrlage“ ab, und müssen Projektierungsmerkmal, aktuelle Lage und dynamische Umstellsperre gemeinsam ausgewertet werden? |
| P1 | Dauerhaft fehlende Rückmeldung/Timeout ist offen. | Route kann unbegrenzt in einem Zwischenzustand hängen. | Ist unbegrenztes Warten gewollt? Welche Bedienung, Diagnose und sichere Rücknahme gibt es bei Elementfehlern? |
| P1 | ZIF-Freiheit wird nur im Code geprüft. | Code und Fachliste divergieren. | Gehört „Ziel-ZIF ist frei“ verbindlich zur Rangier-ZP oder zu einer anderen Prüfung? |
| P1 | Mehrere Fahrstraßen an einem Auflöseelement werden gemeinsam angefordert. | Aktuelle Bedienungsdokumentation und Code stimmen überein, Detailfehlerfall fehlt. | Soll bei mehreren aktiven Routen jede unabhängig aufgelöst werden, auch wenn eine davon die Auflösung aktuell nicht beginnen kann? |
| P2 | Genaue Diagnose-/Abweisungstexte fehlen. | Fachfunktion möglich, aber schwer prüfbar/bedienbar. | Welche Ablehnungsgründe müssen Bediener getrennt erkennen können? |

### 11.2 Widersprüche innerhalb der Dokumentation

| Prio | Widerspruch | Fundstellen | Bewertung |
|---|---|---|---|
| P0 | Rangierstraße darf keine Rolle 7 „Unterwegselement“ haben, gleichzeitig kann ein Signal als Rolle-1-Fahrwegelement ein „Unterwegsignal“ sein; bei Fehlern sollen Unterwegssignale auf Halt. | `rangierstrasse.md:29-30,69-70`; `fahrstrasse.md:47-50` | Ohne separate Funktionskennzeichnung ist nicht eindeutig bestimmbar, welche Signale gemeint sind. |
| P1 | `estw.md` beschreibt `Fs::work()` als fortlaufende `isValid()`-Prüfung, die allgemeine Fachlogik fordert jedoch statusbezogene Invarianten und einmalige ZP. | `estw.md:191`; `fahrstrasse.md:24,28-39` | `estw.md` ist technisch veraltet oder zu ungenau und sollte vor Umsetzung berichtigt werden. |
| P1 | Die Rangierstruktur erlaubt auch Blindziel und Auflöseelement als Rolle-1-Fahrwegelement; deren konkrete Beanspruchung und eventuelle Sonderbehandlung werden im Ablauf nicht separat erklärt. | `rangierstrasse.md:13-15,69` | Eine generische Beanspruchung ist ableitbar, aber genaue Stränge/Anzeigeinteraktionen sind nicht vollständig beschrieben. |

### 11.3 Widersprüche zwischen Dokumentation und Code

| Prio | Widerspruch | Fundstellen | Auswirkung |
|---|---|---|---|
| P0 | Dokumentation: Status ist alleinige Wahrheit. Code: `bool an` plus `int status`. | `fahrstrasse.md:7-14`; `estw/fs.h:80-87` | Inkonsistente Zustände wie `an=true`, `status=0` sind möglich und treten bei negativer ZP auf. |
| P0 | Dokumentation: ZP genau einmal. Code: Wiederholung in jedem Zyklus bei Status 0. | `fahrstrasse.md:24,38`; `estw/fs.cpp:90-102` | Eine zunächst unzulässige Route kann später ohne neue Bedienung anlaufen. |
| P0 | Dokumentation: Auflösung wartet auf Halt. Code: sofort Status 0/`an=false`. | `rangierstrasse.md:112-120`; `estw/fs.cpp:73-89` | Sichernde Ressourcenfolge fehlt vollständig. |
| P0 | Dokumentation: Weiche A plus B/C. Code: nur B und C. | `rangierstrasse.md:73`; `estw/weiche.h:46-49,82-83` | Geforderter Zustand ist nicht vollständig repräsentierbar. |
| P0 | Dokumentation: Ziel-ZIF setzen. Code: Wrapper setzt unabhängig vom Argument `None`. | `rangierstrasse.md:102`; `estw/fs.cpp:178-185` | Status-4-Aktion kann nicht über die vorgesehene Abstraktion ausgeführt werden. |
| P1 | Dokumentation: alle Projektierungsmerkmale maßgeblich. Code lädt Weichen-PSSL/PSSR nicht. | `datenstrukturen.md:58-59`; `lupeform.cpp:977-983` | ZP 4 würde mit falschen Standardwerten prüfen. |
| P1 | Dokumentation: `SollStellung` steuert Weiche/Startsignal. Code verwirft sie beim Laufzeitaufbau. | `estw.md:151-160`; `lupeform.cpp:884-948` | Status 2 und 4 sind nicht korrekt implementierbar. |
| P1 | Rangier-Startstellung muss `-1` oder `-2` sein; der Editor validiert nur Weichenwerte `-1`/`1`, nicht die signal- und fahrstraßentypabhängige Startstellung. | `signale.md:30-42`; `fahrstrasseneditor.cpp:90-105` | Ungültige positive oder sonstige Signalwerte können gespeichert werden und erst im Laufzeitverhalten auffallen. |
| P1 | Dokumentation: Fahrstraßenrollen werden strukturell vollständig geprüft. Codevalidierung prüft für Rolle 1 und Rolle 8 nicht die verbindlichen Elementtypzuordnungen. | `rules_isvalid_estwdata.md:96-127`; `validate.cpp:482-514` | Ungültige Projektierung kann den Laufzeitaufbau erreichen. |
| P1 | Code-Reset sollte Grundstellung herstellen; `Weiche::reset()` leert B/C-Beanspruchungen nicht. | `estw/weiche.cpp:180-189` gegenüber anderen Element-Resets | Nach Reset können scheinbar beanspruchte Weichen verbleiben. Dies ist eine technische Beobachtung, keine neue Fachregel. |
| P2 | Legacy-Button schaltet hart codierte Route 100 direkt über `setAn()` um. | `lupeform.cpp:804-813` | Umgeht Auswahl- und Zulassungssemantik; für eine spätere Abnahme störend. |

### 11.4 Fehlende Datenstrukturen

| Prio | Fehlende Struktur | Warum erforderlich |
|---|---|---|
| P0 | Besitzer-/Tokenmodell für Beanspruchungen | „Andere Fahrstraße“, idempotente Anforderung und gezielte Freigabe sind sonst nicht bestimmbar. |
| P0 | Besitzersichere Verschlussanforderung mit benötigter Lage | Gemeinsamer Zähler allein schützt nicht vor Doppelzählung und Gegenlage. |
| P0 | Typisierte Laufzeitmitgliedschaft mit Rolle, Element, Sollstellung | Aktuelle `FsInitData`-Zeigerlisten verlieren entscheidende Projektierung. |
| P0 | Darstellbarer Weichen-Strang A oder fachlich bestätigte Alternativabbildung | Dokumentierte Beanspruchung kann sonst nicht vollständig geprüft werden. |
| P1 | Explizite Funktion von Fahrwegsignalen | Gegenrichtung und Unterwegsignal sind nicht trennbar. |
| P1 | Expliziter Auflöse-/Fehlerzustand und Teilfortschritt | Mehrzyklische, sichere Rücknahme und Wiederholschutz fehlen. |
| P1 | Strukturierter Prüf-/Fehlergrund | Für Diagnose und deterministische Tests. |
| P2 | Typsichere Enumerationen für Route, Rolle, Sollstellung und Phase | Verhindert Magic Numbers und ungültige Kombinationen; technische Qualitätsmaßnahme. |

### 11.5 Fehlende Zustände oder Operationen im Code

- **P0:** vollständige Rangier-ZP mit genau einmaliger Ausführung und negativem Rückgabewert.
- **P0:** atomare oder kompensierbare Beanspruchungsanforderung pro Route.
- **P0:** Status 1 bis 4 einschließlich bestätigter Übergänge.
- **P0:** gezieltes, idempotentes Weichenstellen.
- **P0:** einmaliger, besitzersicherer Weichenverschluss.
- **P0:** mehrstufige Auflösung mit Halt-Rückmeldung vor Freigabe.
- **P0:** sichere Rücknahme nur der eigenen Ansprüche/Verschlüsse.
- **P1:** zyklische Invariantenprüfung und definierter Fehlerfolgezustand.
- **P1:** korrekte Signal-/ZIF-Eintritts- und Rücknahmeaktionen.
- **P1:** vollständige Strukturvalidierung vor Erzeugung eines `Fs`.
- **P2:** Status-/Diagnosezugriff für Tests und Anzeige.

### 11.6 Noch ungeklärte Abgrenzung Rangier/Zug

- **P0 für gemeinsame Architektur:** Ownership, Verschlussgründe und Phasen dürfen nicht auf nur einen Fahrstraßentyp zugeschnitten werden.
- **P1:** Einfügepunkt, Voraussetzungen und Auflösung des Zug-Flankenschutzes bleiben offen.
- **P1:** Zug-ZP ist nicht definiert; ein gemeinsamer Prüfkatalog darf daher nur den Rahmen, nicht die Bedingungen vorgeben.
- **P1:** Signalrollen und signaltechnische Aktionen müssen erweiterbar bleiben.
- **P2:** Endgültige Phasennummern können später festgelegt werden, wenn der Ablauf semantische Phasen statt überall verteilter Integervergleiche nutzt.

### 11.7 Fehlende Voraussetzungen für Tests und Verifikation

- Kein beobachtbarer Fahrstraßenstatus und keine strukturierten Übergangs-/Ablehnungsgründe.
- Keine Möglichkeit, pro Route den Besitz einer Beanspruchung oder eines Verschlusses abzufragen.
- Keine kontrollierbare Schnittstelle für gezielte Weichenbefehle und verzögerte Rückmeldungen.
- Keine Tests für die zehn Rangier-ZP-Bedingungen.
- Keine Tests für Status 1 bis 4, Verlust einer Voraussetzung oder sichere Auflösung.
- Keine fachlich bestätigten Sollwerte für Belegung, Weiß-/Dunkelschaltung und Teilfehler.
- Vorhandene Tests prüfen Bedien-/Persistenzteile, nicht den sicherungslogischen Lebenszyklus.

## 12. Differenzierte Umsetzbarkeitsbewertung

### 12.1 Fachliche Umsetzbarkeit

**Nur teilweise gegeben.** Die Hauptfolge und viele Zulassungsbedingungen sind klar. Eine vollständige Umsetzung scheitert derzeit vor allem an unbestimmter Fahrwegsignalbehandlung, Fehler-/Wiederanlaufsemantik, Teilfehlern, Belegungsregel und unvollständiger Aufräumlogik.

### 12.2 Technische Umsetzbarkeit

**Grundsätzlich gegeben, mit erheblichen Ergänzungen.** Die Anwendung besitzt zyklische Verarbeitung, Elementobjekte und persistente Projektierung. Das aktuelle `Fs`-Gerüst und die Besitzer-losen Elementzustände reichen jedoch nicht. Die Änderungen wären fachliche Kernimplementierung, nicht bloß kleine Ergänzungen.

### 12.3 Architektur im Hinblick auf Zugstraßen

**Sinnvoll vorbereitbar, aber nicht abschließend festlegbar.** Ein gemeinsamer Lifecycle-Kern, typisierte Projektion, Besitzoperationen und getrennte Strategien sind bereits begründbar. Die genaue Zugphasenkette und Zugregeln müssen offen bleiben.

### 12.4 Testbarkeit

**Aktuell unzureichend.** Elementgetter existieren teilweise, aber Route, Besitz und Übergangsgründe sind nicht hinreichend beobachtbar. Nach Klärung der Fachfragen ist die Logik gut testbar, wenn Uhr-/Zyklusfortschritt, Elementrückmeldungen, Routenstatus und Besitzverhältnisse deterministisch steuer- und abfragbar werden.

## 13. Schrittweiser Plan für eine spätere Implementierung

Dieser Plan enthält keine Codeänderung, sondern ordnet die notwendigen Arbeiten und Abhängigkeiten.

### Schritt 1 – Fachliche Blocker entscheiden

- **Ziel:** Verbindliche Antworten auf alle P0-Fragen erhalten.
- **Betroffene Komponenten:** `rangierstrasse.md`, `fahrstrasse.md`, `signale.md`, gegebenenfalls neue Fachabschnitte für Gleis, Fahrwegsignal und Auflösung.
- **Abhängigkeiten:** Fachverantwortliche müssen Belegung, `PR`, Fahrwegsignalrollen, Verlustverhalten, Teilfehler und vollständige Rücknahme festlegen.
- **Risiken:** Eine nur implizite mündliche Entscheidung wäre später nicht reproduzierbar.
- **Überprüfbares Ergebnis:** Jede P0-Rückfrage aus Abschnitt 11 besitzt eine eindeutige, versionierte Fachantwort samt Beispielen und negativem Fall.

### Schritt 2 – Dokumentation konsolidieren

- **Ziel:** Einen widerspruchsfreien Sollablauf mit Rangier-ZP, Einstell-, Fehler- und Auflösephasen dokumentieren.
- **Betroffene Komponenten:** Allgemeine Fahrstraßen-, Rangierstraßen-, Signal-, Weichen-, Datenstruktur- und Bedienungsdokumentation.
- **Abhängigkeiten:** Schritt 1.
- **Risiken:** Rolle 7 und informeller Begriff „Unterwegsignal“ könnten weiterhin vermischt werden; `estw.md` könnte veralteten Codezustand beschreiben.
- **Überprüfbares Ergebnis:** Eine einzige Zustands-/Übergangstabelle deckt Normalweg, Abweisung, Voraussetzungsausfall, Teilfehler und Auflösung ab; jede Aktion ist als einmalig oder zyklisch markiert.

### Schritt 3 – Projektierungs- und Laufzeitdaten festlegen

- **Ziel:** Alle zur Ausführung benötigten Daten verlustfrei und eindeutig repräsentieren.
- **Betroffene Komponenten:** `Fahrstrasse`, `FahrstrassenElement`, Laufzeitprojektion anstelle beziehungsweise Weiterentwicklung von `FsInitData`, Rollen-/Typdefinitionen, Validator.
- **Abhängigkeiten:** Geklärte Rolle von Fahrwegsignalen, Strängen und `SollStellung`.
- **Risiken:** Neue persistente Felder könnten Datenmigration erfordern; eventuell reicht eine eindeutig definierte vorhandene Rolle, was zuerst zu prüfen ist.
- **Überprüfbares Ergebnis:** Für jede fachliche Aktion kann aus einer validierten Route ohne Heuristik Element, Funktion, Sollwert, Beanspruchungszweig und spätere Freigabe ermittelt werden.

### Schritt 4 – Gemeinsame technische Grundlage schaffen

- **Ziel:** Besitzersichere, idempotente Elementanforderungen und eine einzige Fahrstraßen-Zustandswahrheit bereitstellen.
- **Betroffene Komponenten:** Beanspruchungsmodell der Elemente, Weichenverschluss, Fahrstraßenidentität, gemeinsamer Ablaufkern, Diagnoseergebnis.
- **Abhängigkeiten:** Schritt 3; gemeinsame Mindestanforderungen mit späteren Zugstraßen.
- **Risiken:** Veränderung elementarer Setter betrifft bestehende Bedien- und Testschnittstellen; Besitzer- und Referenzzählung müssen atomar konsistent bleiben.
- **Überprüfbares Ergebnis:** Zweimalige identische Anforderung derselben Route verändert keinen Zähler; Freigabe einer Route lässt fremde Ansprüche unverändert; widersprechende Weichenlagen werden deterministisch abgewiesen.

### Schritt 5 – Rangier-Zulassungsprüfung implementieren

- **Ziel:** Die verbindlichen, ergänzten Rangier-ZP-Bedingungen genau einmal und ohne Zustandsänderung prüfen.
- **Betroffene Komponenten:** Rangierregelstrategie, Zielabstraktion, Signal-/Weichen-/Elementabfragen, `requestFahrstrasse()`.
- **Abhängigkeiten:** Schritte 1 bis 4; korrekter Laufzeittransfer von PRS/PRZ/PSSL/PSSR/Sollstellung.
- **Risiken:** Seiteneffekte in Prüffunktionen würden die Momentaufnahme verfälschen; Code-interne Signalprüfungen dürfen der Fach-ZP nicht unbemerkt zusätzliche Regeln hinzufügen.
- **Überprüfbares Ergebnis:** Jede Bedingung ist separat testbar; negative ZP lässt die Route vollständig in Status 0 und läuft ohne neue Bedienung nicht später an.

### Schritt 6 – Zustandsautomat und zyklische Verarbeitung umsetzen

- **Ziel:** Status 1 bis 4 mit Eintrittsaktionen und zyklischen Rückmeldungen deterministisch abarbeiten.
- **Betroffene Komponenten:** `Fs`/künftiger gemeinsamer Ablaufkern, `Estw::work()`, Elementkommandos und Rückmeldungen.
- **Abhängigkeiten:** Besitzoperationen, gezielter Weichenstellbefehl, vollständige Laufzeitprojektion.
- **Risiken:** Wiederholte Zyklen dürfen keine Befehle umkehren und keine Zähler erhöhen; synchrone Rückmeldungen dürfen keine Phase überspringen, wenn Eintrittsaktionen beobachtbar sein müssen.
- **Überprüfbares Ergebnis:** Der Normalfall erreicht 0 → 1 → 2 → 3 → 4 nur bei bestätigten Voraussetzungen; beliebig viele zusätzliche Zyklen verändern in jeder stabilen Phase weder Besitz noch Zähler.

### Schritt 7 – Anschaltung und Signalbehandlung ergänzen

- **Ziel:** Start- und Fahrwegsignale sowie Ziel-ZIF ausschließlich nach den dokumentierten Regeln setzen und überwachen.
- **Betroffene Komponenten:** Signal-API, Zielabstraktion, Rangierstrategie, Fahrwegsignalrollen.
- **Abhängigkeiten:** Fachentscheidung zur Weiß-/Dunkelschaltung und gültige Start-`SollStellung`.
- **Risiken:** `SetHSV()` enthält eigene Bedingungen, die mit dem Fahrstraßenautomaten abgestimmt werden müssen; fehlgeschlagene Signalanforderung braucht definiertes Verhalten.
- **Überprüfbares Ergebnis:** Signalbegriffe und ZIF entsprechen in jeder Phase dem Soll; Fahrt wird nur einmal beim Eintritt in Status 4 angefordert; Fehler erzwingen die dokumentierte Haltreaktion.

### Schritt 8 – Sichere Auflösemechanik umsetzen

- **Ziel:** Auflösung als mehrzyklischen, besitzersicheren Ablauf realisieren.
- **Betroffene Komponenten:** Auflöseanforderung, Routenautomat, Signalrückmeldung, Weichenverschluss, Beanspruchungen, Anzeigen.
- **Abhängigkeiten:** Schritte 4, 6 und 7; vollständige Fachreihenfolge aus Schritt 1.
- **Risiken:** Teilweise eingestellte Routen besitzen eventuell nur eine Teilmenge von Ressourcen; eine pauschale Freigabe wäre falsch.
- **Überprüfbares Ergebnis:** Vor nachgewiesenem Halt wird keine sichernde Ressource freigegeben; jede eigene Ressource wird exakt einmal entfernt; Status 0 wird erst nach vollständiger Bereinigung erreicht.

### Schritt 9 – Fehlerbehandlung und Diagnose ergänzen

- **Ziel:** Ablehnung, Wartezustand, technischer Fehler und Verlust einer Voraussetzung unterscheidbar machen.
- **Betroffene Komponenten:** Regelprüfer, Automat, Logging, Bedienmeldung im `LupeForm`.
- **Abhängigkeiten:** Dokumentierte Fehlerfolgen und strukturierte Ergebniswerte.
- **Risiken:** Freie Textmeldungen allein sind nicht stabil testbar; Diagnose darf keine sichernde Aktion ersetzen.
- **Überprüfbares Ergebnis:** Jeder negative Test liefert einen definierten Grund, bewahrt einen sicheren Zustand und hinterlässt keine unzugeordneten Teilressourcen.

### Schritt 10 – Tests und Abnahme aufbauen

- **Ziel:** Fachregeln, Zustandsübergänge, Wiederholsicherheit und Interaktionen automatisiert nachweisen.
- **Betroffene Komponenten:** neue isolierte Tests für Route/Elemente, vorhandene Simulations- und Persistenztests, Testdatenbauer.
- **Abhängigkeiten:** Beobachtbarer Status, Besitzabfragen, deterministisch steuerbare Rückmeldungen.
- **Risiken:** Nur UI-End-to-End-Tests würden Fehlerursachen verdecken; nur synchrone Elemente würden Wartezustände nicht prüfen.
- **Überprüfbares Ergebnis:** Die Szenarien aus Abschnitt 14 laufen deterministisch, einschließlich mehrerer Zyklen, Teilfortschritt, zwei Routen und negativer Fälle.

### Schritt 11 – Zugstraßenfähig erweitern

- **Ziel:** Den gemeinsamen Kern wiederverwenden und ausschließlich dokumentierte Zugregeln ergänzen.
- **Betroffene Komponenten:** Zugregelstrategie, zusätzliche Phasen/Flankenschutz, Zug-ZP, Signal- und Auflöseregeln.
- **Abhängigkeiten:** Vollständige Zugstraßendokumentation; stabiler gemeinsamer Kern aus den Schritten 3 bis 10.
- **Risiken:** Rangierdetails dürfen nicht als Default-Zugregeln wirken; Flankenschutzfreigabe ist derzeit ausdrücklich offen.
- **Überprüfbares Ergebnis:** Rangiertests bleiben unverändert erfolgreich; Zugphasen lassen sich ergänzen, ohne Besitzmodell, Element-API oder allgemeinen Lifecycle neu zu entwerfen.

## 14. Fachliche Testszenarien für die spätere Umsetzung

Wo das Soll-Ergebnis fachlich offen ist, wird kein Ergebnis erfunden; stattdessen wird die notwendige Vorbedingung für den Test benannt.

| Nr. | Szenario | Vorbereitung/Aktion | Erwartetes Ergebnis oder offene Festlegung |
|---:|---|---|---|
| 1 | Zulässige Rangierstraße | PRS/PRZ gesetzt; alle zehn ZP-Bedingungen erfüllt; Weichen rückmelden Sollstellung | Genau eine ZP; Folge 0→1→2→3→4; korrekte Ansprüche, Verschlüsse, Startfahrt und Rangier-ZIF. |
| 2 | Auswahl in umgekehrter Reihenfolge | Erst Ziel, dann Start wählen | Dieselbe eindeutige Route wie bei Start→Ziel wird angefordert. |
| 3 | Unzulässige Start-/Zielkombination | Zwei Elemente ohne gemeinsame Route wählen | Abweisung; keine Route und kein Elementzustand ändern sich. |
| 4 | Mehrdeutige Kombination | Zwei Routen mit identischem Start/Ziel projektieren | Keine willkürliche Auswahl; Abweisung. Zusätzlich sollte Strukturvalidierung nach der künftigen Entscheidung geprüft werden. |
| 5 | Start ohne `PRS` | Alle übrigen Bedingungen positiv | ZP negativ; Status 0; keine automatische spätere Aktivierung. |
| 6 | Ziel ohne `PRZ` | Je einmal Signalziel und Blindziel | ZP negativ; Status 0; keine Ressourcen gesetzt. |
| 7 | Besetztes Fahrwegelement | Fahrweg vor Auswahl besetzen | **Offen:** erwartetes Ergebnis erst nach Beantwortung der Belegungsregel festlegen. Unabhängig davon darf eine belegte Weiche nicht unkontrolliert umgestellt werden. |
| 8 | Fremd beanspruchtes Fahrwegelement | Route A besitzt ein Element; Route B fordert es an | Route B wird in der ZP abgewiesen; Besitz von A bleibt unverändert. |
| 9 | Weiche falsche, aber nicht gesperrte/verschlossene Lage | Weiche frei in Gegenlage | ZP positiv, sofern alle anderen Regeln erfüllt; Status 2 fordert gezielt die Sollstellung an und wartet auf Rückmeldung. |
| 10 | Widersprechende Sperrlage | PSSL/PSSR sperrt die Gegenlage zur projektierten Sollstellung | ZP negativ; kein Stellbefehl. Exakte Links/Rechts-Zuordnung zuvor fachlich/technisch bestätigen. |
| 11 | Widersprechend verschlossene Weiche | Weiche ist in Gegenlage durch andere Anforderung verschlossen | ZP negativ; fremder Verschluss bleibt erhalten. |
| 12 | Gleichlage gemeinsam verschlossene Weiche | Andere Route hält dieselbe Weiche in derselben benötigten Lage, soweit gemeinsame Nutzung fachlich zulässig ist | **Teilweise offen:** Konflikt-/Freigaberegel bestätigen. Zähler/Owner dürfen jedenfalls nicht verloren gehen. |
| 13 | Erfolgreiches Stellen und Verschließen | Rückmeldung zunächst Motorlauf, später Sollstellung | Status 2 wartet; nach Ist-Soll Status 3; eigener Verschluss genau einmal; danach Status 4. |
| 14 | Wiederholte Zyklen in jedem Status | Je Phase viele `work()`-Aufrufe ohne Zustandsänderung | Keine doppelten Ansprüche, Stellrichtungswechsel oder Zählererhöhungen. |
| 15 | Fahrwegsignal | Rolle-1-Signal mit fachlich definierter Gegenrichtungs-/Unterwegsfunktion | Beanspruchung A/B; korrekte Weißschaltung, Haltreaktion und Rücknahme gemäß noch zu ergänzender Regel. |
| 16 | Flankenschutz bei Rangierstraße | Rolle 5 oder 8 an einer Rangierstraße projektieren | Strukturvalidierung weist die Projektierung ab; es gibt keine Rangier-Flankenschutzphase. |
| 17 | Verlust einer Voraussetzung nach Anschaltung | Zum Beispiel Beanspruchungsrückmeldung oder Weichen-Istlage in Status 4 entziehen | Start- und definierte Fahrwegsignale gehen Halt; weiterer Status/Ressourcenumgang nach künftig festgelegter Regel. |
| 18 | Startsignal erreicht Fahrtstellung nicht | Status 4 anfordern, Signalanforderung ablehnen/verzögern | Sicherer Warte-/Fehlerzustand gemäß noch festzulegender Regel; niemals fälschlich „vollständig eingestellt“ melden. |
| 19 | Gültiges Auflöseelement | Aktive Route, zugeordneten Auflöser anklicken | Halt anfordern; vor Halt keine Freigabe; danach eigene Verschlüsse, dann eigene Ansprüche, Anzeigen, schließlich Status 0. |
| 20 | Nicht zugeordnetes Auflöseelement | Aktive Route, fremden Auflöser anklicken | Keine Wirkung auf die Route oder ihre Ressourcen. |
| 21 | Mehrere aktive Routen am selben Auflöser | Gemeinsames Auflöseelement bedienen | Jede zulässige aktive Route beginnt unabhängig ihren sicheren Auflöseablauf; Fehler einer Route darf fremde Ressourcen nicht löschen. |
| 22 | Teilweise eingestellte Route auflösen | Auflösung während Status 1, 2 oder 3 | Nur tatsächlich eigene gesetzte Ressourcen zurücknehmen; Reihenfolge und Signalvoraussetzung gemäß noch zu präzisierender Fachregel. |
| 23 | Halt-Rückmeldung bleibt aus | Auflösung starten, HSV bleibt ungleich 0 | Keine Verschluss-/Beanspruchungsfreigabe; Diagnose/Timeout gemäß künftiger Regel. |
| 24 | Zwei Rangierstraßen mit gemeinsamem Element | Route A aktiv; B anfordern; danach A auflösen | B entsprechend Konfliktregel abweisen; Auflösung A entfernt nur A. Anschließende neue Anforderung B funktioniert. |
| 25 | Rangier- und spätere Zugstraße teilen Element | Eine Art hält Beanspruchung/Verschluss, andere fordert an | Besitzer- und Typkonflikt deterministisch prüfen; keine Überschreibung. Konkrete Zugzulassung erst nach Zugdokumentation. |
| 26 | Reset in Zwischenzustand | Route besitzt Teilressourcen, dann Simulationsreset | Definierte Grundstellung ohne Restbeanspruchung/-verschluss. Genaues Reset-Sicherheitsverhalten dokumentieren und testen. |
| 27 | ZIF setzen und lösen | Signalziel und Blindziel separat | Status 4 zeigt Rangier-ZIF; Auflösung entfernt es zum dokumentierten Zeitpunkt; PRZ-Abweisung bleibt wirksam. |
| 28 | Persistenz-Roundtrip | Route mit Weichen- und Signal-Sollstellungen speichern/laden | Typ, Rollen, Element-IDs und Sollstellungen bleiben identisch und gelangen vollständig in die Laufzeitprojektion. |

## 15. Kompakte Zusammenfassung

- **Ist eine vollständige Implementierung derzeit möglich?** Nein. Eine Teilimplementierung ist möglich, eine fachlich belastbare Gesamtlösung noch nicht.
- **Was blockiert?** Vor allem Besitzer-/Referenzmodell, Verlust der Sollstellungen im Laufzeitmodell, fehlende Zustände 1–4 und Auflösungsphasen sowie offene Regeln für Fahrwegsignale, Fehlerfolgen, Belegung und vollständige Rücknahme.
- **Was ist bereits sicher nutzbar?** Ungeordnete eindeutige Start-/Zielauswahl, strukturelle Kernrollen, persistenter Fahrstraßentyp/Rolle/Sollstellung, die zehn dokumentierten Rangier-ZP-Bedingungen, grobe Einstellfolge, Signal-Halt-/Rangierfahrtwerte, Weichenrückmeldung und das Prinzip „Halt vor Freigabe“.
- **Welche Architektur wird empfohlen?** Eine unveränderliche typisierte Routenprojektion, ein gemeinsamer phasenbasierter Ablaufkern mit einer Statuswahrheit, besitzersichere idempotente Elementoperationen und getrennte Regelstrategien für Rangier- und Zugstraßen.
- **Was ist als Nächstes zu tun?** Zuerst die P0-Fachfragen verbindlich beantworten und die Zustands-/Fehler-/Auflösedokumentation konsolidieren. Danach Besitzmodell und Laufzeitprojektion festlegen, bevor der Zustandsautomat implementiert wird.

## 16. Änderungsnachweis dieser Analyse

Für diese Analyse wurden **keine Änderungen am Programmcode, keine vorbereitenden Refactorings und keine Änderungen an bestehenden Dokumentationsdateien vorgenommen**. Neu erstellt wurde ausschließlich dieser Markdown-Analysebericht.
