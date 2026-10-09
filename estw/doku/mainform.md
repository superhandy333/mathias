# Form MainFom

Das MainForm zeigt im linken Bereich eine vertikal angeordnete Reihe von Button.

Das MainForm zeigt im rechten Bereich ein Label mit der Aufschrift: "DataSource:" und darunter ein mehrzeiliges Label, welches die Informationen enthält: Aktuelle DataSource, falls DataSource=mariadb dann Host und Port, falls Local CVS dann Pfad. Wenn sich diese Daten aufgrund von Setup innerhalb der Instanz ändern, dann soll sich auch hier der Inhalt ändern.

Der Button `Setup` befindet sich unterhalb des mehrzeiligen Labels. Er ist etwas schmaler als die anderen Hauptbuttons am linken Rand.

Alle vom Hauptfenster geöffneten Datensatzlisten zeigen bei Doppelklick auf eine belegte Zeile die vollständigen Datensatzfelder im `DetailForm`. Für die Bedienung und die datenquellenabhängige Bearbeitbarkeit gilt [DETAILBEARBEITUNG.md](../DETAILBEARBEITUNG.md). Meldungslisten bleiben reine Listen ohne Detailbearbeitung.
