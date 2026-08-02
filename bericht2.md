# Code-Review: tools::Field (toolsfield.h)

## 1. Kurzbeurteilung

Die Klasse implementiert einen typsicheren, unveränderlichen "Variant"-Wertecontainer mit zehn unterstützten Datentypen, expliziten Konstruktoren pro Typ und einer vollständigen Konvertierungsmatrix über die `get*()`-Methoden. Die Grundidee (immutable Value-Type, explicit-Konstruktoren, final, klare Trennung von Speicherung und SQL-Logik) ist solide und die Absicht, Overflow/Underflow bei numerischen Konvertierungen abzufangen, ist lobenswert und in dieser Konsequenz selten zu sehen.

Allerdings hat die Klasse drei gravierende Probleme, die den Gesamteindruck deutlich trüben:

1. Eine klassische Overload-Resolution-Falle macht `Field(bool)` zum heimlichen Gewinner bei jeder Konstruktion aus einem String-Literal (`Field f("abc")`), wodurch der eigentliche Wert stillschweigend verworfen wird.
2. Mehrere der aufwendig implementierten Grenzwertprüfungen bei Float/Double-Konvertierungen sind selbst undefiniertes Verhalten oder schützen nicht zuverlässig – die "Sicherheitsnetze" sind an mehreren Stellen löchriger als der Code, den sie absichern sollen.
3. Die Klasse speichert alle zehn möglichen Werttypen gleichzeitig als Member, obwohl sie sich selbst im Kommentar als "eigener Variant" bezeichnet – ein klassischer Fall für `std::variant`, der hier ungenutzt bleibt.

Daneben gibt es eine Reihe kleinerer semantischer Inkonsistenzen (uneinheitliche String-Parsing-Strategien, uneinheitliche DateTime-Semantik zwischen `getBool()` und den übrigen numerischen Gettern, fehlende Erweiterbarkeitssicherung durch `if`-Ketten statt `switch`).

**Positiv:** Die Header/Inline-Trennung ist vollständig konsistent, const-Korrektheit ist durchgängig gegeben, und das Grundmuster (immutable, final, keine Vererbung) reduziert die Angriffsfläche für viele klassische C++-Fehlerklassen von vornherein.

---

## 2. Semantisches Modell

* **Verantwortung:** Hält genau einen Wert eines von zehn Typen, geschützt durch `explicit`-Konstruktoren; erlaubt typsicheren Zugriff sowie verlustbewusste Konvertierung in andere Typen.
* **Erwartete Zustände:** "Null" (via Default-Ctor, `m_isNull == true`) und "belegt mit Typ X" (via Wert-Ctor, `m_type == X`). Dazwischen gibt es keine weiteren Zustände – das Objekt ist nach Konstruktion eingefroren.
* **Wichtige Invarianten (vermutet, nicht durchgängig erzwungen):**
  * Wenn `m_isNull == true`, sollte kein Wertzugriff über `get*()` erfolgen (wird durch `throwIfNull()` erzwungen – konsistent umgesetzt).
  * `m_type` sollte den tatsächlich befüllten Member widerspiegeln – gilt für belegte Felder, nicht für Null-Felder (siehe F07).
  * Jeder `get*()` sollte entweder den nativen Wert oder eine "sinnvolle" Konvertierung liefern – wird für `getDateTime()` nicht eingehalten (siehe F13).
* **Besitzverhältnisse:** Reiner Wertetyp, kein Fremdbesitz, keine Zeiger/Handles. `std::string` und `DateTime` werden per Wert gehalten.
* **Vor-/Nachbedingungen:** Vorbedingung für alle `get*()` (außer `getType()`, `isNull()`): `isNull() == false`, sonst `std::runtime_error`. Nachbedingung: Objektzustand bleibt in jedem Fall unverändert (alle Getter sind rein lesend) – das ist sauber eingehalten, auch im Fehlerfall (starke Ausnahmegarantie für Lesezugriffe ist trivial gegeben, da nichts verändert wird).

---

## 3. Befunde

| ID | Schweregrad | Datei/Stelle | Kategorie | Befund | Auswirkung | Empfehlung |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **F01** | Kritisch | Konstruktoren `Field(bool)` / `Field(std::string const&)` | Overload-Resolution / UB-artiger Logikfehler | `Field(const char*)` existiert nicht. Bei `Field f("text")` konkurrieren `Field(std::string const&)` (benutzerdefinierte Konversion `const char*` → `std::string`) und `Field(bool)` (Standardkonversion Zeiger → `bool`). Standardkonversionen werden in der Overload-Resolution immer bevorzugt, also gewinnt `Field(bool)`. | Jede Konstruktion aus einem String-Literal oder `const char*` erzeugt still ein Bool-Feld mit Wert `true`; der eigentliche String geht komplett verloren. Kein Compilerfehler, kein Absturz – nur falsches Verhalten. | Explizite Überladung `explicit Field(char const * value)` ergänzen, die an `Field(std::string const&)` delegiert (Codebeispiel siehe Abschnitt 8). |
| **F02** | Kritisch | `getInt()`, `getUInt()`, `getInt64()`, `getUInt64()` – Zweige Float/Double/LongDouble | Undefiniertes Verhalten | Die Grenzwertprüfung castet `numeric_limits<int>::max()` (bzw. der jeweilige Zieltyp) nach `float`/`double`/`long double`. Diese Fließkommatypen können den Grenzwert oft nicht exakt darstellen und runden aufwärts (z. B. `static_cast<float>(INT_MAX) == 2147483648.0f`, also `INT_MAX + 1`). Werte knapp oberhalb der eigentlichen Grenze bestehen die Prüfung trotzdem. | Der anschließende `static_cast<int>(std::round(...))` wird auf einen Wert angewendet, der außerhalb des Zielbereichs liegt → undefiniertes Verhalten laut `[conv.fpint]`. Auf dem Zielsystem (Windows, `long double == double`) betrifft dies zusätzlich den LongDouble-Zweig von `getInt64()`/`getUInt64()`. | Vergleich mit einem Typ durchführen, der die Grenze exakt darstellen kann (bei 32-Bit-Zielen reicht `double`), bzw. für 64-Bit-Ziele die exakt darstellbare Zweierpotenz als exklusive Grenze verwenden (`< 9223372036854775808.0` statt `> static_cast<double>(INT64_MAX)`). Siehe Patch in Abschnitt 8. |
| **F03** | Kritisch | `getFloat()`, `getDouble()` – Zweige Int/UInt | Undefiniertes Verhalten | Es wird `static_cast<int>(std::numeric_limits<float>::lowest())` bzw. `static_cast<unsigned int>((std::numeric_limits<float>::max)())` (bzw. analog für `double`) berechnet. Diese Fließkommawerte ($\pm3.4 \times 10^{38}$) liegen weit außerhalb des Wertebereichs von `int`/`unsigned int`. | Das Casten eines Fließkommawerts, der außerhalb des Zielbereichs liegt, in einen Integer-Typ ist laut Standard undefiniertes Verhalten – und zwar bei jedem einzelnen Aufruf dieser Zweige, unabhängig vom eigentlichen `m_intValue`/`m_uintValue`. Nicht nur ein Grenzfall, sondern ein struktureller Fehler. | Prüfung entfernen bzw. umkehren: `int`/`unsigned int` liegen wertebereichsmäßig immer innerhalb dessen, was `float`/`double` darstellen können (nur Präzisionsverlust, kein Bereichsproblem) – die Prüfung ist unnötig und gleichzeitig gefährlich. Direkt `static_cast<float>(m_intValue)` verwenden. |
| **F04** | Hoch | Alle expliziten Wert-Konstruktoren | Overload-Resolution / Compilerfehler | Es fehlen Überladungen für `long`, `unsigned long`, `short`, `unsigned short`, `char`, `wchar_t` usw. Für diese Typen sind mehrere Konversionen (zu `int`, `int64_t`, `float`, `double` …) gleichrangig ("Conversion"-Rang), was zu einer mehrdeutigen Overload-Resolution führt. | Sobald ein Aufrufer einen `long`/`unsigned long`-Wert übergibt (unter Windows z. B. viele WinAPI-Typen wie `DWORD`), entsteht ein Compilerfehler ("ambiguous call"). Auf Linux würde derselbe Aufruf u. U. anders/kompilierbar sein → plattformabhängiges Verhalten der Schnittstelle. | Fehlende Überladungen ergänzen (mindestens `long`, `unsigned long`) oder bewusst per `= delete` sperren, um eine klare Fehlermeldung statt Mehrdeutigkeit zu erzeugen. |
| **F05** | Hoch | `getBool()` vs. `getInt()`/`getUInt()`/.../`getFloat()` – Zweig DateTime | Semantischer Widerspruch | Alle numerischen Getter interpretieren einen `DateTime`-Wert als "Sekunden seit Mitternacht" (Datum wird verworfen). `getBool()` hingegen prüft stattdessen `m_dateTimeValue.isValid()` – ein völlig anderes Kriterium (Gültigkeit statt Zahlenwert-ungleich-Null). | Zwei undokumentierte, inkonsistente Konventionen für denselben Quelltyp innerhalb derselben Klasse; für Aufrufer nicht vorhersehbar, welche Semantik bei welchem Getter gilt. | Konvention vereinheitlichen und explizit dokumentieren, z. B. `getBool()` auf `secondsOfDay() != 0` umstellen (oder umgekehrt: numerische Getter werfen für `DateTime`, analog zu `getDateTime()`s Verhalten für andere Typen). |
| **F06** | Mittel | Datei-Kopfkommentar vs. `m_isNull`/`isNull()` | Widersprüchliche Dokumentation | Der Kommentar sagt, auf `isNull` werde verzichtet, da dies über `std::optional` lösbar sei. Tatsächlich implementiert die Klasse `isNull()`, `m_isNull` und einen eigenen "Null erzeugenden" Default-Konstruktor. | Irreführende Dokumentation für zukünftige Bearbeiter; unklar, ob der `std::optional`-Ansatz ursprünglich geplant, dann aber verworfen wurde, ohne den Kommentar zu aktualisieren. | Kommentar an die tatsächliche Implementierung anpassen. |
| **F07** | Mittel | `Field::Field()` (Default-Ctor), `m_type`-Default | Fehlende/verletzte Invariante | Ein "Null"-Feld hat wegen des Default-Member-Initializers immer `getType() == DataType::String`, unabhängig vom eigentlich gewünschten Typ. Es gibt keine Möglichkeit, ein typisiertes NULL abzubilden (z. B. NULL-Wert einer Int-Spalte). | Code, der auf Basis von `getType()` verzweigt, kann ein NULL-Feld fälschlich als "leerer String" statt als "kein Wert vorhanden" behandeln. | `DataType::Null` (oder vergleichbar) ergänzen und im Default-Ctor setzen, oder `Field(DataType)`-Konstruktor für typisierte NULLs anbieten. |
| **F08** | Mittel | Alle `get*()`-Methoden | Erweiterbarkeit / fehlende Exhaustiveness-Prüfung | Statt `switch (m_type)` wird durchgängig eine Kette von `if (m_type == …)` verwendet. Der Compiler kann bei einer `if`-Kette nicht (wie bei `switch` mit `-Wswitch`) warnen, wenn ein neuer `DataType`-Wert nicht behandelt wird. | Wird `DataType` künftig erweitert (z. B. um Blob), fällt jede der zehn Methoden ohne Warnung auf den letzten `return {}`/`return 0;`-Fall zurück – eine stille Fehlfunktion statt eines Compilerfehlers. | Auf `switch (m_type) { case DataType::X: … }` ohne `default:`-Zweig umstellen, damit der Compiler fehlende Fälle meldet. |
| **F09** | Mittel | `getInt()`/`getUInt()`/`getFloat()`/`getDouble()`/`getBool()` vs. `getInt64()`/`getUInt64()`/`getLongDouble()` | Inkonsistenz | Zwei unterschiedliche String-Parsing-Mechanismen innerhalb derselben Klasse für strukturell dieselbe Aufgabe (String → Zahl). Erstere nutzen `tools::string_to_*`, letztere nutzen `std::stoll`/`std::stoull`/`std::stold` direkt. | Ungültige String-Werte können je nach aufgerufenem Getter zu unterschiedlichem Verhalten führen (z. B. stillschweigend 0 vs. Exception) – schwer vorhersehbares Fehlerverhalten für Aufrufer. | Auf einen einheitlichen Parsing-Mechanismus vereinheitlichen (entweder überall `tools::string_to_*` mit definiertem Fehlerverhalten, oder überall `std::sto*`). |
| **F10** | Mittel | Private Member-Liste (`m_stringValue`, `m_intValue`, … `m_dateTimeValue`) | Design / Speicher-Overhead | Die Klasse hält für jeden der zehn Typen ein eigenes Member gleichzeitig, statt nur den aktiv genutzten Wert zu speichern (kein `std::variant`/`union`). Jede `Field`-Instanz trägt dadurch die Summe aller Typgrößen (inkl. `std::string`, `DateTime`, `long double`), unabhängig vom tatsächlich gespeicherten Typ. | Unnötig hoher Speicherverbrauch und Konstruktionskosten pro Instanz; widerspricht dem eigenen Kopfkommentar ("Field ist ein eigener Variant"). Bei Verwendung in großen Zeilen-/Ergebnismengen kann sich das spürbar auswirken. | Siehe Umgestaltungsvorschlag in Abschnitt 8 (`std::variant`-basierte Umsetzung). |
| **F11** | Mittel | `getUInt()`, `getInt64()`, `getUInt64()`, `getFloat()`, `getDouble()` – negative-Wert-Prüfungen | Semantisch unpassende Exception | `std::underflow_error` wird verwendet, wenn ein negativer signed-Wert nicht in einen unsigned-Typ passt. Laut Standardbibliothek ist `underflow_error` für Fließkomma-Unterlauf gedacht, nicht für Vorzeichenkonflikte bei Integer-Konvertierung. | Aufrufer, die anhand des Exception-Typs unterscheiden, erhalten eine irreführende Fehlerklassifikation. | `std::out_of_range` oder eine eigene, aussagekräftige Exception-Klasse verwenden. |
| **F12** | Mittel | `Field(Field const&) = delete;`, `operator=(Field const&) = delete;` | Design / Nutzbarkeit | `Field` ist ein reiner Move-Only-Typ. Für einen kleinen, unveränderlichen Wertetyp, der typischerweise in Zeilen/Ergebnismengen gehalten und häufig kopiert wird, ist das eine ungewöhnliche Einschränkung. | Verwendungscode wie `std::vector<Field> row; row.push_back(f);` mit einem benannten (lvalue) `f` kompiliert nicht mehr; jede Kopie muss explizit über eine neue Konstruktion oder `std::move` erfolgen. | Falls Kopierbarkeit gewünscht ist: Copy-Ctor/-Assignment implementieren oder `= default` (alle Member sind kopierbar). Falls Move-Only beabsichtigt ist: kurz dokumentieren, warum. |
| **F13** | Mittel | `getDateTime()` | Unvollständige/inkonsistente Konvertierung | Für alle Nicht-DateTime-Typen wird ausnahmslos geworfen – auch für String, obwohl alle anderen Getter konsequent String → Zahl/Bool konvertieren. Das widerspricht der Vorgabe "Konvertierung, falls sinnvoll". | Ein Feld, das z. B. einen ISO-8601-String enthält, kann nicht als `DateTime` gelesen werden, obwohl das fachlich naheliegend wäre. | Klären, ob String → DateTime-Konvertierung gewünscht ist; falls ja, über eine vorhandene Parsing-Funktion in `DateTime`/`tools` ergänzen. |
| **F14** | Niedrig | `Field::get()` | Namensgebung | `get()` liefert identisch zu `getString()` immer einen String. Der generische Name suggeriert einen typunabhängigen bzw. nativen Zugriff. | Verwirrende öffentliche Schnittstelle; Aufrufer könnten einen anderen Rückgabetyp erwarten. | Methode umbenennen (z. B. entfernen, da redundant zu `getString()`) oder Zweck im Namen klarstellen. |
| **F15** | Niedrig | `getDateTime()` | DRY-Verstoß | Zehn nahezu identische `if`-Blöcke, die sich nur in der Fehlermeldung unterscheiden. | Hoher Wartungsaufwand bei Änderungen; Copy-Paste-Risiko. | Auf `switch` mit gemeinsamer, typnamen-parametrisierter Fehlermeldung reduzieren (siehe Abschnitt 8). |
| **F16** | Hinweis | `getString()` – Zweige Float/Double/LongDouble | Formatierungsqualität | `std::to_string()` liefert für Fließkommatypen eine feste Anzahl von sechs Nachkommastellen (z. B. `"3.140000"`). | Evtl. nicht das gewünschte/erwartete String-Format für Anzeige- oder Exportzwecke. | Bei Bedarf `std::format` (C++20) mit definierter Präzision verwenden. |
| **F17** | Hinweis | `throwIfNull()` | Diagnosequalität | Die Fehlermeldung (`"Field is null"`) enthält keinen Kontext, welcher Getter aufgerufen wurde. | Erschwerte Fehlersuche bei mehreren `get*()`-Aufrufen in einem größeren Ausdruck. | Aufrufenden Methodennamen (z. B. via `std::source_location` in C++20) in die Meldung aufnehmen. |
| **F18** | Hinweis | Alle `get*()` | Optionale Verbesserung | Kein `[[nodiscard]]` an den Gettern einer reinen Wertklasse. | Versehentliches Verwerfen eines Rückgabewerts wird nicht vom Compiler erkannt. | `[[nodiscard]]` ergänzen. |

## 4. Widersprüche zwischen Header und CPP

Die Datei ist header-only; Deklaration (in der Klasse) und inline-Definition (außerhalb der Klasse) liegen in derselben Datei. Ein Abgleich ergibt:

* Keine funktionalen Widersprüche. Alle Rückgabetypen, `const`-Qualifizierer und Parametertypen stimmen zwischen Deklaration und Definition überein.
* Rein kosmetische Abweichung: Bei den Wert-Konstruktoren (`int`, `unsigned int`, `int64_t`, `uint64_t`, `float`, `double`, `long double`, `bool`) ist der Parameter in der Klassendeklaration ohne `const` deklariert (`Field(int value)`), in der Definition dagegen mit Top-Level-`const` (`Field::Field(int const value)`). Das ist kein Fehler – Top-Level-`const` bei Wertparametern ist nicht Teil der Funktionssignatur und beeinflusst weder ODR noch Overload-Resolution –, aber stilistisch uneinheitlich innerhalb derselben Datei.
* Keine fehlenden Definitionen, keine Definitionen ohne passende Deklaration, keine abweichenden `noexcept`/`override`/Referenzqualifizierer gefunden.

## 5. Copy-, Move- und Lebensdaueranalyse

* **Copy-Konstruktor:** `= delete`. Bewusst gesperrt.
* **Move-Konstruktor:** `= default`. Da alle Member (`std::string`, Primitiven, `DateTime`) grundsätzlich bewegbar sind, ist der defaultierte Move funktional korrekt – sofern `DateTime` selbst einen Move-Konstruktor bereitstellt. Fehlt dieser, degradiert der Member-weise Move für `m_dateTimeValue` stillschweigend zu einer Kopie – kein Fehler, aber ein Performance-Verlust.
* **Copy-Zuweisungsoperator:** `= delete`.
* **Move-Zuweisungsoperator:** `= default`. Gleiche Anmerkung wie beim Move-Konstruktor.
* **Destruktor:** `= default`. Korrekt, da keine Ressourcen außerhalb der RAII-fähigen Member gehalten werden.
* **Rule-of-Five-Bewertung:** Konsistent angewendet (alle fünf Spezialfunktionen sind explizit deklariert), keine Lücke. Die Entscheidung, Move-Only statt Rule of Zero (vollständig defaultiert inkl. Copy) umzusetzen, ist die eigentlich bemerkenswerte Design-Entscheidung (siehe F12).
* **Lebensdauer zurückgegebener Werte:** Alle Getter geben Werte per Wert zurück (keine Referenzen/Zeiger auf interne Member), daher keine Dangling-Reference-Risiken.
* **Ownership externer Ressourcen:** Keine vorhanden; reiner Werte-Container.

## 6. Exception-Sicherheit

* **Mögliche Exception-Quellen:** `std::bad_alloc` (String-/DateTime-Kopien), `std::runtime_error` (`throwIfNull`), `std::overflow_error`/`std::underflow_error` (Bereichsprüfungen, teils fälschlich klassifiziert, siehe F11), `std::invalid_argument` (`getDateTime()`), zusätzlich implizit `std::invalid_argument`/`std::out_of_range` aus `std::stoll`/`std::stoull`/`std::stold` sowie unbekanntes Verhalten aus `tools::string_to_*`.
* **Verhalten bei teilweise ausgeführten Operationen:** Da sämtliche Getter rein lesend sind und im Fehlerfall vor jeder Zustandsänderung werfen (es gibt ohnehin keine Zustandsänderung), bleibt das Objekt in jedem Fall unverändert.
* **Garantie-Stufe:** Für alle Getter gilt de facto die No-throw-auf-Objektzustand-Garantie (stärker als Strong Guarantee, da überhaupt keine Zustandsänderung möglich ist). Für die Konstruktoren gilt die Basic Guarantee (Standardverhalten bei Fehlschlag der String-/DateTime-Konstruktion).
* **noexcept-Korrektheit:** Kein einziger Konstruktor oder Getter ist explizit `noexcept` markiert – korrekt, da praktisch alle Pfade potenziell werfen können. `getType()` und `isNull()` könnten dagegen gefahrlos `noexcept` markiert werden. Die implizit generierten Move-Operationen sind `noexcept`, sofern `DateTime`s Move-Konstruktor es ebenfalls ist.
* **Objektzustand nach Exception:** Unverändert.

## 7. Optimierungsmöglichkeiten

### Notwendige Korrekturen (Bugfixes):
* F01 (`const char*`-Konstruktor)
* F02, F03 (UB in Grenzwertprüfungen)
* F04 (fehlende Ganzzahl-Überladungen)

### Sinnvolle Designverbesserungen:
* F08 (`switch` statt `if`-Ketten für Exhaustiveness)
* F09 (einheitliches String-Parsing)
* F10 (`std::variant`-Umbau, größte Verbesserung)
* F05, F07, F13 (Semantik-Klärungen)

### Performanceoptimierungen:
* F10 ist gleichzeitig die relevanteste Performance-Maßnahme (Objektgröße von "Summe aller Typen" auf "Größe des größten Typs" reduzieren).
* Keine weiteren nennenswerten Performance-Probleme gefunden; die Klasse führt keine unnötigen Kopien in den Gettern durch.

### Rein stilistische Änderungen:
* F14 (Namensgebung `get()`), F16 (String-Formatierung), F17 (Fehlermeldungskontext), F18 (`[[nodiscard]]`), sowie die kosmetische `const`-Uneinheitlichkeit.

## 8. Verbesserungsvorschlag

### Fix F01 – fehlender `const char*`-Konstruktor

~~~cpp
// Deklaration (public-Bereich, vor oder nach Field(std::string const&)):
explicit Field(char const * value);

// Definition:
inline Field::Field(char const * value)
    : Field(value != nullptr ? std::string(value) : std::string())
{
}
~~~

### Fix F03 – UB in getFloat()/getDouble() (Int/UInt-Zweige)

~~~cpp
// Vorher (UB):
if (m_type == DataType::Int) {
    if (m_intValue < static_cast<int>(std::numeric_limits<float>::lowest()) ||
        m_intValue > static_cast<int>((std::numeric_limits<float>::max)())) {
        throw std::overflow_error("int value exceeds float limits");
    }
    return static_cast<float>(m_intValue);
}

// Nachher:
if (m_type == DataType::Int) {
    // int liegt immer im darstellbaren Wertebereich von float (nur Präzisionsverlust möglich, kein Bereichsfehler).
    return static_cast<float>(m_intValue);
}
~~~

### Fix F02 – ungenaue Grenzwertprüfung in getInt64() (Double/LongDouble-Zweig)
~~~cpp
// Vorher (Rundung von INT64_MAX nach double kann UB im finalen Cast erzeugen):
if (m_type == DataType::Double) {
    if (m_doubleValue > static_cast<double>((std::numeric_limits<int64_t>::max)()) ||
        m_doubleValue < static_cast<double>((std::numeric_limits<int64_t>::min)())) {
        throw std::overflow_error("double value exceeds int64 limits");
    }
    return static_cast<int64_t>(std::round(m_doubleValue));
}

// Nachher: exakte Zweierpotenzen als Grenzen verwenden (in double exakt darstellbar)
if (m_type == DataType::Double) {
    constexpr double lower = -9223372036854775808.0; // -2^63, exakt darstellbar
    constexpr double upper =  9223372036854775808.0; //  2^63, exakt darstellbar
    double const rounded = std::round(m_doubleValue);
    if (rounded < lower || rounded >= upper) {
        throw std::overflow_error("double value exceeds int64 limits");
    }
    return static_cast<int64_t>(rounded);
}
~~~

### Fix F08/F15 – switch statt if-Kette (Beispiel getDateTime())

~~~cpp
inline auto Field::getDateTime() const -> DateTime
{
    throwIfNull();
    switch (m_type) {
        case DataType::DateTime:
            return m_dateTimeValue;
        case DataType::String:
        case DataType::Int:
        case DataType::UInt:
        case DataType::Int64:
        case DataType::UInt64:
        case DataType::Float:
        case DataType::Double:
        case DataType::LongDouble:
        case DataType::Bool:
            throw std::invalid_argument("Cannot convert this Field's type to DateTime");
    }
    throw std::logic_error("unreachable: unhandled DataType in Field::getDateTime");
}
~~~

Größere Umgestaltung – std::variant statt Multi-Member-Layout (F10)
Konkretes Problem: Jede Field-Instanz reserviert Speicher für alle zehn Typen gleichzeitig, obwohl immer nur einer aktiv ist.

Nutzen: Objektgröße sinkt auf die Größe des größten Members zzgl. Diskriminante; m_type und die aktive Alternative können nie auseinanderlaufen; std::visit ersetzt die if/switch-Ketten und ist von Natur aus exhaustiv-prüfbar.

Mögliche Nachteile: std::variant<...>::index() liefert einen size_t, keinen DataType – eine Mapping-Schicht wird nötig, wenn die öffentliche DataType-Enum erhalten bleiben soll.

Migrationsaufwand: Mittel – die öffentliche Schnittstelle (getType(), get*()) kann unverändert bleiben, nur die private Speicherung und die Getter-Implementierungen ändern sich.

## 9. Offene Fragen
Welches Fehlerverhalten zeigen tools::string_to_int/float/double/bool bei ungültiger Eingabe (Exception? Rückgabe 0)? Das ist entscheidend für die Bewertung von F09.

Bietet DateTime einen Move-Konstruktor/-Zuweisungsoperator an? Das beeinflusst, ob Fields defaultierter Move tatsächlich verschiebt oder stillschweigend kopiert.

Ist der Move-Only-Charakter von Field (kein Copy) bewusst gewählt, oder wurde er nur "aus Sicherheit" gesetzt (F12)?

Soll ein NULL-Wert einen ursprünglich beabsichtigten Typ tragen können (z. B. NULL einer Int-Spalte), oder ist "NULL ohne Typ" fachlich ausreichend (F07)?

Ist die Interpretation von DateTime als "Sekunden seit Mitternacht" bei den numerischen Gettern tatsächlich das gewünschte Verhalten, oder wird an anderer Stelle ein vollständiger Zeitstempel (z. B. Unix-Zeit) erwartet (F05)?

Wird Field jemals über Modul-/DLL-Grenzen hinweg verwendet? Das wäre relevant für ABI-Stabilität, insbesondere bei künftigen Layoutänderungen (z. B. std::variant-Umbau).

## 10. Abschließende Prioritätenliste
Zwingend zu korrigieren:
F01 – const char*-Konstruktor ergänzen (stiller Datenverlust)

F02, F03 – UB in den Grenzwertprüfungen der Float/Double/Int-Konvertierungen beseitigen

F04 – Overload-Mehrdeutigkeit für long/unsigned long/short/char auflösen

Anschlieβend verbessern:
F05 – DateTime-Semantik zwischen getBool() und den übrigen Gettern vereinheitlichen

F08 – if-Ketten durch switch ersetzen (Exhaustiveness-Schutz)

F09 – String-Parsing-Strategie vereinheitlichen

F10 – std::variant-Umbau (größter struktureller Gewinn, aber planbar)

F06, F07, F11, F12, F13 – Dokumentation/Semantik klären bzw. korrigieren

Optional:
F14, F15, F16, F17, F18 – Namensgebung, DRY, Formatierung, Diagnose, [[nodiscard]]
"""
