# Rangierstrasse

Dieses Dokument beschreibt die Zustandsmechanik einer **Rangierstrasse** in `estw3`. Die Beschreibung ist als fachliche Vorgabe für die Implementierung zu verstehen.

Für Signalarten sowie die bei Rangierstrassen gültigen Halt- und Fahrtstellungen gelten die verbindlichen Vorgaben aus [signale.md](signale.md), insbesondere der Abschnitt [Hauptsignalgeschwindigkeit](signale.md#hauptsignalgeschwindigkeit).

## Struktureller Aufbau und Definition in der Datenstruktur

Im Sinne des Datenmodells verknüpft eine **Fahrstrasse** (`Fahrstrasse`) mehrere **Elemente** (`Element`) eines Projekts über zugeordnete **Fahrstrassenelemente** (`FahrstrassenElement`). Jedes Fahrstrassenelement weist dem referenzierten Element eine spezifische funktionale Rolle (`FahrstrassenElementtyp`) innerhalb der Fahrstrasse zu.

Für die datentechnische Gültigkeit und Konsistenz einer Rangierstrasse gelten hinsichtlich ihrer Fahrstrassenelemente folgende verbindliche Bedingungen:

- **Fahrwegelement (`TypId = 1`):**
  - Eine Rangierstrasse kann 0 oder eine beliebige Anzahl von Fahrwegelementen besitzen.
  - Das über das Fahrwegelement referenzierte Element muss vom Elementtyp **Gleis** (`ElementTypId = 1`), **Weiche** (`ElementTypId = 2`), **Signal** (`ElementTypId = 3`), **Blindziel** (`ElementTypId = 4`) oder **Auflöseelement** (`ElementTypId = 5`) sein.
- **Fahrstrassenstart (`TypId = 2`):**
  - Jede Rangierstrasse muss **genau ein** Fahrstrassenstartelement besitzen.
  - Das über das Startelement referenzierte Element muss vom Elementtyp **Signal** (`ElementTypId = 3`) sein.
- **Fahrstrassenziel (`TypId = 3`):**
  - Jede Rangierstrasse muss **genau ein** Fahrstrassenzielelement besitzen.
  - Das über das Zielelement referenzierte Element muss vom Elementtyp **Signal** (`ElementTypId = 3`) oder **Blindziel** (`ElementTypId = 4`) sein.
- **Auflöseelement (`TypId = 4`):**
  - Jede Rangierstrasse muss **genau ein** Auflöseelement besitzen.
  - Das über das Auflöseelement referenzierte Element muss vom Elementtyp **Auflöseelement** (`ElementTypId = 5`) sein.
- **Flankenschutzelement (`TypId = 5`):**
  - Eine Rangierstrasse darf **kein** Flankenschutzelement besitzen.
- **Vorwegelement (`TypId = 6`):**
  - Eine Rangierstrasse darf **kein** Vorwegelement besitzen.
- **Unterwegselement (`TypId = 7`):**
  - Eine Rangierstrasse darf **kein** Unterwegselement besitzen.
- **Flankenschutztransportelement (`TypId = 8`):**
  - Eine Rangierstrasse darf **kein** Flankenschutztransportelement besitzen.

Diese strukturellen Festlegungen stellen sicher, dass jede Rangierstrasse über eindeutige Start-, Ziel- und Auflösepunkte verfügt und die beteiligten Stellwerkselemente die dafür zulässigen Elementtypen besitzen.

## 1. Zulassungsprüfung

Für eine Rangierstrasse gelten die folgenden Zulassungsbedingungen. Die Zulassungsprüfung ist nur dann positiv, wenn **alle** Bedingungen erfüllt sind.

Der Zustand des Zielfestlegemelders (ZIF) am Fahrstrassenzielelement wird bei der Zulassungsprüfung **nicht abgefragt** und beeinflusst deren Ergebnis nicht.

1. **Startelement als Rangierstraßenstart zugelassen:** Das als Fahrstrassenstart projektierte **Signal** muss in den Projektierungsdaten als Start einer Rangierstrasse zugelassen sein (`PRS = 1`). Ist `PRS` nicht gesetzt, ist die Zulassungsprüfung negativ.
2. **Zielelement als Rangierstrassenziel zugelassen:** Das als Fahrstrassenziel projektierte **Signal oder Blindziel** muss in den Projektierungsdaten als Ziel einer Rangierstrasse zugelassen sein (`PRZ = 1`). Ist `PRZ` nicht gesetzt, ist die Zulassungsprüfung negativ.
3. **Gleiselement für Rangierstrassen zugelassen:** Für jedes der Rangierstrasse zugeordnete **Fahrstrassenelement**, dessen referenziertes Element vom Elementtyp **Gleis** ist, muss der Programmfall `PR` an genau diesem Gleiselement gesetzt sein (`PR = 1`). Ist `PR` bei mindestens einem solchen Gleiselement nicht gesetzt, ist die Zulassungsprüfung negativ.
4. **Keine Beanspruchung der Fahrwegelemente, Auflöseelemente und Blindziele:** Alle zur Rangierstrasse gehörenden **Fahrwegelemente**, **Auflöseelemente** und **Blindziele** müssen frei von jeder bereits vorhandenen Beanspruchung sein. Ist mindestens eines dieser Elemente bereits beansprucht, ist die Zulassungsprüfung negativ.
5. **Keine widersprechende Sperrlage einer Weiche:** Für alle zur Rangierstrasse gehörenden **Weichen** wird die für diese Rangierstrasse projektierte `SollStellung` betrachtet. Eine Weiche darf nicht in einer Lage gesperrt sein, die dieser Sollstellung widerspricht. Ist eine Weiche beispielsweise in der Gegenlage zur benötigten Sollstellung gesperrt, ist die Zulassungsprüfung negativ. Als „gesperrt“ gilt eine Weiche sowohl bei projektierter Stellsperre durch einen gesetzten Programmfall `PSSL` oder `PSSR` als auch bei einer zur Laufzeit durch den Befehl `WUS` wirksamen Umstellsperre.
6. **Keine widersprechende Verschlusslage einer Weiche:** Für alle zur Rangierstrasse gehörenden **Weichen** wird die für diese Rangierstrasse projektierte `SollStellung` betrachtet. Eine Weiche darf nicht in einer Lage verschlossen sein, die dieser Sollstellung widerspricht. Ist eine Weiche beispielsweise in der Gegenlage zur benötigten Sollstellung verschlossen, ist die Zulassungsprüfung negativ.
7. **Keine aufgefahrene Weiche:** Keine für die Rangierstrasse benötigte **Weiche** darf zum Zeitpunkt der Zulassungsprüfung aufgefahren sein. Ist mindestens eine dieser Weichen aufgefahren, ist die Zulassungsprüfung negativ – unabhängig davon, ob ihre aktuelle Lage ansonsten der projektierten `SollStellung` entspricht oder entsprechen könnte.
8. **Startsignal auf Strang A nicht beansprucht:** Am **Startsignal** darf auf **Strang A** keine bereits vorhandene Beanspruchung vorliegen. Liegt dort eine Beanspruchung vor, ist die Zulassungsprüfung negativ.
9. **Startsignal in Haltstellung:** Das **Startsignal** muss die **Haltstellung** zeigen. Zeigt es einen anderen Signalbegriff, ist die Zulassungsprüfung negativ.
10. **Keine Signalsperre am Startsignal:** Am **Startsignal** darf keine **Signalsperre (SS)** aktiv sein. Ist das Startsignal durch die Signalsperre gesperrt, ist die Zulassungsprüfung negativ.
11. **Zielsignal auf Strang B nicht beansprucht:** Ist das Fahrstrassenziel ein **Signal**, darf an diesem **Zielsignal** auf **Strang B** keine bereits vorhandene Beanspruchung vorliegen. Liegt dort eine Beanspruchung vor, ist die Zulassungsprüfung negativ.
12. **Zielsperre am Zielsignal beachten:** Ist das Fahrstrassenziel ein **Signal**, muss für genau dieses **Zielsignal** mindestens eine der folgenden Voraussetzungen erfüllt sein: Der Programmfall `PZSP` ist am Zielsignal nicht gesetzt, oder `PZSP` ist dort gesetzt und das Zielsignal wird zur Laufzeit von keiner **aktiven Zugstrasse** als **Flankenschutzelement** verwendet. Eine Zugstrasse gilt hierfür als aktiv, wenn sie eingeschaltet ist (Status `> 0`). Die Zulassungsprüfung ist somit negativ, wenn `PZSP` am Zielsignal gesetzt ist und genau dieses Signal gleichzeitig in mindestens einer aktiven Zugstrasse als Flankenschutzelement verwendet wird. Eine bloße Projektierung des Signals als Flankenschutzelement in einer ausgeschalteten Zugstrasse genügt nicht. Bei einem Blindziel ist diese Bedingung nicht anzuwenden.
13. **Keine Befahrbarkeitssperre an Fahrstrassenelementen:** Bei **keinem Fahrstrassenelement** der anzufordernden Fahrstrasse darf eine **Befahrbarkeitssperre** aktiv sein. Ist bei mindestens einem Fahrstrassenelement eine Befahrbarkeitssperre gesetzt, ist die Zulassungsprüfung negativ.

Für die Bedingungen 4, 8 und 11 ist ausschließlich entscheidend, ob an dem jeweils geprüften Element beziehungsweise Strang bereits eine Beanspruchung vorhanden ist. Ihre Herkunft ist ohne Bedeutung; auch eine von der zu prüfenden Fahrstrasse selbst verursachte Beanspruchung gilt als vorhanden und führt zu einer negativen Zulassungsprüfung. Eine Zuordnung der Beanspruchung zu einer Fahrstrasse oder eine Referenzzählung wird für diese Prüfung nicht benötigt.

## 2. Zustandsmodell

Die Rangierstrasse durchläuft beim Einstellen die Zustände **0 bis 4**. Der Übergang in den jeweils nächsten Zustand erfolgt erst, wenn die dafür genannten Voraussetzungen erfüllt sind. Dadurch werden angeforderte Aktionen zunächst an den beteiligten ESTW-Elementen ausgelöst und ihr tatsächlicher Zustand anschließend geprüft.

### Status 0 – Fahrstrasse ausgeschaltet

Die Fahrstrasse ist ausgeschaltet. Dies ist ihre Grundstellung.

Wird die Fahrstrasse gewählt, während sie sich in Status 0 befindet, wird die **Zulassungsprüfung (ZP)** durchgeführt. Die dafür geltenden rangierstrassenspezifischen Bedingungen sind im Abschnitt **„Zulassungsprüfung“** dieses Dokuments beschrieben.

Nur bei positiver Zulassungsprüfung darf mit dem Einstellen der Fahrstrasse fortgefahren werden.

### Status 1 – Beanspruchungen anfordern

**Voraussetzung:** Die Zulassungsprüfung ist positiv abgeschlossen.

**Aktionen:**

- An allen **Fahrwegelementen** wird versucht, die für die Fahrstrasse erforderliche Rangierstrassen-Beanspruchung (im Folgenden nur "Beanspruchung") zu setzen.
- Ist ein **Signal** als **Fahrwegelement** im Sinne der Fahrstrasse gekennzeichnet, wird die Beanspruchung an diesem Signal sowohl auf **Strang A** als auch auf **Strang B** gesetzt. Dabei handelt es sich um ein Signal der **Gegenrichtung**.
- Am **Startsignal** wird die Beanspruchung auf **Strang A** gesetzt.
- Ist das Fahrstrassenziel ein **Zielsignal**, wird dort die Beanspruchung auf **Strang B** gesetzt.
- Ist das Fahrstrassenziel ein **Blindziel**, wird dort die Beanspruchung auf **Strang A** und **Strang B** gesetzt.
- Am **Auflöseelement** wird die Beanspruchung gesetzt.
- Bei jeder zur Fahrstrasse gehörenden **Weiche** wird die Beanspruchung auf **Strang A** sowie auf demjenigen Strang gesetzt, der der projektierten `SollStellung` entspricht. Abhängig von der Sollstellung ist dies **Strang B oder Strang C**.

Das Setzen einer Beanspruchung ist als Anforderung zu verstehen. Der Übergang zum nächsten Schritt darf erst erfolgen, wenn die erforderlichen Beanspruchungszustände an den Elementen tatsächlich vorliegen.

### Status 2 – Weichen in Sollstellung bringen

**Voraussetzung:** An allen Elementen der Fahrstrasse sind die Beanspruchungen entsprechend den Regeln aus Status 1 tatsächlich gesetzt.

**Aktion:** Alle zur Fahrstrasse gehörenden **Weichen** werden in ihre für die Fahrstrasse projektierte `SollStellung` gebracht, sofern sie diese noch nicht eingenommen haben.

### Status 3 – Weichen verschließen

**Voraussetzungen:**

- Die Voraussetzungen aus Status 2 sind weiterhin erfüllt.
- Zusätzlich haben alle zur Fahrstrasse gehörenden **Weichen** ihre projektierte `SollStellung` tatsächlich erreicht.

**Aktion:** Alle zur Fahrstrasse gehörenden Weichen werden **verschlossen**.

### Status 4 – Rangierstrasse eingestellt

**Voraussetzungen:**

- Die Voraussetzungen aus Status 3 sind weiterhin erfüllt.
- Zusätzlich sind alle zur Fahrstrasse gehörenden **Weichen** tatsächlich verschlossen.

**Aktionen:**

- Das **Startsignal** wird in die für die Fahrstrasse projektierte Sollstellung, also die **Fahrtstellung**, gebracht.
- Der Zielfestlegemelder (ZIF) am Fahrstrassenzielelement wird angezeigt.

**Bemerkung:** Im realen ESTW existiert für Rangierstrassen kein ZIF. Im Projekt estw3 existiert er.

Status 4 beschreibt damit den Zustand, in dem Fahrweg und Weichen für die Rangierstrasse beansprucht, die Weichen korrekt gestellt und verschlossen sind. Das entspricht bei Zugstrassen dem Überwachungsniveau FÜM blinkend.

Ab Status 4 wird zyklisch überprüft, ob die Voraussetzungen aus Status 3 weiterhin erfüllt sind und alle Weichen verschlossen sind. Sind diese Status-4-Voraussetzungen nicht mehr erfüllt, wird am Startsignal die Haltstellung angefordert, der Zielfestlegemelder (ZIF) am Fahrstrassenzielelement zurückgenommen und die aktive Rangierstrasse fällt auf Status 3 zurück. Die Rangierstrasse bleibt dabei aktiv; Weichenverschlüsse und Beanspruchungen werden nicht aufgehoben. Die ZIF-Rücknahme bei Status 3 erfolgt beim Rückfall aus Status 4, nicht beim erstmaligen Erreichen von Status 3 während des Einstellens. Sind anschließend die Voraussetzungen für Status 4 wieder erfüllt und ist die Rangierstrasse weiterhin aktiv, wird das Startsignal erneut in die projektierte Fahrtstellung gebracht, der Ziel-ZIF wieder gesetzt und Status 4 wieder erreicht.

## 3. Auflösung der Rangierstrasse

Die Auflösung der Fahrstrasse wird durch **Anklicken des zugehörigen Auflöseelements** gestartet.

Die Auflösung erfolgt in einer festen Reihenfolge:

1. Zunächst wird am **Startsignal die Haltstellung angefordert** und der **Zielfestlegemelder (ZIF) am Fahrstrassenzielelement zurückgenommen**.
2. Danach wird geprüft, ob das Startsignal die **Haltstellung tatsächlich erreicht** hat. Solange dies nicht der Fall ist, darf die weitere Auflösung nicht durchgeführt werden.
3. Erst nachdem die Haltstellung erreicht wurde, wird der **Verschluss der zur Fahrstrasse gehörenden Weichen entfernt**, indem deren Verschlusszähler um eins dekrementiert wird.
4. Anschließend werden die durch die Fahrstrasse gesetzten **Beanspruchungen entfernt**.
5. Abschließend wird der Status der Fahrstrasse wieder auf **Status 0 – Fahrstrasse ausgeschaltet** gesetzt.

Damit gilt insbesondere die fachliche Sicherheitsregel: **Weichenverschluss und Beanspruchungen dürfen bei der Fahrstrassenauflösung erst entfernt werden, nachdem das Startsignal nachweislich die Haltstellung erreicht hat.**

## 4. Zustandsfolge

```text
Status 0
  Fahrstrasse ausgeschaltet
       |
       | Fahrstrasse gewählt
       v
  Zulassungsprüfung
       |
       | ZP positiv
       v
Status 1
  Beanspruchungen anfordern
       |
       | alle erforderlichen Beanspruchungen gesetzt
       v
Status 2
  Weichen in Sollstellung bringen
       |
       | alle Weichen in Sollstellung
       v
Status 3
  Weichen verschließen
       |
       | alle Weichen verschlossen
       v
Status 4
  Startsignal in Fahrtstellung bringen
```

Die Fahrstrassenauflösung wird unabhängig von dieser Einstellfolge durch das Auflöseelement angestoßen und führt nach Haltstellung des Startsignals, Aufhebung der Weichenverschlüsse und Entfernung der Beanspruchungen zurück zu Status 0.
