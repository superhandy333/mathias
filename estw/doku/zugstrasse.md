# Zugstrasse – Mechanik

Dieses Dokument beschreibt den ersten fachlichen Entwurf der Zustandsmechanik einer **Zugstrasse** in `estw3`. Die Beschreibung basiert zunächst auf der Mechanik der Rangierstrasse. Für die Zugstrasse ist zusätzlich ein eigener Zustand für den **Flankenschutz** erforderlich. Die genaue fachliche Ausgestaltung dieses Zustands ist noch festzulegen.

Für Signalarten, Halt- und Fahrtstellungen sowie die Ermittlung der Vorsignalgeschwindigkeit gelten die verbindlichen Vorgaben aus [signale.md](signale.md).

## Struktureller Aufbau und Definition in der Datenstruktur

Im Sinne des Datenmodells verknüpft eine **Fahrstrasse** (`Fahrstrasse`) mehrere **Elemente** (`Element`) eines Projekts über zugeordnete **Fahrstrassenelemente** (`FahrstrassenElement`). Jedes Fahrstrassenelement weist dem referenzierten Element eine spezifische funktionale Rolle (`FahrstrassenElementtyp`) innerhalb der Fahrstrasse zu.
  
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
  - Eine Zugstrasse darf **höchstens ein** Vorwegelement besitzen.
  - Sofern ein Vorwegelement vorhanden ist, muss das darüber referenzierte Element vom Elementtyp **Signal** (`ElementTypId = 3`) sein.
- **Unterwegselement (`TypId = 7`):**
  - Eine Zugstrasse kann 0 oder eine beliebige Anzahl von Unterwegselementen besitzen.
  - Sofern ein Unterwegselement vorhanden ist, müssen die darüber referenzierten Elemente vom Elementtyp **Signal** (`ElementTypId = 3`) sein.
- **Flankenschutztransportelement (`TypId = 8`):**
  - Eine Zugstrasse darf 0 oder eine beliebige Anzahl von Flankenschutztransportelementen besitzen.
  - Das über ein Flankenschutztransportelement referenzierte Element muss vom Elementtyp **Gleis** (`ElementTypId = 1`), **Weiche** (`ElementTypId = 2`), **Signal** (`ElementTypId = 3`), **Blindziel** (`ElementTypId = 4`) oder **Auflöseelement** (`ElementTypId = 5`) sein.

Diese strukturellen Festlegungen stellen sicher, dass jede Zugstrasse über eindeutige Start-, Ziel- und Auflösepunkte verfügt und die beteiligten Stellwerkselemente die dafür zulässigen Elementtypen besitzen.

## 1. Zulassungsprüfung

Die speziellen Zulassungsbedingungen für eine Zugstrasse werden in diesem Abschnitt dokumentiert. Die Zulassungsprüfung ist nur dann positiv, wenn **alle** für die Zugstrasse geltenden Bedingungen erfüllt sind.

Die zugstrassenspezifischen Zulassungsbedingungen sind derzeit noch nicht abschließend festgelegt und werden hier ergänzt, sobald sie fachlich definiert sind. Allgemeine Bedeutung und Zweck der Zulassungsprüfung sind in [estw.md](estw.md) beschrieben.

## 2. Zustandsmodell

Die Zugstrasse durchläuft beim Einstellen mehrere aufeinanderfolgende Zustände. Der Übergang in den jeweils nächsten Zustand erfolgt erst, wenn die dafür genannten Voraussetzungen erfüllt sind. Dadurch werden angeforderte Aktionen zunächst an den beteiligten ESTW-Elementen ausgelöst und ihr tatsächlicher Zustand anschließend geprüft.

### Status 0 – Fahrstrasse ausgeschaltet

Die Fahrstrasse ist ausgeschaltet. Dies ist ihre Grundstellung.

Wird die Fahrstrasse gewählt, während sie sich in Status 0 befindet, wird die **Zulassungsprüfung (ZP)** durchgeführt. Die dafür geltenden zugstrassenspezifischen Bedingungen werden im Abschnitt **„Zulassungsprüfung“** dieses Dokuments beschrieben.

Nur bei positiver Zulassungsprüfung darf mit dem Einstellen der Fahrstrasse fortgefahren werden.

### Status 1 – Beanspruchungen anfordern

**Voraussetzung:** Die Zulassungsprüfung ist positiv abgeschlossen.

**Aktionen:**

- An allen **Fahrwegelementen** wird versucht, die für die Fahrstrasse erforderliche Beanspruchung zu setzen.
- Am **Startsignal** wird die Beanspruchung auf **Strang A** gesetzt.
- Ist das Fahrstrassenziel ein **Zielsignal**, wird dort die Beanspruchung auf **Strang B** gesetzt.
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

### Zusätzlicher Status – Flankenschutz herstellen

Für eine **Zugstrasse** ist gegenüber der Rangierstrasse ein zusätzlicher Zustand zur Herstellung und Prüfung des **Flankenschutzes** erforderlich.

Die genaue Position dieses Zustands innerhalb der Zustandsfolge, seine Voraussetzungen, die auszuführenden Aktionen sowie die Bedingungen, unter denen der Flankenschutz als vollständig hergestellt gilt, sind **noch festzulegen**. Bis zu dieser Festlegung wird hierfür bewusst keine Statusnummer vergeben.

### Status 4 – Startsignal in Fahrtstellung bringen

**Voraussetzungen:**

- Die Voraussetzungen aus Status 3 sind weiterhin erfüllt.
- Zusätzlich sind alle zur Fahrstrasse gehörenden **Weichen** tatsächlich verschlossen.
- Für die Zugstrasse muss zusätzlich der erforderliche **Flankenschutz vollständig hergestellt** sein. Die Detailregeln hierfür werden noch ergänzt.

**Aktion:** Das **Startsignal** wird in die für die Fahrstrasse projektierte Sollstellung, also die **Fahrtstellung**, gebracht.

## 3. Auflösung der Zugstrasse

Die Auflösung der Fahrstrasse wird durch **Anklicken des zugehörigen Auflöseelements** gestartet.

Die Auflösung erfolgt zunächst entsprechend der Rangierstrasse in einer festen Reihenfolge:

1. Zunächst wird am **Startsignal die Haltstellung angefordert**.
2. Danach wird geprüft, ob das Startsignal die **Haltstellung tatsächlich erreicht** hat. Solange dies nicht der Fall ist, darf die weitere Auflösung nicht durchgeführt werden.
3. Erst nachdem die Haltstellung erreicht wurde, wird der **Verschluss der zur Fahrstrasse gehörenden Weichen entfernt**.
4. Anschließend werden die durch die Fahrstrasse gesetzten **Beanspruchungen entfernt**.
5. Abschließend wird der Status der Fahrstrasse wieder auf **Status 0 – Fahrstrasse ausgeschaltet** gesetzt.

Für die Zugstrasse muss zusätzlich noch festgelegt werden, **wann und unter welchen Bedingungen der Flankenschutz bei der Auflösung aufgehoben wird**. Bis zu dieser Festlegung wird hierzu keine konkrete Reihenfolge vorgegeben.

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
  Flankenschutz herstellen
  (Statusnummer und Detailmechanik noch festzulegen)
       |
       | Flankenschutz hergestellt
       v
Status 4
  Startsignal in Fahrtstellung bringen
```

Die genaue Einordnung und Nummerierung des zusätzlichen Flankenschutz-Zustands wird ergänzt, sobald die fachliche Mechanik festgelegt ist.

## 5. Noch zu ergänzende Punkte

Dieser Stand ist ein erster Entwurf auf Basis der Rangierstrassenmechanik. Insbesondere sind für den **Flankenschutz** noch festzulegen:

- Position und Statusnummer innerhalb der Zustandsfolge,
- Voraussetzungen für die Herstellung des Flankenschutzes,
- Aktionen an Flankenschutzelementen und Flankenschutztransportelementen,
- Prüfung, wann der Flankenschutz vollständig hergestellt ist,
- Verhalten bei nicht herstellbarem Flankenschutz,
- Aufhebung des Flankenschutzes bei der Fahrstrassenauflösung.

Weitere fachliche Bedingungen, Fehler- und Rückfallzustände sowie gegebenenfalls weitere Zustände werden ergänzt, sobald sie festgelegt sind.
