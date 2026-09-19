# Analysebericht: Rangierstraßenlogik in `estw3`

## 1. Ergebnis und Untersuchungsrahmen

Die dokumentierte Rangierstraßenmechanik ist im Laufzeitcode derzeit nur als Gerüst vorhanden. Nutzbar implementiert sind einzelne Elementzustände, die zyklische Aufrufstruktur und Teile der Datenvalidierung. Die eigentliche Zustandsfolge 0–4, das Setzen und Prüfen der Beanspruchungen, das gezielte Stellen und Verschließen der Weichen sowie die sichere Auflösung fehlen.

Die wesentlichen technischen Blocker sind:

- `SollStellung` geht beim Aufbau des Laufzeitmodells verloren.
- `Fs` unterscheidet im Ablauf nicht zwischen Rangier- und Zugstraßen.
- Beanspruchungen speichern nur die Fahrstraßenart, nicht den Besitzer.
- Weichen können nur umgesteuert, aber nicht gezielt in eine Sollstellung befohlen werden.
- Der dokumentierte Status wird durch ein zusätzliches, unabhängiges `bool an` verfälscht.
- Die Bedienoberfläche fordert keine Fahrstraße anhand von Start und Ziel an und löst sie nicht über das Auflöseelement auf.
- `ZielElement::setZIF()` kann wegen eines Fehlers keinen ZIF setzen.
- Es gibt keine zyklische Überwachung der erreichten Sicherungsstufen.

Für eine sichere Implementierung reicht deshalb keine lokale Ergänzung in `Fs::work()`. Benötigt wird eine gemeinsame Fahrstraßen-Zustandsmaschine mit rangierstraßenspezifischer Strategie sowie eine owner-basierte Beanspruchungs- und Verschlussverwaltung.

Die Untersuchung war rein lesend. Es wurden keine Dateien geändert, kein Build und keine Formatierung ausgeführt.

## 2. Relevante fachliche Dokumentation

### 2.1 Allgemeine Fahrstraßenlogik

Die gemeinsame Vorgabe definiert den ganzzahligen Status als alleinige Zustandsquelle:

- Status `0`: ausgeschaltet.
- Status `> 0`: eingeschaltet, einschließlich aller Zwischenzustände.
- Ein zusätzliches unabhängiges Ein-/Aus-Flag ist ausdrücklich nicht vorgesehen.
- Die Fahrstraße wird über ein Startsignal und ein Zielsignal beziehungsweise Blindziel ausgewählt.
- Die Zulassungsprüfung wird genau einmal zu Beginn ausgeführt.
- Voraussetzungen für Statuswerte größer als 1 müssen zyklisch überwacht werden.
- Das Startsignal darf nur einmal beim Erreichen von Status 4 in Fahrt gestellt werden.
- Bei Verlust einer zyklisch überwachten Voraussetzung sind Startsignal und vorhandene Unterwegssignale auf Halt zu stellen.

Siehe [fahrstrasse.md:7](D:/csharp/CPP/estw/estw3/.agents/fahrstrasse.md:7), [fahrstrasse.md:16](D:/csharp/CPP/estw/estw3/.agents/fahrstrasse.md:16), [fahrstrasse.md:26](D:/csharp/CPP/estw/estw3/.agents/fahrstrasse.md:26) und [fahrstrasse.md:32](D:/csharp/CPP/estw/estw3/.agents/fahrstrasse.md:32).

### 2.2 Struktur einer Rangierstraße

Eine Rangierstraße muss besitzen:

- genau ein Startsignal,
- genau ein Zielsignal oder Blindziel,
- genau ein Auflöseelement,
- beliebig viele Fahrwegelemente.

Sie darf keine Rollen für Flankenschutz, Vorweg-, Unterwegs- oder Flankenschutztransportelemente besitzen. Signale können jedoch als normale Fahrwegelemente vorkommen und werden dort fachlich als Gegenrichtungs- oder Unterwegssignale behandelt.

Siehe [rangierstrasse.md:7](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:7) und die verbindliche Zuordnungstabelle in [datenstrukturen.md:128](D:/csharp/CPP/estw/estw3/.agents/datenstrukturen.md:128).

### 2.3 Einmalige Zulassungsprüfung

Vor Status 1 müssen einmalig alle folgenden Bedingungen erfüllt sein:

1. Kein Fahrwegelement wird durch eine andere Zug- oder Rangierstraße beansprucht.
2. Keine Weiche ist entgegen ihrer projektierten `SollStellung` gesperrt.
3. Keine Weiche ist entgegen ihrer `SollStellung` verschlossen.
4. Strang A des Startsignals ist frei.
5. Das Startsignal steht für Rangierstraßen fachlich auf Halt.
6. Am Startsignal ist keine Signalsperre aktiv.
7. Bei einem Zielsignal ist dessen Strang B frei.
8. Auf keinem Fahrstraßenelement liegt eine Befahrbarkeitssperre.

Siehe [rangierstrasse.md:36](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:36).

Die Zulassungsprüfung darf nicht zyklisch wiederholt werden, weil Status 1 gerade Zustände verändert, die Bestandteil dieser Prüfung sind.

### 2.4 Zustandsfolge

Die dokumentierte Folge lautet:

| Status | Bedeutung | Aktion beziehungsweise Übergang |
|---|---|---|
| 0 | Grundstellung | Einmalige Zulassungsprüfung nach Benutzeranforderung |
| 1 | Beanspruchungen anfordern | Fahrweg beanspruchen; Signale A+B; Start A; Zielsignal B; Weichen A und B/C |
| 2 | Weichen stellen | Erst nach bestätigten Beanspruchungen gezielt in `SollStellung` bringen |
| 3 | Weichen verschließen | Erst nach bestätigten Beanspruchungen und tatsächlicher Sollstellung |
| 4 | Eingestellt | Erst nach bestätigtem Verschluss Startsignal einmalig in projektierte Fahrtstellung bringen und ZIF anzeigen |

Siehe [rangierstrasse.md:49](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:49), [rangierstrasse.md:61](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:61), [rangierstrasse.md:75](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:75), [rangierstrasse.md:81](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:81) und [rangierstrasse.md:90](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:90).

### 2.5 Weichen und Verschlüsse

Eine Weiche besitzt genau einen gemeinsamen, nichtnegativen Verschlusszähler. Jede Fahrstraßen- oder Flankenschutzforderung erhöht ihn einmal; jede zugehörige Aufhebung vermindert ihn einmal. Bei einem Wert größer 0 darf die Weiche nicht umgestellt werden.

Siehe [weiche.md:5](D:/csharp/CPP/estw/estw3/.agents/weiche.md:5).

Für Rangierstraßen ist kein Flankenschutzstatus vorgesehen. Flankenschutzrollen sind strukturell verboten. Der gemeinsame Zähler muss trotzdem für spätere Zugstraßen- und gemeinsame Weichennutzung geeignet bleiben.

### 2.6 Signale sowie Weiß- und Dunkelschaltung

Für Rangierstraßen gilt:

- HSV `0`: Halt.
- HSV `-1`: Rangierfahrt mit Rot.
- HSV `-2`: Rangierfahrt ohne Rot.
- HSV größer `0`: für Rangierstraßen ungültig und als Halt zu interpretieren.
- HSV `-3`: Signal ist für die Fahrstraße nicht relevant.
- VSV ist ausschließlich für Zugstraßen relevant.

Siehe [signale.md:21](D:/csharp/CPP/estw/estw3/.agents/signale.md:21).

Signale als Rangierstraßen-Fahrwegelemente sollen laut [rangierstrasse.md:67](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:67) auf A und B beansprucht und im Verlauf weißgeschaltet werden. Ein genauer Status für diese Weißschaltung ist nicht festgelegt. Eine Dunkelschaltung ist für Rangierstraßen nicht ausdrücklich gefordert; HSV `-3` wird lediglich als „nicht relevant“ definiert.

### 2.7 Fahrstraßenauflösung

Das Anklicken des zugehörigen Auflöseelements startet die Auflösung. Die Reihenfolge ist bindend:

1. Halt am Startsignal anfordern.
2. Tatsächliche Haltstellung abwarten.
3. Eigene Weichenverschlüsse entfernen.
4. Eigene Beanspruchungen entfernen.
5. Status auf 0 setzen.

Siehe [rangierstrasse.md:106](D:/csharp/CPP/estw/estw3/.agents/rangierstrasse.md:106).

Das Löschen des ZIF und die Rücknahme weißgeschalteter Fahrwegsignale werden nicht ausdrücklich beschrieben, sind technisch aber notwendig beziehungsweise fachlich zu klären.

### 2.8 Gleisfreimeldung

Eine gesonderte Dokumentation zur Gleisfreimeldung existiert im Projekt nicht. Die Rangierstraßenvorgabe nennt auch keine Freimeldebedingung anhand von `besetzt`. Dokumentiert ist nur die Befahrbarkeitssperre.

Daher darf eine Belegungsprüfung nicht stillschweigend als Zulassungs- oder Statusbedingung ergänzt werden. Ob Rangierstraßen belegte Fahrwege zulassen sollen, muss fachlich geklärt werden.

### 2.9 Unterschiede zu Zugstraßen

Die gemeinsame Basis aus Beanspruchung, Weichenstellung, Verschluss, zyklischer Überwachung und sicherer Auflösung ist wiederverwendbar. Zugstraßen benötigen zusätzlich:

- andere Signalgeschwindigkeiten,
- VSV,
- Unterwegs- und Vorwegelemente,
- Flankenschutz und Flankenschutztransporte,
- eine noch nicht festgelegte zusätzliche Sicherungsstufe.

Die Zugstraßenzulassung und die Flankenschutzmechanik sind noch nicht vollständig dokumentiert; siehe [zugstrasse.md:40](D:/csharp/CPP/estw/estw3/.agents/zugstrasse.md:40) und [zugstrasse.md:86](D:/csharp/CPP/estw/estw3/.agents/zugstrasse.md:86). Rangierstraßen dürfen deshalb nicht von einer noch unvollständigen Zugstraßenimplementierung abhängig gemacht werden.

---

## 3. Bestehende Implementierung

### 3.1 Persistentes Datenmodell

Die Strukturen sind grundsätzlich geeignet:

- `Fahrstrasse::TypId` unterscheidet Rangierstraße `1` und Zugstraße `2`.
- `FahrstrassenElement` enthält Rolle, Elementreferenz und `SollStellung`.
- `FullProjekt` enthält die Fahrstraßen bereits mit ihren Zuordnungen.

Siehe [structs.h:74](D:/csharp/CPP/estw/estw3/structs.h:74) und [structs.h:85](D:/csharp/CPP/estw/estw3/structs.h:85).

`generate_fullprojekte()` ordnet die flachen Fahrstraßenelemente korrekt ihren Fahrstraßen zu; dabei bleibt `SollStellung` erhalten: [aktionen.cpp:665](D:/csharp/CPP/estw/estw3/aktionen.cpp:665).

### 3.2 Validierung und Fahrstraßeneditor

`isvalid_estwdata()` prüft bereits:

- Fahrstraßentyp `1` oder `2`,
- genau einen Start, ein Ziel und ein Auflöseelement,
- die Elementtypen dieser drei Rollen,
- das Verbot der Rollen 5–8 bei Rangierstraßen.

Siehe [validate.cpp:328](D:/csharp/CPP/estw/estw3/validate.cpp:328) und [validate.cpp:473](D:/csharp/CPP/estw/estw3/validate.cpp:473).

Unvollständig sind insbesondere:

- die allgemeine Zuordnungstabelle für Rolle 1 und Rolle 8,
- der Wertebereich von `SollStellung` am Startsignal,
- die Abhängigkeit der Startsignalstellung vom Fahrstraßentyp,
- die Signal-Unterelementarten,
- gegebenenfalls die Programmfälle `PRS` beziehungsweise `PRZ`.

Der Editor prüft bei Weichen lediglich `-1` oder `1`; andere Rollen-/Elementkombinationen werden zunächst zugelassen: [fahrstrasseneditor.cpp:90](D:/csharp/CPP/estw/estw3/fahrstrasseneditor.cpp:90).

Ein bestehender Persistenztest speichert für eine Rangierstraße am Startsignal die Sollstellung `16`, obwohl positive HSV-Werte nach aktueller Dokumentation keine Rangierfahrtstellung sind: [fahrstrassen_tests.cpp:152](D:/csharp/CPP/estw/estw3/tests/fahrstrassen_tests.cpp:152). Dieser Testbestand widerspricht damit der aktuellen Fachvorgabe.

### 3.3 Laufzeitmodell `Fs`

`FsInitData` enthält Referenzen auf Start, Ziel, Auflöseelement sowie Listen für die verschiedenen Rollen: [fs.h:39](D:/csharp/CPP/estw/estw3/estw/fs.h:39).

Problematisch sind:

- Die Zuordnungen enthalten nur Elementzeiger; `SollStellung` und Rollenmetadaten fehlen.
- Der Fahrstraßentyp wird nur indirekt in der vollständigen Kopie von `Fahrstrasse` gehalten.
- `status` ist ein untypisierter `int`.
- Daneben existiert `bool an`, obwohl die Dokumentation Ein/Aus aus dem Status ableitet.
- Der Status ist von außen nicht prüfbar.
- `Fs` ist `final`; eine spätere Aufteilung müsste daher über Komposition/Strategien oder eine Refaktorierung erfolgen.

Siehe [fs.h:56](D:/csharp/CPP/estw/estw3/estw/fs.h:56).

`Fs::work()` implementiert nur:

- bei `an == false`: nichts,
- bei Status 0: wiederholter Aufruf von `condition1()`,
- bei Erfolg Übergang auf Status 1.

Die Statuswerte 1–4 sind nur auskommentierte Fragmente: [fs.cpp:61](D:/csharp/CPP/estw/estw3/estw/fs.cpp:61).

Damit wird die Zulassungsprüfung entgegen der Vorgabe bei jedem Zyklus wiederholt, bis sie irgendwann positiv wird. Eine frühere Benutzeranforderung bleibt also unbegrenzt aktiv.

`setAn(true)` setzt bereits `an`, ohne Zulassung oder Statusübergang. Dadurch kann eine fachlich ausgeschaltete Fahrstraße mit Status 0 im Auflösemelder bereits als eingeschaltet erscheinen. `setAn(false)` stellt lediglich den Status auf 0 und das Startsignal auf Halt; Beanspruchungen, Verschlüsse und ZIF werden nicht freigegeben: [fs.cpp:50](D:/csharp/CPP/estw/estw3/estw/fs.cpp:50).

### 3.4 Bestehende Zulassungsprüfung

`condition1()` prüft nur:

- ZIF des Zielelements ist `None`,
- kein Fahrwegelement ist beansprucht,
- Strang A des Startsignals ist frei.

Siehe [fs.cpp:103](D:/csharp/CPP/estw/estw3/estw/fs.cpp:103).

Es fehlen:

- entgegenstehende Weichensperren,
- entgegenstehende Weichenverschlüsse,
- Haltstellung des Startsignals,
- Signalsperre,
- Strang B des Zielsignals,
- sämtliche Befahrbarkeitssperren.

Die Prüfung auf freien ZIF ist dagegen nicht Bestandteil der dokumentierten Rangierstraßenzulassung und stellt eine zusätzliche, bislang unbegründete Bedingung dar.

`condition2()` und `condition3()` liefern bedingungslos `true` und werden derzeit nicht benutzt.

### 3.5 Verlust der Projektierung beim Aufbau

`LupeForm::estwInit()` baut die Laufzeitreferenzen auf, übernimmt aber weder `SollStellung` noch das jeweilige `FahrstrassenElement`. Außerdem werden die Rollen 5, 7 und 8 nicht in ihre vorhandenen Vektoren übertragen: [lupeform.cpp:855](D:/csharp/CPP/estw/estw3/lupeform.cpp:855).

Damit fehlen der Laufzeit insbesondere:

- die Sollstellung jeder Weiche,
- die projektierte HSV des Startsignals,
- eine belastbare Zuordnung zwischen einem Element und seiner Rolle,
- die Grundlage für spätere Zugstraßenrollen.

### 3.6 Beanspruchungen

Vorhanden sind:

- `Gleis`: ein `BEA_t`,
- `Signal`: `BEAA` und `BEAB`,
- `Weiche`: `BEAB` und `BEAC`.

Siehe [gleis.h:12](D:/csharp/CPP/estw/estw3/estw/gleis.h:12), [signal.h:59](D:/csharp/CPP/estw/estw3/estw/signal.h:59) und [weiche.h:68](D:/csharp/CPP/estw/estw3/estw/weiche.h:68).

Probleme:

- Es wird nur `Rangier` oder `Zug`, aber keine Fahrstraßen-ID gespeichert.
- Eine Rangierstraße kann daher nicht nachweisen, dass eine vorhandene Beanspruchung ihre eigene ist.
- Eine Auflösung kann nicht sicher ausschließlich die eigenen Beanspruchungen entfernen.
- Der dokumentierte Strang A einer Weiche existiert im Modell nicht.
- `Weiche::reset()` löscht `BEAB` und `BEAC` nicht: [weiche.cpp:180](D:/csharp/CPP/estw/estw3/estw/weiche.cpp:180).
- Keine Fahrstraßenlogik setzt oder entfernt diese Beanspruchungen derzeit.

Positiv ist, dass `Signal::setBEAA()` bei einer Änderung automatisch Halt anfordert. Dieser Sicherheitsmechanismus kann grundsätzlich weiterverwendet werden: [signal.cpp:172](D:/csharp/CPP/estw/estw3/estw/signal.cpp:172).

### 3.7 Weichenstellung und Verschluss

`Weiche` besitzt bereits:

- aktuelle Stellung,
- Motorlauf,
- Umstellsperre,
- Auffahrzustand,
- einen gemeinsamen Verschlusszähler.

Der Zähler wird nichtnegativ gehalten und blockiert `WU()`: [weiche.cpp:73](D:/csharp/CPP/estw/estw3/estw/weiche.cpp:73).

Für Fahrstraßen ungeeignet ist derzeit:

- `WU()` schaltet nur in die jeweils andere Lage; eine gewünschte Lage kann nicht übergeben werden.
- Wiederholte Befehle während eines Motorlaufs können die Laufrichtung umkehren.
- Der Verschlusszähler besitzt keine Besitzerinformation.
- Wiederholtes `setVerschluss()` erhöht den Zähler erneut.
- Eine Fahrstraße kann nicht prüfen, ob gerade ihr Verschluss wirksam ist.
- `PSSL` und `PSSR` werden beim Aufbau der Weiche im `LupeForm` nicht aus den persistenten Daten übernommen: [lupeform.cpp:961](D:/csharp/CPP/estw/estw3/lupeform.cpp:961).

Die dokumentierte Abbildung ist ausdrücklich `SollStellung -1 = Links` und `1 = Rechts`; siehe [estw.md:149](D:/csharp/CPP/estw/estw3/.agents/estw.md:149). Sie darf nicht aus dem internen Vorzeichen der Zungenstellung abgeleitet werden, da die Implementierung intern `-10` als `Rechts` und `10` als `Links` verwendet: [weiche.cpp:63](D:/csharp/CPP/estw/estw3/estw/weiche.cpp:63).

### 3.8 Signale

`Signal` implementiert bereits HSV, VSV, Halt, Signalsperre, Beanspruchungen, ZIF und Bedingungen für verschiedene Signalstellungen: [signal.h:26](D:/csharp/CPP/estw/estw3/estw/signal.h:26).

Teilweise wiederverwendbar sind:

- `HaGT()`,
- `SetHSV(-1/-2)`,
- `GetHSV()`,
- `GetFSS()`,
- A/B-Beanspruchung,
- ZIF-Datenfeld.

Probleme:

- `SignalInitData` enthält keine `UnterElementArt`; die fachliche Signalart ist im Laufzeitobjekt unbekannt.
- `SetHSV()` meldet ungültige Bereiche nur per Log und bricht nicht grundsätzlich ab.
- Der Rückgabewert ist `void`; eine Fahrstraße erfährt nicht direkt, ob der Befehl angenommen wurde.
- Die Fahrtbedingungen prüfen teilweise `PRS`, obwohl dies in der Rangierstraßenzulassung nicht genannt ist.
- Signalziel und Blindziel behandeln ihre Projektierungsfälle unterschiedlich.
- Es gibt keine Fahrstraßensteuerung der Weißschaltung.

Die Lupe kann HSV `-2` als weißen Rangierbegriff darstellen und unterscheidet die Unterelementarten nur auf Darstellungsebene: [lupesignal.cpp:157](D:/csharp/CPP/estw/estw3/lupe/lupesignal.cpp:157) und [lupesignal.cpp:293](D:/csharp/CPP/estw/estw3/lupe/lupesignal.cpp:293).

### 3.9 Ziel und ZIF

`ZielElement` kapselt Signal oder Blindziel, bietet aber nur ZIF-Zugriffe. Es kann insbesondere den B-Strang eines Zielsignals weder prüfen noch beanspruchen.

Darüber hinaus ignoriert `ZielElement::setZIF()` sein Argument und setzt in beiden Fällen immer `ZIF_t::None`: [fs.cpp:149](D:/csharp/CPP/estw/estw3/estw/fs.cpp:149). Status 4 kann deshalb über diese Abstraktion keinen Rangier-ZIF anzeigen.

### 3.10 Zyklische Ausführung und Bedienung

`Estw::work()` ruft zuerst alle Elemente und anschließend alle Fahrstraßen auf: [estw.cpp:22](D:/csharp/CPP/estw/estw3/estw/estw.cpp:22). Der Simulationstimer startet diesen Zyklus einmal pro Sekunde: [lupeform.cpp:138](D:/csharp/CPP/estw/estw3/lupeform.cpp:138).

Die UI-Anbindung ist nicht fachlich verwendbar:

- „Action 7“ schaltet ausschließlich die fest codierte Fahrstraße 100 über `setAn()` um.
- Die Verarbeitung zweier markierter Signale ist durch ein sofortiges `return` deaktiviert.
- Blindziele können in der Simulation nicht als Ziel ausgewählt werden.
- Linksklicks markieren nur Signale.
- Ein Linksklick auf ein Auflöseelement erzeugt keine Auflösungsanforderung.

Siehe [lupeform.cpp:802](D:/csharp/CPP/estw/estw3/lupeform.cpp:802) und [lupeform.cpp:1165](D:/csharp/CPP/estw/estw3/lupeform.cpp:1165).

Der sichtbare Auflösemelder wird lediglich aus `Fs::getAn()` und der Zuordnung des Auflöseelements abgeleitet: [estw.cpp:61](D:/csharp/CPP/estw/estw3/estw/estw.cpp:61). Die Darstellung selbst ist vorhanden, aber nicht die Auflösungsaktion.

### 3.11 Gleisbelegung

`EstwElement` enthält `besetzt` und `sperre`: [element.h:40](D:/csharp/CPP/estw/estw3/estw/element.h:40). Beide Zustände können im Kontextmenü manuell geschaltet werden: [lupeelement.cpp:127](D:/csharp/CPP/estw/estw3/lupe/lupeelement.cpp:127).

Eine automatische Gleisfreimeldung oder eine Verwendung von `besetzt` in der Fahrstraßenlogik existiert nicht. Das entspricht dem fehlenden fachlichen Kriterium, muss aber vor der Implementierung bewusst bestätigt werden.

### 3.12 Tests

Vorhanden sind:

- Editor- und Persistenztests,
- ein Test für die Sichtbarkeit des Auflösemelders.

Der Auflösemeldertest manipuliert nur `setAn()`; eine echte Auflösung wird nicht getestet: [aufloese_melder_tests.cpp:13](D:/csharp/CPP/estw/estw3/tests/aufloese_melder_tests.cpp:13).

Es fehlen Tests für:

- die Zustandsfolge 0–4,
- die acht Zulassungsbedingungen,
- Weichenbewegung und Verschluss,
- zyklischen Bedingungsverlust,
- einmalige Signalstellung,
- vollständige Auflösung,
- überlappende Fahrstraßen und Besitzverhältnisse.

---

## 4. Soll-Ist-Vergleich

| Dokumentierte Anforderung | Bestehende Implementierung | Bewertung | Erforderliche Änderung |
|---|---|---|---|
| Status 0 = aus, Status >0 = an | Separates `bool an` neben `status` | widersprüchlich | `getAn()` aus dem Status ableiten; unabhängigen Zustand entfernen |
| Anforderung über Start und Ziel | Nur fest codierte Fahrstraße 100; Auswahlverarbeitung deaktiviert | nicht vorhanden | Start-/Zielauswahl und eindeutige Routensuche in `Estw` |
| Rangier/Zug unterscheiden | `TypId` ist persistent vorhanden, wird im Ablauf nicht ausgewertet | teilweise vorhanden | Typisierte Fahrstraßenart und rangierspezifische Policy |
| Zulassung genau einmal | `condition1()` wird bei jedem Zyklus in Status 0 erneut ausgeführt | widersprüchlich | Ereignisgesteuerte Anforderung; Prüfresultat verbrauchen |
| Vollständige Rangierzulassung | Drei Teilprüfungen, zusätzlich undokumentierte ZIF-Prüfung | teilweise vorhanden | Alle acht Bedingungen implementieren und benennen |
| Keine Befahrbarkeitssperre auf irgendeinem Mitglied | `sperre` existiert, wird nicht geprüft | nicht vorhanden | Über alle projektierten Mitglieder prüfen |
| Startsignal auf fachlichem Halt | HSV wird nicht geprüft | nicht vorhanden | Zentraler fahrstraßentypabhängiger Halt-Prädikator |
| Startsignal ohne Signalsperre | Signalzustand vorhanden, nicht geprüft | nicht vorhanden | `GetFSS()` in die Zulassung aufnehmen |
| Zielsignal B frei | `ZielElement` kann B nicht abfragen | nicht vorhanden | Zielabstraktion um B-Beanspruchung erweitern |
| Projektierte `SollStellung` verfügbar | Persistiert, beim Laufzeitaufbau verworfen | nicht vorhanden | Laufzeitmitglied aus Projektion und Elementzeiger bilden |
| Beanspruchungen in Status 1 anfordern und bestätigen | Elementfelder vorhanden, `Fs` setzt nichts | nicht vorhanden | Owner-basierte Beanspruchungsanforderung und Nachprüfung |
| Weiche auf A und B/C beanspruchen | Weiche besitzt nur B/C | nicht vorhanden | Strang A ergänzen oder fachlich anders eindeutig modellieren |
| Signal-Fahrwegelement auf A+B und weiß | Darstellung vorhanden, keine Fahrstraßensteuerung | teilweise vorhanden | Claims setzen und definierte Weißstellung einmalig anfordern |
| Weichen gezielt stellen | Nur richtungsumschaltendes `WU()` | nicht vorhanden | Idempotenter Befehl `requestStellung(SollStellung)` |
| Weichenstellung zyklisch prüfen | Stellung lesbar, von `Fs` ungenutzt | teilweise vorhanden | Kumulative Statusbedingung |
| Gemeinsamer Verschlusszähler | Zähler vorhanden und nichtnegativ | teilweise vorhanden | Besitzer-/Ledgerverwaltung und idempotentes Sperren |
| Verschluss nur einmal setzen | `setVerschluss()` zählt bei jedem Aufruf hoch | nicht vorhanden | Pro Fahrstraße nur eine registrierte Forderung |
| Kein Flankenschutz bei Rangierstraße | Datenvalidierung verbietet Rollen 5–8 | vollständig strukturell vorhanden | Rangierpolicy ohne Flankenschutzstufe halten |
| Startsignal in Status 4 einmalig auf Fahrt | `SetHSV()` vorhanden, wird von `Fs` nicht benutzt | nicht vorhanden | Eintrittsaktion mit projektierter HSV `-1/-2` |
| ZIF in Status 4 setzen | Felder vorhanden; Wrapper löscht immer | fehlerhaft | `ZielElement::setZIF(value)` korrigieren und Ergebnis prüfen |
| Zyklische Überwachung für Status >1 | Keine Statuslogik oberhalb 1 | nicht vorhanden | Kumulative Guards je Status |
| Halt bei weggefallenen Voraussetzungen | Nur `setAn(false)` ruft Halt auf | nicht vorhanden | Sicherheitsreaktion im Zyklus; keine automatische Wiederfahrt |
| Auflösung per Auflöseelement | Nur Anzeige und manuelle `FAUF`-Umschaltung | nicht vorhanden | Klickereignis an `Estw`/`Fs` weiterleiten |
| Halt vor Entsicherung | Keine Auflösungssequenz | nicht vorhanden | Mehrphasige Auflösung mit tatsächlicher Haltprüfung |
| Nur eigene Claims/Verschlüsse entfernen | Zustände enthalten keine Route-ID | nicht vorhanden | Besitzerbasierte Claims und Verschlüsse |
| Gleisfreimeldung | `besetzt` vorhanden; keine fachliche Vorgabe oder Fahrstraßennutzung | noch unklar | Vor Implementierung fachlich entscheiden |
| Strukturvalidierung | Kernrollen geprüft; Zuordnungstabelle und Sollwerte unvollständig | teilweise vorhanden | Validator und Editor vervollständigen |
| Laufzeittests | Nur Melderanzeige, Editor und Persistenz | nicht vorhanden | Deterministische Zustandsmaschinen- und Integrationstests |

---

## 5. Vorgeschlagene Architektur

### 5.1 Gemeinsame Zustandsmaschine mit typabhängiger Strategie

Empfohlen wird, `Fs` als gemeinsamen Ablaufkoordinator beizubehalten, die typabhängigen Regeln aber über Komposition auszulagern:

```text
Fs
 ├─ gemeinsamer Status- und Auflösungsautomat
 ├─ RuntimeFahrstrassenDaten
 ├─ Besitzer-/Aktionsledger
 └─ FahrstrassenartPolicy
      ├─ RangierstrassenPolicy
      └─ ZugstrassenPolicy (später, derzeit fachlich unvollständig)
```

`Fs` bleibt verantwortlich für:

- Status und Eintrittsaktionen,
- einmalige Behandlung einer Anforderung,
- zyklische kumulative Prüfungen,
- Fehlerlatch,
- sichere Auflösungsreihenfolge,
- Protokollierung der tatsächlich gesetzten Claims und Verschlüsse.

Die Policy definiert:

- Zulassungsbedingungen,
- Fahrstraßenart der Beanspruchung,
- gültige Startsignalgeschwindigkeiten,
- Behandlung von Fahrwegsignalen,
- ZIF/FÜM/VSV,
- optionale zusätzliche Sicherungsstufen wie Flankenschutz.

Damit wird keine zweite unabhängige Zustandsmaschine für Zugstraßen benötigt.

### 5.2 Typisierte Zustände und Rollen

Empfohlen werden mindestens:

```cpp
enum class FahrstrassenStatus : int {
    Aus = 0,
    Beanspruchungen = 1,
    WeichenStellen = 2,
    WeichenVerschliessen = 3,
    Eingestellt = 4
};

enum class AufloesePhase {
    Keine,
    HaltAnfordern,
    HaltPruefen,
    VerschluesseLoesen,
    BeanspruchungenLoesen
};
```

Zusätzlich sollten die magischen Rollen- und Fahrstraßentyp-IDs durch `enum class`-Werte zentralisiert werden. Die persistenten Integerwerte können weiterhin unverändert gespeichert werden.

`getAn()` sollte ausschließlich `status != Aus` liefern. Eine Auflösungsphase bleibt damit bis zur vollständig abgeschlossenen Entsicherung eingeschaltet.

### 5.3 Laufzeitmitglieder müssen die Projektierung behalten

Statt bloßer Elementzeiger wird eine Struktur benötigt, beispielsweise:

```cpp
struct RuntimeFahrstrassenElement {
    FahrstrassenElement projektion;
    PEstwElement element;
};
```

Daraus können beim Aufbau typisierte Sichten erzeugt werden:

- Startsignal samt projektierter HSV,
- Ziel samt Projektion,
- Weiche samt projektierter Lage,
- Signal als Fahrwegelement,
- alle Mitglieder für die Befahrbarkeitssperrenprüfung.

Dadurch bleiben Rolle, `ElementId`, `SollStellung` und Laufzeitobjekt untrennbar verbunden.

### 5.4 Owner-basierte Beanspruchungen

`BEA_t` allein reicht als Besitznachweis nicht aus. Empfohlen wird eine Beanspruchung mit mindestens:

- Fahrstraßen-ID,
- Fahrstraßenart,
- Strang beziehungsweise Beanspruchungsart.

Für Signal und Weiche werden getrennte Claims für A/B/C benötigt. Für Gleise reicht ein Ganz-Element-Claim.

Technisch sollte eine wiederholte Anforderung derselben Fahrstraße idempotent sein. Das Lösen darf nur einen Claim entfernen, dessen Besitzer die auflösende Fahrstraße ist.

### 5.5 Owner-basierter Weichenverschluss

Der dokumentierte gemeinsame Zähler bleibt fachlich erhalten. Intern sollte er aus registrierten Forderungen oder ergänzend zu einer Besitzerliste verwaltet werden:

- `lock(routeId, purpose)` ist idempotent,
- `isLockedBy(routeId, purpose)` prüft den eigenen Verschluss,
- `unlock(routeId, purpose)` entfernt genau eine eigene Forderung,
- `getVerschluss()` liefert weiterhin die Gesamtzahl.

Damit können spätere Fahrstraßen- und Flankenschutzforderungen denselben Zähler sicher gemeinsam verwenden.

### 5.6 Deterministischer Weichenbefehl

Zusätzlich zu der manuellen Bedienfunktion `WU()` wird ein Fahrstraßenbefehl benötigt:

```cpp
requestStellung(WeicheStellung ziel)
```

Der Befehl muss:

- nichts tun, wenn die Zielstellung bereits erreicht ist,
- einen laufenden Motor nicht bei jedem Zyklus umkehren,
- eine entgegenstehende Sperre oder einen Verschluss ablehnen,
- seinen Annahmestatus zurückgeben,
- zyklisch erneut und idempotent aufgerufen werden können.

Die Konvertierung muss ausdrücklich `-1 → Links` und `1 → Rechts` verwenden.

### 5.7 Signalabstraktion

Erforderlich sind gemeinsame Prädikate und Befehle:

- `isHalt(FahrstrassenArt)`,
- `isFahrt(FahrstrassenArt)`,
- `requestHSV(int)` mit Ergebnis,
- Signal-Unterelementart im Laufzeitobjekt,
- owner-basierte A-/B-Beanspruchungen.

Für Rangierstraßen darf die VSV nicht verändert werden.

`ZielElement` muss zusätzlich anbieten:

- Erkennung Signal/Blindziel,
- B-Claim setzen, prüfen und lösen, soweit es ein Signal ist,
- ZIF mit dem übergebenen Wert setzen und anschließend prüfen.

### 5.8 Anforderung und Auflösung über `Estw`

Empfohlene öffentliche Operationen:

```cpp
requestFahrstrasse(startElementId, zielElementId);
requestAufloesung(aufloeseElementId);
```

`Estw` ermittelt die passende Fahrstraße und liefert ein strukturiertes Ergebnis wie:

- angenommen,
- nicht gefunden,
- mehrdeutig,
- nicht in Status 0,
- Zulassung abgelehnt,
- Projektierung ungültig.

So bleibt die UI frei von Stellwerkslogik.

### 5.9 Annahmen des Architekturvorschlags

Folgende Punkte sind Empfehlungen und keine bereits dokumentierten Fachregeln:

1. Ein nach Status 1 auftretender Verlust einer zyklischen Voraussetzung wird verriegelt: Signale gehen auf Halt, die Fahrtstellung wird nicht automatisch erneut gesetzt und die Fahrstraße muss aufgelöst werden.
2. Fahrwegsignale werden nach bestätigter Beanspruchung und vor dem Stellen der Weichen einmalig mit HSV `-2` weißgeschaltet.
3. Bei mehreren aktiven Fahrstraßen am selben Auflöseelement wird nicht automatisch jede Fahrstraße aufgelöst; die Bedienung meldet zunächst Mehrdeutigkeit.
4. `besetzt` wird ohne zusätzliche fachliche Festlegung nicht als Zulassungskriterium verwendet.
5. Ein Fehler beim teilweisen Beanspruchen führt zu einer kontrollierten Rücknahme über das Owner-Ledger; er darf keine fremden Claims berühren.

Diese Annahmen müssen vor der Implementierung bestätigt werden.

---

## 6. Vorgeschlagener vollständiger Ablauf

### 6.1 Anforderung

1. Der Bediener wählt ein Startsignal und anschließend ein Zielsignal oder Blindziel.
2. `Estw` sucht genau eine passende Fahrstraße.
3. Die Anforderung wird nur akzeptiert, wenn deren Status 0 ist.
4. Die Rangierzulassung wird genau einmal geprüft.
5. Bei Ablehnung bleibt Status 0; es werden keine Claims gesetzt und die Anforderung wird nicht zyklisch wiederholt.
6. Bei Annahme erfolgt der Übergang zu Status 1.

### 6.2 Zustandsautomat

| Zustand | Eintrittsbedingung | Einmalige Aktion | Zyklische Prüfung und Folgezustand | Fehlerverhalten |
|---|---|---|---|---|
| Status 0 | Grundstellung | Keine | Nur auf neues Benutzerereignis Zulassung ausführen | Zulassung negativ: Status 0 ohne Nebenwirkungen |
| Status 1 | Zulassung einmalig positiv | Alle Rangier-Claims owner-basiert anfordern | Prüfen, ob alle eigenen Claims tatsächlich vorliegen; dann Status 2 | Bei unauflösbarem Claimkonflikt eigene Teilclaims kontrolliert zurücknehmen |
| Status 2 | Alle Claims bestätigt | Fahrwegsignale entsprechend bestätigter Fachregel weißschalten | Claims weiter prüfen; Weichen idempotent in Sollstellung befehlen; bei allen Sollstellungen Status 3 | Bei Claimverlust alle gesteuerten Signale auf Halt und Fehler verriegeln |
| Status 3 | Claims vorhanden, alle Weichen in Sollstellung | Jede Weiche genau einmal für diese Fahrstraße verschließen | Claims, Sollstellungen und eigene Verschlüsse prüfen; dann Status 4 | Bei Bedingungsverlust Signale auf Halt; keine weitere Freigabe |
| Status 4 | Alle vorherigen Bedingungen einschließlich Verschluss bestätigt | Startsignal genau einmal auf projektierte HSV `-1/-2`; ZIF auf Rangier | Claims, Sollstellungen und Verschlüsse zyklisch überwachen | Bei Verlust sofort Halt an Start- und Fahrwegsignalen; keine automatische Wiederfahrt |

Beim Eintritt in Status 4 muss anschließend geprüft werden, ob Startsignalstellung und ZIF tatsächlich angenommen wurden. Eine Ablehnung darf nicht durch wiederholtes zyklisches Stellen kaschiert werden, weil die Fahrtstellung laut Dokumentation nur einmal angefordert werden darf.

### 6.3 Auflösung

Die Auflösung muss aus jedem Status größer 0 möglich sein:

| Phase | Aktion/Prüfung |
|---|---|
| Halt anfordern | Startsignal einmalig auf Halt stellen; fachlich zu klärende Fahrwegsignale ebenfalls sichern |
| Halt prüfen | Zyklisch warten, bis das Startsignal tatsächlich eine Rangier-Haltstellung erreicht hat |
| Verschlüsse lösen | Nur die von dieser Fahrstraße registrierten Weichenverschlüsse einmal entfernen |
| Beanspruchungen lösen | Nur eigene A-/B-/C-/Gleis-Claims entfernen |
| Anzeigen bereinigen | ZIF löschen und weißgeschaltete Signale in Grundstellung bringen |
| Abschließen | Owner-Ledger leeren, Auflösungsphase beenden, Status auf 0 setzen |

Kann ein eigener Verschluss oder Claim nicht konsistent entfernt werden, darf Status 0 nicht stillschweigend gesetzt werden. Der Fehler muss sichtbar bleiben, damit keine scheinbar freie Fahrstraße mit verbliebener Sicherung entsteht.

---

## 7. Voraussichtlich betroffene Dateien

| Datei | Erwartete spätere Änderung |
|---|---|
| [structs.h](D:/csharp/CPP/estw/estw3/structs.h:26) | Typisierte Fahrstraßenarten und Rollen oder zentrale Konvertierungsfunktionen |
| [validate.cpp](D:/csharp/CPP/estw/estw3/validate.cpp:413) | Vollständige Rollen-/Elementtabelle, rangierspezifische Sollstellungen und Signalarten prüfen |
| [fahrstrasseneditor.cpp](D:/csharp/CPP/estw/estw3/fahrstrasseneditor.cpp:90) | Fehlerhafte Rollen und Sollstellungen bereits bei der Eingabe ablehnen |
| [estw/fs.h](D:/csharp/CPP/estw/estw3/estw/fs.h:39) | Typisierte Statuswerte, Projektionen, Owner-Ledger, Anforderungs- und Auflösungs-API |
| [estw/fs.cpp](D:/csharp/CPP/estw/estw3/estw/fs.cpp:19) | Vollständiger Zustandsautomat, Zulassung, Überwachung und Auflösung |
| [estw/estw.h](D:/csharp/CPP/estw/estw3/estw/estw.h:25) / [estw.cpp](D:/csharp/CPP/estw/estw3/estw/estw.cpp:22) | Fahrstraße anhand Start/Ziel anfordern und anhand Auflöseelement auflösen |
| [estw/element.h](D:/csharp/CPP/estw/estw3/estw/element.h:40) | Gemeinsame Claim-Datentypen oder typsichere Basisschnittstellen |
| [estw/gleis.h](D:/csharp/CPP/estw/estw3/estw/gleis.h:12) / `gleis.cpp` | Owner-basierter Ganz-Element-Claim |
| [estw/weiche.h](D:/csharp/CPP/estw/estw3/estw/weiche.h:28) / `weiche.cpp` | A-Claim, gezielter Stellbefehl, owner-basierter Verschluss, vollständiger Reset |
| [estw/signal.h](D:/csharp/CPP/estw/estw3/estw/signal.h:7) / `signal.cpp` | Unterelementart, Fahrstraßentyp-Prädikate, Befehlsergebnisse und owner-basierte Claims |
| [estw/blind.h](D:/csharp/CPP/estw/estw3/estw/blind.h:13) / `blind.cpp` | Konsistente ZIF-Anforderung und gegebenenfalls Besitzerinformation |
| [estw/aufloese.h](D:/csharp/CPP/estw/estw3/estw/aufloese.h:11) / `aufloese.cpp` | Wahrscheinlich nur Ereignis-/Anforderungsanbindung; keine eigene Fahrstraßenlogik |
| [lupeform.cpp](D:/csharp/CPP/estw/estw3/lupeform.cpp:855) / `lupeform.h` | Vollständige Runtime-Projektion, Start-/Zielbedienung, Auflöseklick, Übergabe von UEA und Weichenprogrammierungen |
| [lupe/lupeaufloese.cpp](D:/csharp/CPP/estw/estw3/lupe/lupeaufloese.cpp:23) | Auflösungsanforderung oder Callback anbinden |
| [tests/fahrstrassen_tests.cpp](D:/csharp/CPP/estw/estw3/tests/fahrstrassen_tests.cpp:39) | Ungültige Rangier-HSV korrigieren und neue Validierungsfälle ergänzen |
| [tests/aufloese_melder_tests.cpp](D:/csharp/CPP/estw/estw3/tests/aufloese_melder_tests.cpp:13) | Vom manuellen `setAn()` auf echte Statusanforderung umstellen |
| neue Datei `tests/rangierstrassenlogik_tests.cpp` | Zustands-, Fehler-, Überwachungs- und Auflösungstests |
| neue Dateien, z. B. `estw/fahrstrassenlogik.h/.cpp` | Optional: gemeinsame Engine und Rangier-/Zugstrategie sauber trennen |
| [CMakeLists.txt](D:/csharp/CPP/estw/estw3/CMakeLists.txt:133) | Neue Laufzeit- und Testdateien einbinden |

Eine Änderung des persistenten Datenbankschemas ist voraussichtlich nicht erforderlich, weil Rolle und `SollStellung` bereits gespeichert werden. Ressourcenänderungen sind ebenfalls nicht absehbar.

---

## 8. Risiken und offene Fragen

1. **Zeitpunkt und genauer Begriff der Weißschaltung:** Soll jedes Signal als Fahrwegelement HSV `-2` erhalten? Erfolgt dies nach bestätigten Claims, erst nach gestellten Weichen oder gemeinsam mit Status 4?

2. **Dunkelschaltung:** Ist HSV `-3` für einzelne Rangierstraßen-Fahrwegsignale vorgesehen, oder gilt ausschließlich Weißschaltung?

3. **Reaktion auf zyklischen Bedingungsverlust:** Soll der Status stehen bleiben, zurückfallen oder in einen eigenen Störzustand wechseln? Darf nach Wiederkehr der Voraussetzung automatisch weitergeschaltet werden? Wegen der einmaligen Fahrtstellung wird ein verriegelter Fehlerzustand empfohlen.

4. **Gleisfreimeldung:** Muss `besetzt` für Rangierstraßen geprüft werden? Falls ja: bei der Zulassung, vor Status 4 oder während der gesamten eingestellten Fahrstraße?

5. **Startsignalprogrammierung:** Muss `PRS` bereits statisch oder bei der Zulassung geprüft werden? Welche Kombinationen aus Unterelementart und HSV `-1/-2` sind zulässig?

6. **Zielprogrammierung:** Muss ein Signal- oder Blindziel `PRZ` besitzen, bevor ein Rangier-ZIF gesetzt werden darf? Das Verhalten ist derzeit zwischen `Signal` und `Blind` unterschiedlich.

7. **Mehrere Fahrstraßen mit identischem Start und Ziel:** Die Validierung verbietet diese Konstellation nicht. Es wird eine Eindeutigkeitsregel oder eine Bedienauswahl benötigt.

8. **Gemeinsames Auflöseelement:** Bestehende Tests erlauben mehrere Fahrstraßen am selben Auflöseelement. Es muss festgelegt werden, welche davon ein Klick auflöst.

9. **Weichen-Strang A:** Die Dokumentation verlangt A und B/C; das Laufzeitmodell kennt nur B und C. Die genaue Bedeutung und Darstellung von A muss festgelegt werden.

10. **Teilweise gesetzte Claims:** Ohne atomare Anforderung können mehrere Fahrstraßen einander mit Teilbeanspruchungen blockieren. Owner-Ledger und sichere Kompensation sind erforderlich.

11. **Verschlusszähler:** Das heutige Klemmen bei 0 verdeckt doppelte Freigaben. Eine Freigabe ohne eigenen Lock sollte als Fehler erkannt werden.

12. **Globale Rücksetzung:** `Estw::reset()` setzt aktuell zuerst die Elemente und danach die Fahrstraßen zurück: [estw.cpp:33](D:/csharp/CPP/estw/estw3/estw/estw.cpp:33). Bei einer owner-basierten Auflösung muss entweder zuerst die Fahrstraße entsichert oder ein ausdrücklich atomarer „Force Reset“ verwendet werden.

13. **Asynchroner Weichenmotor:** Die Weiche wird über einen eigenen Timer mit zufälligem Intervall bewegt. Zustandsmaschinentests benötigen eine deterministische Zeit- oder Motorschnittstelle.

14. **Dokumentationsabweichung:** [estw.md:181](D:/csharp/CPP/estw/estw3/.agents/estw.md:181) behauptet, `Fs::work()` prüfe laufend `isValid()`. Der Code tut dies nicht. Außerdem muss strukturelle Gültigkeit klar von einmaliger Zulassung und zyklischer Statusüberwachung getrennt werden.

15. **ZIF-Rücknahme und Fahrwegsignale bei Auflösung:** Beides fehlt in der dokumentierten Auflösungsfolge, ist aber für erneute Anforderung beziehungsweise sichere Grundstellung relevant.

16. **Zugstraßenregressionen:** Gemeinsame Element-APIs dürfen Zugstraßen nicht versehentlich nach Rangierregeln behandeln. Solange Zugzulassung und Flankenschutz offen sind, sollte Typ 2 nicht durch die neue Rangierpolicy laufen.

---

## 9. Empfohlener Umsetzungsplan

| Schritt | Ziel und vorgesehene Änderungen | Abhängigkeiten und Prüfungen | Erfolgskriterium |
|---|---|---|---|
| 1. Fachliche Entscheidungen schließen | Offene Punkte zu Weiß-/Dunkelschaltung, Gleisfreimeldung, Fehlerreaktion, PRS/PRZ, ZIF-Rücknahme und Mehrdeutigkeiten dokumentieren | Keine Codeabhängigkeit | Jeder Zustandsübergang besitzt eine eindeutige Fachregel |
| 2. Domänenwerte typisieren und validieren | Fahrstraßenart, Rollen, Status und Sollstellungen zentralisieren; Validator und Editor erweitern | Schritt 1 | Ungültige Rangier-HSV, Rollen und Elementkombinationen werden vor der Simulation abgewiesen |
| 3. Runtime-Projektion korrigieren | `SollStellung`, Rolle und Elementzeiger gemeinsam an `Fs` übergeben; UEA sowie PSSL/PSSR übernehmen | Schritt 2 | Jede Weiche und jedes Startsignal besitzt zur Laufzeit seine projektierte Sollstellung |
| 4. Element-APIs absichern | Owner-Claims, Weichen-A-Claim, deterministischer Stellbefehl, owner-basierter Verschluss, Signalprädikate und ZIF-Korrektur | Schritte 1–3 | Wiederholte Befehle sind idempotent; fremde Zustände können nicht freigegeben werden |
| 5. Rangier-Zustandsmaschine implementieren | Status 0–4, einmalige Eintrittsaktionen, kumulative Guards und Fehlerlatch in `Fs` beziehungsweise einer Policy | Schritt 4 | Ein deterministischer Test durchläuft 0→1→2→3→4 ausschließlich bei erfüllten Voraussetzungen |
| 6. Sichere Auflösung implementieren | Mehrphasige Haltprüfung, eigene Verschlüsse/Claims lösen, ZIF und Signale zurücksetzen | Schritt 5 | Kein Lock oder Claim wird vor bestätigtem Halt entfernt; danach erreicht die Fahrstraße Status 0 |
| 7. Bedienung anbinden | Start-/Zielauswahl, Fahrstraßensuche, Rückmeldungen und Auflöseklick über `Estw` | Schritte 5–6 | Eine Rangierstraße kann ohne fest codierte ID angefordert und aufgelöst werden |
| 8. Zyklische Störfälle testen | Claims verlieren, Weiche auffahren, Sollstellung verlieren, Verschluss verlieren, Signalsperre setzen | Schritte 5–7 | Start- und Fahrwegsignale gehen zuverlässig auf Halt; Fahrt wird nicht ungewollt erneut freigegeben |
| 9. Konflikt- und Zählertests ergänzen | Überlappende Routen, gemeinsame Auflöseelemente, doppelte Befehle und mehrfache Verschlüsse testen | Schritte 4–8 | Fremde Claims bleiben erhalten; Zähler stimmen nach jeder Auflösung |
| 10. Zugstraßenisolation absichern | Gemeinsame Engine testen, Zugpolicy aber bis zur fachlichen Klärung getrennt halten | Alle vorherigen Schritte | Typ 1 funktioniert vollständig; Typ 2 verwendet keine Rangierregeln und erleidet keine stille Verhaltensänderung |
| 11. Dokumentation angleichen | Tatsächliche APIs, Statusreaktionen und geklärte Fachregeln in den Markdown-Dateien nachführen | Abgeschlossene Implementierung | Dokumentation, Validator, Tests und Laufzeit verwenden dasselbe Modell |

## Kurzzusammenfassung

Die heutige Implementierung enthält Elementzustände und Datenstrukturen, aber keine funktionsfähige Rangierstraßenmechanik. Besonders kritisch sind der Verlust von `SollStellung`, der unabhängige Zustand `an`, fehlende Besitzerinformationen, die nicht deterministisch ansteuerbaren Weichen und die fehlende Bedien- und Auflösungsanbindung.

Empfohlen wird eine gemeinsame, typisierte Fahrstraßen-Zustandsmaschine mit rangierstraßenspezifischer Policy, owner-basierten Claims und Verschlüssen sowie einer getrennten Auflösungsphase. Vor der Implementierung müssen insbesondere Weißschaltung, Gleisfreimeldung, Verhalten bei zyklischem Bedingungsverlust und Mehrdeutigkeiten bei Start/Ziel beziehungsweise Auflöseelement fachlich geklärt werden.