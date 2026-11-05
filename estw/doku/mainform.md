# Form MainFom

Das MainForm zeigt im linken Bereich eine vertikal angeordnete Reihe von Button.

Das MainForm zeigt im rechten Bereich ein Label mit der Aufschrift: "DataSource:" und darunter ein mehrzeiliges Label, welches die Informationen enthält: Aktuelle DataSource, falls DataSource=mariadb dann Host und Port, falls Local CVS dann Pfad. Wenn sich diese Daten aufgrund von Setup innerhalb der Instanz ändern, dann soll sich auch hier der Inhalt ändern.

Der Button `Setup` befindet sich unterhalb des mehrzeiligen Labels. Er ist etwas schmaler als die anderen Hauptbuttons am linken Rand.

Die Projektliste öffnet bei Doppelklick auf eine belegte Zeile direkt den Dialog `Projekt`. Die übrigen vom Hauptfenster geöffneten Datensatzlisten zeigen dabei die vollständigen Datensatzfelder im `DetailForm`. Für die Bedienung und die datenquellenabhängige Bearbeitbarkeit gilt [DETAILBEARBEITUNG.md](../DETAILBEARBEITUNG.md). Meldungslisten bleiben reine Listen ohne Detailbearbeitung.

Im Dialog `Projekt` sind Bezeichnung und Beschreibung ausschließlich bei MariaDB und mrdb bearbeitbar. Ungespeicherte Änderungen blenden Simulation, Edit und FS-Edit aus und zeigen stattdessen Speichern. Nach erfolgreicher gemeinsamer Speicherung beider Texte werden Vergleichsstand, Projektcache und Projektliste aktualisiert. Fehler erhalten Eingaben und Änderungszustand. Das Wiederherstellen beider Ausgangswerte stellt die Modus-Buttons ohne Speicherung wieder her. ID und Kennzahlen bleiben lesend.
