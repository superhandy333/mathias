# Zugstrasse

Dieses Dokument beschreibt die Zustandsmechanik einer **Zugstrasse** in `estw3`. Die Beschreibung ist als fachliche Vorgabe für die Implementierung zu verstehen.

Für Signalarten, Halt- und Fahrtstellungen sowie die Ermittlung der Vorsignalgeschwindigkeit gelten die verbindlichen Vorgaben aus [signale.md](signale.md).

## Struktureller Aufbau und Definition in der Datenstruktur

Im Sinne des Datenmodells verknüpft eine **Fahrstrasse** (`Fahrstrasse`) mehrere **Elemente** (`Element`) eines Projekts über zugeordnete **Fahrstrassenelemente** (`FahrstrassenElement`). Jedes Fahrstrassenelement weist dem referenzierten Element eine spezifische funktionale Rolle (`FahrstrassenElementtyp`) innerhalb der Fahrstrasse zu.

### Bedingungen nach Elementtyp

Für die Projektierung eines Elements in einer **Zugstrasse** sind ausschließlich die folgenden Zuordnungen von Elementtyp zu Fahrstrassenelementtyp zulässig. `ElementTypId` bezeichnet dabei den Typ des referenzierten Elements (`Element.TypId`); die Rollen-IDs beziehen sich auf `FahrstrassenElement.TypId`.

- **Gleis (`ElementTypId = 1`):**
  - Ein Element vom Typ Gleis darf nur als **Fahrwegelement (`TypId = 1`)** oder **Flankenschutztransportelement (`TypId = 8`)** projektiert sein.
  - Seine `SollStellung` muss `0` sein.
- **Weiche (`ElementTypId = 2`):**
  - Ein Element vom Typ Weiche darf nur als **Fahrwegelement (`TypId = 1`)**, **Flankenschutzelement (`TypId = 5`)** oder **Flankenschutztransportelement (`TypId = 8`)** projektiert sein.
  - Seine `SollStellung` muss `-1` (links) oder `1` (rechts) sein.
- **Signal (`ElementTypId = 3`):**
  - Ein Element vom Typ Signal darf nur als **Fahrwegelement (`TypId = 1`)**, **Fahrstrassenstart (`TypId = 2`)**, **Fahrstrassenziel (`TypId = 3`)**, **Flankenschutzelement (`TypId = 5`)**, **Vorwegelement (`TypId = 6`)**, **Unterwegselement (`TypId = 7`)** oder **Flankenschutztransportelement (`TypId = 8`)** projektiert sein.
  - Als **Fahrwegelement (`TypId = 1`)** muss seine `SollStellung` `0` sein.
  - Als **Fahrstrassenstart (`TypId = 2`)** muss seine `SollStellung` im ganzzahligen Bereich von `1` bis einschließlich `16` liegen.
  - Als **Fahrstrassenziel (`TypId = 3`)** muss seine `SollStellung` `0` sein. Die projektierte `SollStellung` wird für diese Rolle im Fahrstrassenablauf nicht ausgewertet oder berücksichtigt.
  - Als **Flankenschutzelement (`TypId = 5`)** muss seine `SollStellung` `0` sein.
  - Als **Vorwegelement (`TypId = 6`)** muss seine `SollStellung` `0` sein. Die projektierte `SollStellung` wird für diese Rolle im Fahrstrassenablauf nicht ausgewertet oder berücksichtigt.
  - Als **Unterwegselement (`TypId = 7`)** muss seine `SollStellung` im ganzzahligen Bereich von `-3` bis einschließlich `-1` liegen.
  - Als **Flankenschutztransportelement (`TypId = 8`)** muss seine `SollStellung` `0` sein.
- **Blindziel (`ElementTypId = 4`):**
  - Ein Element vom Typ Blindziel darf nur als **Fahrwegelement (`TypId = 1`)**, **Fahrstrassenziel (`TypId = 3`)** oder **Flankenschutztransportelement (`TypId = 8`)** projektiert sein.
  - Seine `SollStellung` muss `0` sein. Als **Fahrstrassenziel (`TypId = 3`)** wird die projektierte `SollStellung` im Fahrstrassenablauf nicht ausgewertet oder berücksichtigt.
- **Auflöseelement (`ElementTypId = 5`):**
  - Ein Element vom Typ Auflöseelement darf nur als **Fahrwegelement (`TypId = 1`)**, **Auflöseelement (`TypId = 4`)** oder **Flankenschutztransportelement (`TypId = 8`)** projektiert sein.
  - Seine `SollStellung` muss `0` sein.

Jedes referenzierte Element muss zum selben Projekt wie die Zugstrasse gehören. Ein Element darf innerhalb derselben Zugstrasse nur einmal zugeordnet sein, auch bei unterschiedlichen Rollen. In unterschiedlichen Fahrstrassen darf dasselbe Element erneut verwendet werden.

Die Zuordnungen entsprechen der verbindlichen Tabelle in [datenstrukturen.md](datenstrukturen.md#zulässige-zuordnungen-zu-elementtypen). Für Referenzen, Eindeutigkeit, Projektzugehörigkeit und Sollstellungen gelten die Bedingungen aus [rules_isvalid_estwdata.md](rules_isvalid_estwdata.md). Die hier genannten Sollstellungen beschreiben die strukturelle Datenvalidität; weitere Bedingungen für die Zulassung und den Ablauf der Zugstrasse stehen in den folgenden Abschnitten.

### Bedingungen nach Fahrstrassenelementtyp

Für die datentechnische Gültigkeit und Konsistenz einer Zugstrasse gelten hinsichtlich ihrer Fahrstrassenelemente folgende verbindliche Bedingungen:

- **Fahrwegelement (`TypId = 1`):**
  - Eine Zugstrasse kann 0 oder eine beliebige Anzahl von Fahrwegelementen besitzen.
  - Das über das Fahrwegelement referenzierte Element muss vom Elementtyp **Gleis** (`ElementTypId = 1`), **Weiche** (`ElementTypId = 2`), **Signal** (`ElementTypId = 3`), **Blindziel** (`ElementTypId = 4`) oder **Auflöseelement** (`ElementTypId = 5`) sein.
- **Fahrstrassenstart (`TypId = 2`):**
  - Jede Zugstrasse muss **genau ein** Fahrstrassenstartelement besitzen.
  - Das über das Startelement referenzierte Element muss vom Elementtyp **Signal** (`ElementTypId = 3`) sein.
- **Fahrstrassenziel (`TypId = 3`):**
  - Jede Zugstrasse muss **genau ein** Fahrstrassenzielelement besitzen.
  - Das über das Zielelement referenzierte Element muss vom Elementtyp **Signal** (`ElementTypId = 3`) oder **Blindziel** (`ElementTypId = 4`) sein.
- **Auflöseelement (`TypId = 4`):**
  - Jede Zugstrasse muss **genau ein** Auflöseelement besitzen.
  - Das über das Auflöseelement referenzierte Element muss vom Elementtyp **Auflöseelement** (`ElementTypId = 5`) sein.
- **Flankenschutzelement (`TypId = 5`):**
  - Eine Zugstrasse kann 0 oder eine beliebige Anzahl von Flankenschutzelementen besitzen.
  - Sofern Flankenschutzelemente vorhanden sind, müssen die darüber referenzierten Elemente vom Elementtyp **Weiche** (`ElementTypId = 2`) oder **Signal** (`ElementTypId = 3`) sein.
- **Vorwegelement (`TypId = 6`):**
  - Eine Zugstrasse kann 0 oder 1 Vorwegelement besitzen.
  - Sofern ein Vorwegelement vorhanden ist, muss das darüber referenzierte Element vom Elementtyp **Signal** (`ElementTypId = 3`) sein.
- **Unterwegselement (`TypId = 7`):**
  - Eine Zugstrasse kann 0 oder eine beliebige Anzahl von Unterwegselementen besitzen.
  - Sofern ein Unterwegselement vorhanden ist, müssen die darüber referenzierten Elemente vom Elementtyp **Signal** (`ElementTypId = 3`) sein.
  - Ein als Unterwegselement projektiertes Signal (Fahrwegsignal) soll beim Einstellen der Fahrstrasse die für dieses Fahrstrassenelement projektierte `SollStellung` einnehmen.
- **Flankenschutztransportelement (`TypId = 8`):**
  - Eine Zugstrasse darf 0 oder eine beliebige Anzahl von Flankenschutztransportelementen besitzen.
  - Das über ein Flankenschutztransportelement referenzierte Element muss vom Elementtyp **Gleis** (`ElementTypId = 1`), **Weiche** (`ElementTypId = 2`), **Signal** (`ElementTypId = 3`), **Blindziel** (`ElementTypId = 4`) oder **Auflöseelement** (`ElementTypId = 5`) sein.

Diese strukturellen Festlegungen stellen sicher, dass jede Zugstrasse über eindeutige Start-, Ziel- und Auflösepunkte verfügt und die beteiligten Stellwerkselemente die dafür zulässigen Elementtypen besitzen.

## Zulassungsprüfung

Für eine Zugstrasse gelten die folgenden Zulassungsbedingungen. Die Zulassungsprüfung ist nur dann positiv, wenn **alle** Bedingungen erfüllt sind.

1. **Fahrstrassenziel aufgelöst:** Das Fahrstrassenziel muss aufgelöst sein. Das als Fahrstrassenziel projektierte **Signal** oder **Blindziel** darf keinen Zielfestlegemelder zeigen. Der Zustand des Zielfestlegemelders (ZIF) am Fahrstrassenzielelement wird bei der Zulassungsprüfung geprüft. Ist er gesetzt, ist die Zulassungsprüfung negativ.

2. **Startelement als Zugstraßenstart zugelassen:** Das als Fahrstrassenstart projektierte **Signal** muss in den Projektierungsdaten als Start einer Zugstrasse zugelassen sein (`PZS = 1`). Ist `PZS` nicht gesetzt, ist die Zulassungsprüfung negativ.

3. **Zielelement als Zugstraßenziel zugelassen:** Das als Fahrstrassenziel projektierte **Signal** oder **Blindziel** muss in den Projektierungsdaten als Ziel einer Zugstrasse zugelassen sein (`PZZ = 1`). Ist `PZZ` nicht gesetzt, ist die Zulassungsprüfung negativ.

4. **Gleiselement für Zugstraßen zugelassen:** Für jedes der Zugstrasse zugeordnete **Fahrwegelement**, dessen referenziertes Element vom Elementtyp **Gleis** ist, muss der Programmfall `PZ` an genau diesem Gleiselement gesetzt sein (`PZ = 1`). Ist `PZ` bei mindestens einem solchen Gleiselement nicht gesetzt, ist die Zulassungsprüfung negativ.

5. **Keine Beanspruchung der Fahrwegelemente, Auflöseelemente und Blindziele:** Alle zur Zugstrasse gehörenden **Fahrwegelemente**, das als **Auflöseelement** (Fahrstrassenelementtyp) projektierte **Auflöseelement** (Elementtyp) und gegebenenfalls das als Fahrstrassenziel projektierte **Blindziel** müssen frei von jeder bereits vorhandenen Beanspruchung sein. Ist mindestens eines dieser Elemente bereits beansprucht, ist die Zulassungsprüfung negativ.

6. **Keine Beanspruchung auf Strang B eines Vorwegelementes:** Ein zur Zugstrasse gehörendes **Vorwegelement (Signal)** muss frei von einer bereits vorhandenen Beanspruchung auf **Strang B** sein. Liegt dort eine solche Beanspruchung vor, ist die Zulassungsprüfung negativ.

7. **Keine Beanspruchung auf Strang A und Strang B von Unterwegselementen:** Jedes zur Zugstrasse gehörende **Unterwegselement (Signal)** muss sowohl auf **Strang A** als auch auf **Strang B** frei von jeder bereits vorhandenen Beanspruchung sein. Liegt an mindestens einem dieser Signale auf einem der beiden Stränge eine Beanspruchung vor, ist die Zulassungsprüfung negativ.

8. **Keine widersprechende Sperrlage einer Weiche:** Für alle zur Zugstrasse gehörenden **Weichen** als **Fahrwegelement** wird die für diese Zugstrasse projektierte `SollStellung` betrachtet. Eine solche Weiche darf nicht in einer Lage gesperrt sein, die dieser Sollstellung widerspricht. Ist eine Weiche beispielsweise in der Gegenlage zur benötigten Sollstellung gesperrt, ist die Zulassungsprüfung negativ. Als „gesperrt“ gilt eine Weiche sowohl bei projektierter Stellsperre durch einen gesetzten Programmfall `PSSL` oder `PSSR` als auch bei einer zur Laufzeit durch den Befehl `WUS` wirksamen Umstellsperre.

9. **Keine widersprechende Verschlusslage einer Weiche:** Für alle zur Zugstrasse gehörenden **Weichen** als **Fahrstrassenelement** wird die für diese Zugstrasse projektierte `SollStellung` betrachtet. Eine solche Weiche darf nicht in einer Lage verschlossen sein, die dieser Sollstellung widerspricht. Ist eine Weiche beispielsweise in der Gegenlage zur benötigten Sollstellung verschlossen, ist die Zulassungsprüfung negativ.

10. **Keine aufgefahrene Weiche:** Keine für die Zugstrasse benötigte **Weiche** als **Fahrwegelement** darf zum Zeitpunkt der Zulassungsprüfung aufgefahren sein. Ist mindestens eine dieser Weichen aufgefahren, ist die Zulassungsprüfung negativ – unabhängig davon, ob ihre aktuelle Lage ansonsten der projektierten `SollStellung` entspricht oder entsprechen könnte.

11. **Startsignal auf Strang A nicht beansprucht:** Am **Startsignal** darf auf **Strang A** keine bereits vorhandene Beanspruchung vorliegen. Liegt dort eine Beanspruchung vor, ist die Zulassungsprüfung negativ.

12. **Startsignal in Haltstellung:** Das **Startsignal** muss die **Haltstellung** zeigen. Zeigt es einen anderen Signalbegriff, ist die Zulassungsprüfung negativ.

13. **Keine Signalsperre am Startsignal oder an Unterwegssignalen:** Am **Startsignal** und an jedem als **Unterwegselement** projektierten **Signal** darf keine **Signalsperre (SS)** aktiv sein. Ist an mindestens einem dieser Signale eine Signalsperre aktiv, ist die Zulassungsprüfung negativ.

14. **Zielsignal auf Strang B nicht beansprucht:** Ist das Fahrstrassenziel ein **Signal**, darf an diesem **Zielsignal** auf **Strang B** keine bereits vorhandene Beanspruchung vorliegen. Liegt dort eine Beanspruchung vor, ist die Zulassungsprüfung negativ.

15. **Zielsperre am Zielsignal beachten:** Ist das Fahrstrassenziel ein **Signal**, muss für genau dieses **Zielsignal** mindestens eine der folgenden Voraussetzungen erfüllt sein: Der Programmfall `PZSP` ist am Zielsignal nicht gesetzt, oder `PZSP` ist dort gesetzt und das Zielsignal wird zur Laufzeit von keiner **aktiven Zugstrasse** als **Flankenschutzelement** verwendet. Eine Zugstrasse gilt hierfür als aktiv, wenn sie eingeschaltet ist (Status `> 0`). Die Zulassungsprüfung ist somit negativ, wenn `PZSP` am Zielsignal gesetzt ist und genau dieses Signal gleichzeitig in mindestens einer aktiven Zugstrasse als Flankenschutzelement verwendet wird. Eine bloße Projektierung des Signals als Flankenschutzelement in einer ausgeschalteten Zugstrasse genügt nicht. Bei einem Blindziel ist diese Bedingung nicht anzuwenden.

16. **Keine Befahrbarkeitssperre im befahrenen Bereich:** Bei keinem Element des **befahrenen Bereichs** der anzufordernden Zugstrasse darf eine **Befahrbarkeitssperre** aktiv sein. Für diese Prüfung umfasst der befahrene Bereich die Elemente in den Rollen **Fahrwegelement (`TypId = 1`)**, **Fahrstrassenstart (`TypId = 2`)**, **Fahrstrassenziel (`TypId = 3`)**, **Auflöseelement (`TypId = 4`)**, **Vorwegelement (`TypId = 6`)** und **Unterwegselement (`TypId = 7`)**. Ist bei mindestens einem dieser Elemente eine Befahrbarkeitssperre gesetzt, ist die Zulassungsprüfung negativ. Befahrbarkeitssperren an **Flankenschutzelementen (`TypId = 5`)** und **Flankenschutztransportelementen (`TypId = 8`)** werden für diese Bedingung nicht berücksichtigt. Maßgeblich ist die Rolle des Elements in der anzufordernden Zugstrasse.

Für die Bedingungen 5, 6, 7, 11 und 14 ist ausschließlich entscheidend, ob an dem jeweils geprüften Element beziehungsweise Strang bereits eine Beanspruchung vorhanden ist. Ihre Herkunft ist ohne Bedeutung; auch eine von der zu prüfenden Fahrstrasse selbst verursachte Beanspruchung gilt als vorhanden und führt zu einer negativen Zulassungsprüfung. Eine Zuordnung der Beanspruchung zu einer Fahrstrasse oder eine Referenzzählung wird für diese Prüfung nicht benötigt.

## Zustandsmodell

Die Zugstrasse durchläuft beim Einstellen mehrere aufeinanderfolgende Zustände. Der Übergang in den jeweils nächsten Zustand erfolgt erst, wenn die dafür genannten Voraussetzungen erfüllt sind. Dadurch werden angeforderte Aktionen zunächst an den beteiligten ESTW-Elementen ausgelöst und ihr tatsächlicher Zustand anschließend geprüft.

### Status 0 – Fahrstrasse ausgeschaltet

Die Fahrstrasse ist ausgeschaltet. Dies ist ihre Grundstellung.

Wird die Fahrstrasse gewählt, während sie sich in Status 0 befindet, wird die **Zulassungsprüfung (ZP) für Zugstrassen** durchgeführt. Die dafür geltenden zugstrassenspezifischen Bedingungen werden im Abschnitt **„Zulassungsprüfung“** dieses Dokuments beschrieben.

Nur bei positiver Zulassungsprüfung darf mit dem Einstellen der Fahrstrasse fortgefahren werden.
---------------------------------------------------------------------------------------------------------------------------
### Status 1 – Beanspruchungen anfordern

**Voraussetzung:** Die Zulassungsprüfung ist positiv abgeschlossen.

Die in diesem Abschnitt beschriebenen Beanspruchungen sind **Zugstraßenbeanspruchungen**, im Folgenden kurz **Beanspruchungen** genannt.

Die folgenden Regeln gelten für die Rollen **Fahrwegelement**, **Fahrstrassenstart**, **Fahrstrassenziel**, **Auflöseelement**, **Vorwegelement** und **Unterwegselement**. Für **Flankenschutzelemente** und **Flankenschutztransportelemente** werden in diesem Schritt keine Beanspruchungen nach diesen Regeln angefordert.

**Aktionen:**

- An allen **Fahrwegelementen**, **Blindzielen** und **Auflöseelementen** wird versucht, die für die Fahrstrasse erforderliche Beanspruchung zu setzen.
- Am **Startsignal** wird die Beanspruchung ausschließlich auf **Strang A** gesetzt. Die Beanspruchung auf **Strang B** wird durch diese Zugstrasse nicht verändert.
- Ist das **Fahrstrassenziel ein Signal**, wird dort die Beanspruchung ausschließlich auf **Strang B** gesetzt. Die Beanspruchung auf **Strang A** wird durch diese Zugstrasse nicht verändert.
- Ist das **Fahrstrassenziel ein Blindziel**, wird dort der gesamte Bereich von **Strang A bis Strang B** beansprucht. Der durchgehende Gleismelder umfasst beide Stränge.
- Am als **Auflöseelement** projektierten Element wird der gesamte Bereich von **Strang A bis Strang B** beansprucht. Der durchgehende Gleismelder umfasst beide Stränge.
- Ist ein Signal als **Fahrwegelement** im Sinne der Fahrstrasse gekennzeichnet, wird die Beanspruchung an diesem Signal sowohl auf **Strang A** als auch auf **Strang B** gesetzt. Dabei handelt es sich um ein Signal der **Gegenrichtung**.
- Ist ein Signal als **Unterwegselement** im Sinne der Fahrstrasse gekennzeichnet, wird die Beanspruchung an diesem Signal sowohl auf **Strang A** als auch auf **Strang B** gesetzt.
- Ist ein Signal als **Vorwegelement** im Sinne der Fahrstrasse gekennzeichnet, wird die Beanspruchung an diesem Signal ausschließlich auf **Strang B** gesetzt. Die Beanspruchung auf **Strang A** wird durch diese Zugstrasse weder gesetzt noch verändert. Diese Regel gilt sowohl bei einem **Blindziel** als auch bei einem **Signal** als projektiertem Fahrstrassenziel.
- Bei jeder als **Fahrwegelement** projektierten **Weiche** wird die Beanspruchung auf **Strang A** sowie auf demjenigen Strang gesetzt, der der projektierten `SollStellung` entspricht: **Strang B** bei `SollStellung = 1` (rechts), **Strang C** bei `SollStellung = -1` (links). Die Beanspruchung des jeweils anderen Strangs wird durch diese Zugstrasse nicht verändert.

Das Setzen einer Beanspruchung ist als Anforderung zu verstehen. Der Übergang zum nächsten Schritt darf erst erfolgen, wenn die erforderlichen Beanspruchungszustände an den oben genannten Elementen und Strängen tatsächlich vorliegen.

---------------------------------------------------------------------------------------------------------------------------
### Status 2 – Weichen in Sollstellung bringen

**Voraussetzung:** Die in Status 1 geforderten Beanspruchungen sind an den dort genannten Elementen und Strängen tatsächlich gesetzt.

**Aktion:** Alle als **Fahrwegelemente** projektierten **Weichen** werden in ihre für die Fahrstrasse projektierte `SollStellung` gebracht, sofern sie diese noch nicht eingenommen haben.

---------------------------------------------------------------------------------------------------------------------------
### Status 3 – Weichen verschließen

**Voraussetzungen:**

- Die Voraussetzungen aus Status 2 sind weiterhin erfüllt.
- Zusätzlich haben alle als **Fahrwegelemente** projektierten **Weichen** ihre projektierte `SollStellung` tatsächlich erreicht.

**Aktion:** Alle als **Fahrwegelemente** projektierten **Weichen** werden **verschlossen**.

---------------------------------------------------------------------------------------------------------------------------
### Status 4 – Fahrstrassenüberwachungsebene Rangierstrasse (FÜM blinkend)

**Voraussetzungen**

- Die Voraussetzungen aus Status 3 sind weiterhin erfüllt.
- Alle als **Fahrwegelemente** projektierten **Weichen** sind **verschlossen**.

**Aktion:**
- Der Fahrstrassenüberwachungsmelder am Startsignal wird auf blinkend gesetzt.
- Alle als **Unterwegssignale** projektierten **Signale** werden in ihre für die Fahrstrasse projektierte `SollStellung` gebracht. Das gilt nur dann, wenn die SollStellung -1, -2 oder -3 ist.
- Alle als **Flankenschutzelemente** projektierten **Weichen** werden in ihre für die Fahrstrasse projektierte `SollStellung` gebracht, sofern sie diese noch nicht eingenommen haben. Mit Erreichen dieser Stellung werden sie sofort **verschlossen**.
- An allen als **Flankenschutzelemente** projektierten **Signalen** wird das Flag `BEAF` gesetzt. Ein Signal darf zur Laufzeit gleichzeitig höchstens **einer Fahrstrasse** als Flankenschutzsignal dienen. Eine gleichzeitige Flankenbeanspruchung desselben Signals durch mehrere Fahrstrassen ist unzulässig. Die Projektierung desselben Signals als Flankenschutzelement in unterschiedlichen Fahrstrassen bleibt zulässig.

Solange ein Flankenschutz momentan nicht herstellbar ist, bleibt die Zugstrasse in **Status 4** und versucht die Anforderungen zyklisch erneut. Bereits erworbene eigene Beanspruchungen und Verschlussbeiträge bleiben bestehen. Fremde Flankenbindungen werden nicht übernommen oder aufgehoben. Eine bestehende Start-/Fahrtverwendung auf Strang A und ein eingeschaltetes Ersatzsignal verhindern den Erwerb einer Flankenbindung; bei `PZSP` verhindert zusätzlich eine aktive Zielverwendung den Erwerb. Eine reine Vorwegbeanspruchung auf Strang B ist keine Zielverwendung. An Flankensignale wird keine Haltanforderung gesendet.

Auch die Sollstellungsanforderungen an Unterwegssignale werden während dieses Wartens wiederholt. Nach Erreichen der Überwachungsebene wird ein Stellungsverlust nach den Regeln unter **Weitere Bedingungen** behandelt; dann erfolgt keine erneute Sollstellungsanforderung.

---------------------------------------------------------------------------------------------------------------------------
### Status 5 – Fahrstrassenüberwachungsebene Zugstrasse (FÜM Ruhelicht)

**Voraussetzungen:**

- Die Voraussetzungen aus Status 4 sind weiterhin erfüllt.
- Alle als **Flankenschutzelemente** projektierten **Weichen** sind in ihrer projektierten `SollStellung` verschlossen.
- Alle als **Flankenschutzelemente** projektierten **Signale** sind in Haltstellung.
- Alle als **Flankenschutzelemente** projektierten **Signale** mit gesetztem Programmfall `PZSP` werden momentan nicht als Ziel einer Rangier- oder Zugstrasse beansprucht.
- Alle als **Startsignal**, **Zielsignal**, **Fahrwegelement**, **Blindziel**, **Auflöseelement**, **Unterwegselement**, **Vorwegelement** und **Flankenschutztransportelement** projektierten **Elemente** sind nicht besetzt.
- Alle als **Flankenschutzelemente** projektierten **Signale** sind nicht besetzt.
- Alle als **Unterwegselemente** projektierten Signale (Fahrwegsignale) haben ihre jeweils für das Fahrstrassenelement projektierte `SollStellung` erreicht.

**Aktion:**
- Der Fahrstrassenüberwachungsmelder am Startsignal wird auf dauerhaft an (Ruhelicht) gesetzt.
- Der Zielfestlegemelder am Ziel wird gesetzt (Ruhelicht).

---------------------------------------------------------------------------------------------------------------------------
### Status 6 – Startsignal in Fahrtstellung bringen

**Voraussetzungen:**

- Die Voraussetzungen aus Status 5 sind weiterhin erfüllt.
- Der Fahrstrassenüberwachungsmelder am Startsignal ist dauerhaft an.
- Der Zielfestlegemelder am Ziel ist gesetzt (Ruhelicht).

**Aktionen:**

- Beim Erreichen von **Status 6** wird am **Startsignal** einmalig die für die Fahrstrasse projektierte Sollstellung, also die **Fahrtstellung**, angefordert. Die tatsächlich erreichte Fahrtstellung wird nicht geprüft und ist keine Voraussetzung für diesen Status. Wird die Anforderung nicht umgesetzt, führt dies allein weder zu einer erneuten Fahrtanforderung noch zu einem Rückfall oder Abbruch der Zugstrasse.
- Zyklisch wird in diesem Status die Vorsignalgeschwindigkeit des Startsignals gemäß [signale.md](signale.md#vorsignalgeschwindigkeit) ermittelt und eingetragen. Als Quelle dient das **Vorwegsignal**, bei dessen Fehlen das **Zielsignal**. Ist kein Vorwegsignal vorhanden und das Ziel ein **Blindziel**, wird `VSV = 0` eingetragen. Eine Hauptsignalgeschwindigkeit der Quelle von `-3` bis `-1` ergibt `VSV = 0`; Werte von `0` bis `16` werden unverändert übernommen. Die zyklische Ermittlung und Aktualisierung reicht aus; ein vorheriger oder gleichzeitiger Eintrag der VSV ist keine Voraussetzung für die Fahrtfreigabe des Startsignals.

---------------------------------------------------------------------------------------------------------------------------
## Auflösung der Zugstrasse

Die Auflösung der Fahrstrasse wird durch **Anklicken des zugehörigen Auflöseelements** gestartet.

Die Auflösung erfolgt in einer festen Reihenfolge:

1. Zunächst wird am **Startsignal und an allen Unterwegssignalen die Haltstellung angefordert**.
2. Danach wird geprüft, ob das Startsignal **und alle Unterwegssignale** die **Haltstellung tatsächlich erreicht** haben (`HSV == 0`). Solange dies nicht der Fall ist, darf die weitere Auflösung nicht durchgeführt werden. Insbesondere ist Kennlicht keine Haltbestätigung.
3. Erst nachdem die Haltstellung erreicht wurde, wird der **Verschluss der zur Fahrstrasse gehörenden Weichen entfernt**. Das gilt auch für Flankenschutzweichen. "Verschluss wird entfernt" heisst: Der Verschlusszähler der Weiche wird um 1 dekrementiert.
4. Anschließend werden die durch die Fahrstrasse gesetzten **Beanspruchungen entfernt**. Dazu gehört ausdrücklich auch die Rücknahme von `BEAF` an den durch diese Fahrstrasse als Flankenschutzsignal beanspruchten Signalen.
5. Der FÜM am Startsignal erlischt.
6. Der ZIF am Ziel erlischt.
7. Abschließend wird der Status der Fahrstrasse wieder auf **Status 0 – Fahrstrasse ausgeschaltet** gesetzt.

Damit gilt insbesondere die fachliche Sicherheitsregel: **Weichenverschluss und Beanspruchungen dürfen bei der Fahrstrassenauflösung erst entfernt werden, nachdem Startsignal und alle Unterwegssignale nachweislich die Haltstellung erreicht haben.** Die Auflösung ist aus jeder aktiven Einstellphase möglich. Zurückgenommen werden ausschließlich tatsächlich erworbene eigene Verschlussbeiträge und Flankenbindungen; fremde Beiträge bleiben erhalten. Nach vollständiger Auflösung wird die Wiederholungssperre des Startsignals für eine neue Einstellung zurückgesetzt.

## Weitere Bedingungen

- Fällt während der Fahrstellung des Startsignals eines der Unterwegssignale auf Halt, so fällt auch das Startsignal auf Halt. Eine Wiederholung der Fahrtstellung tritt nicht ein, solange die Fahrstrasse nicht wieder vollständig aufgelöst wurde (Wiederholungssperre).

- Fällt eine Bedingung für den Zustand "Fahrstrassenüberwachungsebene Zugstrasse" weg, so erlischt der FÜM am Startsignal. Die Fahrstrasse bleibt bis zum Zeitpunkt festgelegt, d.h. der ZIF am Ziel bleibt weiterhin sichtbar.

- Bei Verlust einer bereits erforderlichen Statusvoraussetzung werden Startsignal und Unterwegssignale auf Halt angefordert. Der Verlust wird bis zur vollständigen Auflösung gespeichert. Der zuletzt erreichte Status bleibt erhalten; es gibt keine automatische Wiederfreigabe. Auch nach Wiederkehr der Bedingungen bleiben die Signale auf Halt und der FÜM aus. Beanspruchungen, eigene Verschlussbeiträge, Flankenbindungen und ein bereits gesetzter Ziel-ZIF bleiben bis zur geregelten Auflösung erhalten. Fehlende Bedingungen für einen erstmals bevorstehenden Übergang (beispielsweise ein noch nicht freier Transportbereich in Status 4) sind dagegen ein Wartegrund.

## Noch zu ergänzende Punkte

Die zulässigen Element-/Rollenzuordnungen, die rollenabhängigen Sollstellungswerte der Signale und die Bedingungen der Zulassungsprüfung sind beschrieben. Der Haltbegriff ist für Zug- und Rangierstrassen einheitlich auf `HSV == 0` festgelegt; Kennlicht (`HSV = -3`) ist kein Haltbegriff.

Für Flankenweichen ist bereits festgelegt, dass sie ab **Status 4** in ihre Sollstellung gebracht und nach deren tatsächlichem Erreichen verschlossen werden. **Status 5** verlangt die verschlossenen Flankenweichen in Sollstellung, die Haltstellung der Flankensignale, die dort beschriebene Freiprüfung einschließlich der Flankenschutztransportelemente und die tatsächlich erreichten Sollstellungen der Unterwegssignale. Anschließend wird der FÜM auf Ruhelicht gesetzt. **Status 6** fordert bei weiterhin erfüllten Voraussetzungen aus Status 5 und dauerhaft eingeschaltetem FÜM die Fahrtstellung des Startsignals an und aktualisiert zyklisch dessen Vorsignalgeschwindigkeit. Für die Weichenverschlüsse gilt der gemeinsame Verschlusszähler gemäß [weiche.md](weiche.md).

Für die folgenden fachlichen Festlegungen und Präzisierungen gilt dieser Klärungsstand:

1. **Bezug der Statusvoraussetzungen – erledigt:** Die Statusfolge **0 bis 6** ist beschrieben. **Status 5** verlangt nun die weiterhin erfüllten Voraussetzungen aus **Status 4**; der bisherige Selbstverweis ist korrigiert. **Status 6** übernimmt die Voraussetzungen aus Status 5 und verlangt zusätzlich den dauerhaft eingeschalteten FÜM. Das blinkende FÜM-Anzeigebild ist in Status 4 als Aktion beschrieben und wird nicht als Voraussetzung in Status 5 übernommen.

2. **Herstellung und Bindung des Signalflankenschutzes – Statusprüfung und Freigabe geklärt, Schutzfunktionen gesondert zu behandeln:** In **Status 4** wird an allen als Flankenschutzelemente projektierten Signalen `BEAF` gesetzt. **Status 5** prüft, ob sich diese Signale tatsächlich in Haltstellung (`HSV == 0`) befinden. Eine Haltanforderung an die Flankensignale ist hierfür nicht vorgesehen; ihre Haltstellung wird ausschließlich als Statusvoraussetzung geprüft. Ein Signal kann gleichzeitig nur einer Fahrstrasse Flankenschutz bieten. Bei deren Auflösung wird `BEAF` im Schritt **Beanspruchungen entfernen** zurückgenommen. Der Schutz gegen das Einschalten des Ersatzsignals bei `BEAF = true` ist Gegenstand der noch zu erstellenden Funktionsbeschreibung **Ersatzsignal**. Die Schutzbindung gegen Fahrt wird gesondert untersucht und hier noch nicht festgelegt.

3. **Konflikte und nicht herstellbarer Flankenschutz – geklärt:** Die Zugstrasse wartet in Status 4 und prüft zyklisch erneut. Fremde Flankenbindungen und bestehende Fahrtverwendungen werden nicht übernommen. Bei `PZSP` wird eine aktive Zielverwendung berücksichtigt. Gleichliegende Weichenverschlüsse dürfen gemeinsam genutzt werden; jede Fahrstrasse verwaltet und entfernt nur ihren eigenen Zählerbeitrag.

4. **Flankenschutztransportelemente:** Die Freiprüfung in Status 5 und die Ausnahme von der Befahrbarkeitssperrenprüfung in ZP 16 sind festgelegt. Offen ist, ob diese Elemente zusätzlich gegen andere Nutzungen gebunden werden müssen. Falls eine solche Bindung erforderlich ist, fehlen Erwerb, Konfliktprüfung und Freigabe. Aus Status 1 folgt keine Beanspruchung dieser Rolle.

5. **Unterwegssignale – geklärt:** Die Sollstellungen `-3`, `-2` und `-1` werden in Status 4 zyklisch angefordert und vor Status 5 tatsächlich geprüft. Ein späterer Stellungsverlust führt zu Haltanforderungen an Start und Unterwegs sowie dauerhaft ausgeschaltetem FÜM bis zur Auflösung. Bei der Auflösung wird die Haltbestätigung aller Unterwegssignale abgewartet; Kennlicht zählt nicht als Halt.

6. **Fahrtfreigabe des Startsignals – Anforderung und fehlende Stellungsprüfung geklärt: - erledigt** Die Fahrtstellung wird beim Erreichen von **Status 6** einmalig angefordert und nicht zyklisch erneut angefordert. Die tatsächlich erreichte Fahrtstellung wird nicht geprüft. Auch eine nicht umgesetzte Anforderung löst allein keine Wiederholung, keinen Rückfall und keinen Abbruch der Zugstrasse aus. Die allgemeine Regel in [fahrstrasse.md](fahrstrasse.md) unterscheidet zwischen Zugstrassen (Status 6) und Rangierstrassen (Status 4); der bisherige Anschlusswiderspruch ist damit beseitigt. Die gesonderten Schutzbedingungen zu Signalsperre, Ersatzsignal und Flankenbeanspruchung bleiben davon unberührt.

7. **Vorsignalgeschwindigkeit – erledigt:** Quelle und Wertumsetzung sind in [signale.md](signale.md#vorsignalgeschwindigkeit) festgelegt und in **Status 6** berücksichtigt: Vorwegsignal, ersatzweise Zielsignal; bei Blindziel ohne Vorwegsignal sowie bei negativer Quell-HSV gilt `VSV = 0`, Quell-HSV von `0` bis `16` wird unverändert übernommen. Die VSV wird in Status 6 zyklisch ermittelt und eingetragen. Eine gültige VSV muss nicht bereits vor oder gleichzeitig mit der Fahrtfreigabe vorliegen; die zyklische Ermittlung und Aktualisierung reicht aus.

8. **Zielfestlegemelder und Melderrücknahme – erledigt:** Zug-ZIF und FÜM-Ruhelicht werden beim Erreichen von Status 5 gesetzt. Bei Überwachungsverlust erlischt der FÜM; der Ziel-ZIF bleibt erhalten. Bei geregelter Auflösung werden zuerst die Sicherungsbeiträge, dann FÜM und ZIF zurückgenommen. Eine neue Einstellung setzt einen aufgelösten Ziel-ZIF voraus.

9. **Überwachungsverlust und Wiederfreigabe – geklärt:** Start- und Unterwegssignale werden auf Halt angefordert, der FÜM erlischt und der zuletzt erreichte Status bleibt mit gespeichertem Überwachungsverlust erhalten. Eigene Sicherungsbeiträge und ein gesetzter Ziel-ZIF bleiben bestehen. Bis zur vollständigen Auflösung erfolgt keine Wiederfreigabe, auch nicht nach Freimeldung oder sonstiger Wiederkehr der Bedingungen.

10. **Auflösung und Flankenschutzfreigabe – geklärt:** Start und Unterwegs auf Halt anfordern und alle Haltbestätigungen abwarten; anschließend eigene Weichenverschlussbeiträge einschließlich Flanke entfernen, eigene Beanspruchungen einschließlich `BEAF` zurücknehmen, FÜM und Zug-ZIF löschen und Status 0 setzen. Das gilt auch während des Einstellens. Fremde und noch nicht erworbene Beiträge werden nicht zurückgenommen.
