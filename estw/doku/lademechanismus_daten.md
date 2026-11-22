## Lademechnismus Daten

Beim Start der Anwendung bzw. Öffnen des MainForm sollen noch keine Daten geladen sein. Der Zustand, ob Daten geladen sind, wird im Flag `isDataLoaded` festgehalten. Wird eines der Button gedrückt, für die die Daten benötigt werden, werden diese entsprechend DataSource geladen und das Flag wird umgeschaltet, damit beim nächsten Anklicken eines Buttons die Daten nicht noch einmal geladen werden.

Wird zwischenzeitlich der `Setup` aufgerufen und die DataSource geändert, wird das Flag `isLoaded` zurückgesetzt.

Bei Local CSV werden Feldanzahl und UTF-8-Kodierung aller sechs Tabellen vor der
Übernahme der Zeilen geprüft. Das Fenster `Datenfehler` nennt bei falscher
Feldanzahl Datei, Zeile, erwartete und tatsächliche Feldanzahl; bei ungültigem
UTF-8 zusätzlich die Feldnummer und den Hinweis, die Datei als UTF-8 zu speichern.
Die Zeilen- und Feldnummern beginnen bei 1. Fehlerhafte Zeilen werden nicht geladen,
gültige Zeilen bleiben verfügbar. Die Quelldateien werden dabei nicht verändert.
`projekte.csv` erlaubt sowohl das alte Format mit 3 als auch das erweiterte mit
27 Feldern. `elemente.csv` erwartet weiterhin 19 Felder.

Das Skript [export.sql](../../sql/export.sql) exportiert die 27 Projektfelder in
der Reihenfolge `id,bezeichnung,beschreibung,x1,y1,zoom1,...,x8,y8,zoom8`.
Alle sechs CSV-Ausgaben verwenden ausdrücklich `CHARACTER SET utf8mb4`, auch wenn
die Quelltabellen Latin-1 verwenden. Vor dem Export müssen die 24 Ausschnittspalten
in `projekte` vorhanden sein; hierfür dient
[projekte_ausschnitte.sql](../sql/projekte_ausschnitte.sql).

Die Meldungsliste wird modal mit dem Hauptfenster als Besitzer angezeigt.
Nach ihrem Schließen wird die angeforderte Liste geöffnet. Fehler beim Laden
oder beim Aufbau einer der sechs Datensatzlisten werden über `showMessage`
angezeigt. Auch fehlerhaft kodierte Ausnahmebeschreibungen bleiben darstellbar.

