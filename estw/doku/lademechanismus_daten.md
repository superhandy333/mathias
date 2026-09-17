## Lademechnismus Daten

Beim Start der Anwendung bzw. Öffnen des MainForm sollen noch keine Daten geladen sein. Der Zustand, ob Daten geladen sind, wird im Flag `isDataLoaded` festgehalten. Wird eines der Button gedrückt, für die die Daten benötigt werden, werden diese entsprechend DataSource geladen und das Flag wird umgeschaltet, damit beim nächsten Anklicken eines Buttons die Daten nicht noch einmal geladen werden.

Wird zwischenzeitlich der `Setup` aufgerufen und die DataSource geändert, wird das Flag `isLoaded` zurückgesetzt.

