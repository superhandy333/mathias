# Analyse: Grafische Inhalte im `SegmentControl`

## Analyseergebnis

Am besten passt eine bewusst kleine Form von Variante C: Jedes Segment erhält einen Inhalt vom Typ Text oder Grafik-Callback. Intern ist das zugleich Variante B, weil die Auswahl pro Segment erfolgt.

Die bisherige Überladung mit `std::vector<std::string>` bleibt unverändert und wird intern lediglich auf das neue Inhaltsmodell abgebildet. Ein globaler `SegmentDisplayMode` ist dann nicht erforderlich; die Bezeichnung `SegmentContent` trennt Darstellung eindeutig vom bestehenden Auswahlzustand.

## 1. Derzeitige Implementierung

### Erzeugung und Verwaltung

`SegmentControl` ist kein Win32-Standard-Control und auch kein klassisches Owner-/Custom-Draw-Control. Win32Form registriert dafür eine eigene Fensterklasse `Win32FormSegmentedControl`:

- Registrierung in `win32form.cpp`, ab Zeile 277
- Sonderbehandlung gegenüber `createControl()` in `win32form.cpp`, ab Zeile 820
- Erzeugung durch `createSegmentControl()` in `win32form.cpp`, ab Zeile 1663
- eigene Window-Procedure in `win32form.cpp`, ab Zeile 2454

Das Fenster wird mit `WS_CHILD | WS_VISIBLE | WS_TABSTOP` erstellt. Die anfängliche Breite ist `Anzahl × itemWidth`, die Höhe ist fest 25 Pixel.

Danach wird ein `std::shared_ptr<ControlData>` erzeugt und in `Win32Form::controls` gespeichert. Die Texte werden kopiert; es bleiben keine Referenzen auf Aufruferdaten bestehen.

### Interne Daten

Die relevanten Felder stehen in `win32form.h`, ab Zeile 218:

```cpp
std::vector<std::string> SegmentedControlItems{};
std::optional<std::size_t> SegmentedControlSelectedIndex{};
std::optional<std::size_t> SegmentedControlHoverIndex{};
int SegmentedControlItemWidth{75};
tools::Event<std::size_t> OnSegmentedControlChanged{};
```

Damit sind Inhalt, Auswahl und Hover bereits getrennt gespeichert. Für eine Erweiterung sollte diese Trennung beibehalten werden.

### Zeichnung

Die vollständige Zeichnung erfolgt mit GDI in `paintSegmentedControl()` ab `win32form.cpp`, Zeile 2346:

1. Client-Rechteck auslesen.
2. Kompatiblen Memory-DC und Bitmap erzeugen.
3. Hintergrund vollständig in den Memory-DC zeichnen.
4. Abgerundete Clip-Region setzen.
5. Für jedes Segment Hintergrund, Text und Trennlinie zeichnen.
6. Rahmen und Fokusrechteck zeichnen.
7. Ergebnis mit `BitBlt` übertragen.
8. GDI-Objekte freigeben.

Der Text wird folgendermaßen ausgegeben:

```cpp
DrawText(
    memDC,
    data->SegmentedControlItems[i].c_str(),
    -1,
    &segmentRect,
    DT_CENTER | DT_VCENTER | DT_SINGLELINE |
        DT_END_ELLIPSIS | DT_NOPREFIX);
```

Durch Memory-DC, `WM_ERASEBKGND`-Unterdrückung und abschließendes `BitBlt` ist das Control bereits grundsätzlich flimmerarm.

### Zustände und Interaktion

Die bestehende Darstellung ist:

| Zustand | Darstellung |
|---|---|
| Normal | Control-/Form-Hintergrund und Vordergrundfarbe |
| Ausgewählt | festes Blau, weiße Schrift |
| Hover | aufgehellter neutraler Hintergrund |
| Disabled | neutraler Hintergrund, graue Schrift |
| Fokus | Fokusrechteck um das gesamte Control |

Ein ausgewähltes Segment verliert im Disabled-Zustand seine hervorgehobene Fläche; der Auswahlindex bleibt intern aber erhalten.

Die Verarbeitung erfolgt in `win32form.cpp`, ab Zeile 2473:

- Mausklick: Fokus setzen, Hit-Test, Auswahl ändern
- Hover: `TrackMouseEvent`, Hover-Index aktualisieren
- Mouse Leave: Hover löschen
- Tastatur: Links/Rechts, Oben/Unten, Pos1 und Ende
- Fokus und Enabled-Änderungen: Neuzeichnen

Die Auswahl wird in `win32form.cpp`, ab Zeile 2335, zentral geändert. Programmatische Änderungen lösen kein `OnChange` aus; Benutzeraktionen nur bei tatsächlich geändertem Index.

Die Tastatur- und Ereignislogik hängt ausschließlich von Segmentanzahl und Index ab. Sie muss für Grafikinhalte nicht verändert werden.

### Größenänderungen

`paintSegmentedControl()` liest bei jedem Paint das aktuelle Client-Rechteck aus. Die Segmentbreiten werden trotzdem weiterhin aus `SegmentedControlItemWidth` berechnet. Eine beliebige Vergrößerung über `setControlSize()` verteilt den Platz daher nicht automatisch neu.

`setSegmentControlButtonWidth()` in `win32form.cpp`, ab Zeile 1731, ändert die gespeicherte Breite, passt die Gesamtbreite des Controls an und invalidiert es.

## 2. Ressourcen- und Lebensdauermechanismen

Win32Form besitzt zwei RAII-Wrapper:

- `Win32Font` besitzt ein `HFONT` und löscht es im Destruktor.
- `Win32Color` besitzt ein `HBRUSH`, ersetzt und löscht es kontrolliert; siehe `win32form.cpp`, ab Zeile 213.

Für Pens, Regions, DCs und Bitmaps existieren dagegen keine Wrapper. Diese Objekte werden in der Paint-Methode manuell angelegt und freigegeben:

- Backbuffer-DC und Bitmap
- Clip-Region
- Hover-Brush
- je eine Pen pro Trennlinie
- Rahmen-Pen
- temporäre `Win32Color`-Objekte für Akzent und Separatoren

GDI+ wird im Projekt nicht verwendet. Es gibt insbesondere keine:

- `GdiplusStartup`-/`GdiplusShutdown`-Verwaltung,
- GDI+-Image- oder Graphics-Wrapper,
- WIC- oder Direct2D-Infrastruktur,
- Bild- oder SVG-Caches.

Ein Grafik-Callback sollte deshalb zunächst nur einen geliehenen `HDC` erhalten. Dadurch wird GDI+ nicht Teil der öffentlichen Win32Form-Abhängigkeit.

## 3. Betroffene öffentliche Methoden

Die bestehenden Deklarationen stehen in `win32form.h`, ab Zeile 434.

### `createSegmentControl()`

Diese Methode ist direkt betroffen, weil sie Inhalt und Segmentanzahl erzeugt. Die bestehende Signatur muss unverändert bleiben:

```cpp
void createSegmentControl(
    std::string const & name,
    std::vector<std::string> const & items,
    std::optional<std::size_t> selectedIndex = std::nullopt,
    int itemWidth = 75);
```

Reale Aufrufer existieren sowohl im Test ab `main.cpp`, Zeile 376, als auch in `estw/estw3/toolform.cpp`, ab Zeile 56.

### `setSegmentControlState()` und `getSegmentControlState()`

Diese Methoden verwalten ausschließlich den Auswahlindex und sollten unverändert bleiben. Text/Grafik ist kein neuer „State“.

Intern muss lediglich die Größenprüfung künftig die Anzahl der allgemeinen Segmentinhalte statt ausschließlich `SegmentedControlItems.size()` verwenden.

### `setSegmentControlOnChange()`

Keine Signaturänderung erforderlich. Das Ereignis meldet weiterhin den gewählten Index.

### `setSegmentControlButtonWidth()`

Keine Signaturänderung erforderlich. Die Methode beeinflusst Text- und Grafiksegmente gleichermaßen.

## 4. Vergleich der Darstellungsmodelle

| Variante | Vorteile | Nachteile | Bewertung |
|---|---|---|---|
| A: Modus pro Control | kleinste interne Änderung; einfache Text- oder Callback-Überladung | keine gemischten Segmente; zusätzlicher Moduszustand; weniger flexibel | nur sinnvoll, wenn garantiert immer alle Segmente gleichartig sind |
| B: Modus pro Segment | erfüllt „wahlweise“ exakt; gemischte Controls möglich | parallele Text-/Callback-Vektoren können inkonsistent werden | technisch passend, wenn sauber gekapselt |
| C: gemeinsames Inhaltsmodell | ein Vektor ist einzige Quelle für Inhalt und Anzahl; gut erweiterbar; keine Modusverwechslung | einige neue öffentliche Typen; etwas mehr Validierung | beste Lösung, sofern das Modell bewusst klein bleibt |

Empfohlen wird C mit genau zwei Inhaltsalternativen: Text und Grafik-Callback. Das ist intern gleichzeitig Variante B, ohne parallele Datenstrukturen.

Eine Kombination aus Symbol und Text muss jetzt nicht implementiert werden. Sie könnte später als dritte klar benannte Alternative ergänzt werden.

## 5. Grafikmöglichkeiten

### Direkte GDI-Zeichnung

Das ist die architektonisch einfachste Lösung:

- Der vorhandene Memory-DC kann direkt weiterverwendet werden.
- Keine neuen Bibliotheken oder Initialisierungsschritte.
- Sehr gut für kleine monochrome Symbole aus Linien, Polygonen, Rechtecken und Ellipsen.
- Mit `DC_PEN` und `DC_BRUSH` können zustandsabhängige Farben ohne neue GDI-Objekte pro Paint gesetzt werden.
- Die Zeichenfunktion kann die von Win32Form ermittelte Vordergrundfarbe übernehmen.

Nachteile:

- GDI ist nicht automatisch antialiased.
- Komplexe Kurven und importierte Illustrationen sind umständlich.
- Der Callback muss sauber vom übrigen DC-Zustand isoliert werden.

Win32Form sollte vor dem Callback `SaveDC`, eine Clip-Region und danach `RestoreDC` verwenden. Der Callback darf den `HDC` nicht speichern oder außerhalb des Aufrufs verwenden.

### GDI+-Zeichnung

Ein Callback mit `HDC` schließt GDI+ nicht aus: Der Aufrufer kann darin temporär ein `Gdiplus::Graphics` für den bereitgestellten DC erzeugen.

Vorteile:

- Antialiasing und Pfade
- komfortablere Vektorgrafik
- PNG mit Alpha-Unterstützung

Nachteile:

- GDI+ muss vor der ersten Nutzung gestartet und danach beendet werden.
- `Gdiplus.lib` muss gelinkt werden.
- Eine verlässliche globale Lebensdauerverwaltung fehlt momentan.
- GDI+ unterstützt die üblichen Bitmap- und Metafile-Formate, aber kein SVG. Das folgt auch aus der offiziellen Liste der [von GDI+ unterstützten Grafikformate](https://learn.microsoft.com/en-us/windows/win32/gdiplus/-gdiplus-types-of-bitmaps-about).
- GDI+-Objekte müssen vor `GdiplusShutdown` zerstört sein; die Initialisierung ist damit eine anwendungsweite Verantwortlichkeit, siehe [GDI+-Initialisierung](https://learn.microsoft.com/en-us/windows/win32/api/gdiplusinit/ns-gdiplusinit-gdiplusstartupinput).

Für die öffentliche Win32Form-API sollte deshalb kein `Gdiplus::Graphics&` verwendet werden. Ein `HDC` hält die Abhängigkeit optional.

### SVG

GDI und GDI+ besitzen keinen SVG-Parser.

Technisch mögliche Wege:

1. **Direct2D**

   Direct2D kann eigenständige SVG-Dokumente seit Windows 10 Creators Update rendern. Es unterstützt dabei eine definierte Teilmenge von SVG 1.1; nicht unterstützte Elemente und Attribute werden ignoriert. [Microsoft: Direct2D SVG Support](https://learn.microsoft.com/en-us/windows/win32/direct2d/svg-support)

   `ID2D1DeviceContext5::CreateSvgDocument()` liest SVG aus einem `IStream`. [API-Dokumentation](https://learn.microsoft.com/en-us/windows/win32/api/d2d1_3/nf-d2d1_3-id2d1devicecontext5-createsvgdocument)

   Für Win32Form wäre das jedoch ein größerer Architekturwechsel:

   - Direct2D-Factory und Device Context
   - Direct2D-/GDI-Interop oder Rasterisierung in eine zwischengespeicherte Bitmap
   - COM-Streams für Ressourcen
   - neue Link-Abhängigkeiten, mindestens `d2d1`
   - Mindestanforderung Windows 10 Version 1703 für eigenständige SVGs
   - Behandlung verlorener Render-Targets und DPI-abhängiger Caches

2. **Fremdbibliothek**

   Eine Bibliothek wie NanoSVG, LunaSVG oder resvg könnte SVG parsen beziehungsweise rasterisieren. Das bringt jedoch zusätzliche CMake-, Lizenz-, Update- und gegebenenfalls Deployment-Anforderungen mit sich. Für 16×16-Symbole wäre dies nur gerechtfertigt, wenn SVG ausdrücklich ein zwingendes Eingabeformat ist.

3. **Vorab-Rasterisierung**

   SVGs werden beim Erstellen der Anwendung in PNGs oder Mehrgrößen-Icons umgewandelt. Das ist zur Laufzeit deutlich einfacher, verliert aber die freie Vektorskalierung.

SVG-Dateien können bytegetreu als benutzerdefinierte Windows-Ressource eingebettet werden. Der Resource Compiler interpretiert diese Daten nicht; Win32Form müsste sie laden und anschließend selbst parsen. [Microsoft: User-Defined Resource](https://learn.microsoft.com/en-us/windows/win32/menurc/user-defined-resource)

Im aktuellen Projekt existiert keine `.rc`-Datei. Aufgrund der Vorgabe, keine neuen Dateien anzulegen, könnte eine spätere Implementierung nur Ressourcen des aufrufenden Programms konsumieren oder SVG-Daten direkt entgegennehmen. Das Anlegen eigener Ressourcen würde eine Erweiterung des erlaubten Dateiumfangs erfordern.

### Bitmap, PNG und ICO

Für bereits gestaltete Symbole sind Rasterformate robuster als eine neue SVG-Infrastruktur:

- BMP ist direkt mit GDI nutzbar, besitzt aber keine moderne Alpha-Behandlung.
- ICO lässt sich mit `LoadImage`/`DrawIconEx` gut in GDI verwenden und kann mehrere DPI-Größen enthalten.
- Seit Windows Vista dürfen Icon-Ressourcen PNG-komprimierte Bilder enthalten. [Microsoft: Resource File Formats](https://learn.microsoft.com/en-us/windows/win32/menurc/resource-file-formats)
- WIC besitzt einen eingebauten PNG-Decoder und ist seit Windows Vista verfügbar. SVG gehört nicht zu den nativen WIC-Codecs. [Microsoft: WIC Overview](https://learn.microsoft.com/en-us/windows/win32/wic/-wic-about-windows-imaging-codec)

Für ein allgemeines Win32Form-Bildmodell wäre WIC solide, aber deutlich aufwendiger als ein HDC-Callback. Für die konkrete Anforderung sind direkte GDI-/GDI+-Callbacks zunächst ausreichend.

## 6. API-Vorschlag 1: minimale Erweiterung

Die existierenden Textsegmente bleiben die Grundlage. Einzelne Segmente können anschließend durch einen Grafik-Callback ersetzt werden:

```cpp
struct SegmentDrawContext final
{
    HDC hdc{};
    RECT segmentBounds{};
    RECT contentBounds{};
    UINT dpi{USER_DEFAULT_SCREEN_DPI};
    COLORREF foregroundColor{};
    COLORREF backgroundColor{};
    std::size_t index{};
    bool selected{};
    bool hovered{};
    bool focused{};
    bool enabled{};
};

using SegmentDrawCallback =
    std::function<void(SegmentDrawContext const &)>;

void setSegmentControlItemDraw(
    std::string const & name,
    std::size_t index,
    SegmentDrawCallback const & draw) const;

void clearSegmentControlItemDraw(
    std::string const & name,
    std::size_t index) const;
```

Intern wäre dafür ein paralleler Vektor nötig:

```cpp
std::vector<std::optional<SegmentDrawCallback>>
    segmentedControlItemDrawCallbacks{};
```

Vorteile:

- Bestehende `createSegmentControl()`-Aufrufe bleiben exakt unverändert.
- Wenige neue Typen und Methoden.
- Text und Grafik können gemischt werden.
- Der bestehende Text kann als Fallback beziehungsweise zugängliche Bezeichnung erhalten bleiben.

Nachteile:

- Zwei parallele Vektoren müssen stets dieselbe Größe besitzen.
- Der Aufruf `create...()` und anschließende Setter sind weniger deklarativ.
- Symbol und Text gleichzeitig sind nicht sauber modelliert.

Diese Variante eignet sich für eine sehr kleine, kurzfristige Erweiterung.

## 7. API-Vorschlag 2: gemeinsames Inhaltsmodell

```cpp
struct SegmentText final
{
    std::string text{};
};

struct SegmentGraphic final
{
    std::string accessibleName{};
    SegmentDrawCallback draw{};
};

using SegmentContent =
    std::variant<SegmentText, SegmentGraphic>;

struct SegmentItem final
{
    SegmentContent content;
};

void createSegmentControl(
    std::string const & name,
    std::vector<std::string> const & items,
    std::optional<std::size_t> selectedIndex = std::nullopt,
    int itemWidth = 75);

void createSegmentControl(
    std::string const & name,
    std::vector<SegmentItem> const & items,
    std::optional<std::size_t> selectedIndex = std::nullopt,
    int itemWidth = 75);
```

Die alte Überladung konvertiert intern:

```text
std::vector<std::string>
    -> std::vector<SegmentItem>
    -> SegmentText
```

Später könnte bei tatsächlichem Bedarf beispielsweise ergänzt werden:

```cpp
struct SegmentTextAndGraphic final
{
    std::string text{};
    SegmentDrawCallback draw{};
};
```

Vorteile:

- Ein Vektor bestimmt Inhalt und Segmentanzahl.
- Keine parallelen Strukturen.
- Text-/Grafikauswahl pro Segment.
- Bestehender Quellcode bleibt unverändert.
- Erweiterbar, ohne jetzt Layoutoptionen vorwegzunehmen.
- `SegmentContent` ist eindeutig vom Auswahl-„State“ getrennt.

Nachteile:

- Neue öffentliche Typen und `<variant>`.
- Änderung des `ControlData`-Layouts und damit eine ABI-Änderung, falls Win32Form binär verteilt wird.
- Etwas größerer Implementierungs- und Testumfang.

## 8. Empfohlenes Zeichen- und Farbmodell

Win32Form sollte weiterhin selbst zeichnen:

- Segmenthintergrund
- Hover- und Auswahlfläche
- Trennlinien
- Rahmen
- Fokusrechteck

Der Callback zeichnet ausschließlich den Inhalt innerhalb von `contentBounds`.

`contentBounds` sollte ein zentriertes Rechteck von ungefähr 16×16 logischen Pixeln sein:

```cpp
auto const iconExtent =
    MulDiv(16, static_cast<int>(dpi), USER_DEFAULT_SCREEN_DPI);
```

Zusätzlich sollte ein DPI-skalierter Innenabstand eingehalten und das Rechteck auf den verfügbaren Segmentbereich begrenzt werden.

Für Farben empfiehlt sich ein kooperatives Modell:

- Win32Form berechnet `foregroundColor` und `backgroundColor` passend zu Normal, Hover, Selected und Disabled.
- Ein monochromes Symbol verwendet die übergebene Vordergrundfarbe.
- Ein mehrfarbiges Symbol darf diese Farbe ignorieren.
- Win32Form versucht nicht, beliebige SVGs, PNGs oder Icons automatisch umzufärben.
- Der Callback muss mindestens den Disabled-Zustand berücksichtigen oder die übergebene Disabled-Farbe benutzen.

Eine erzwungene automatische Umfärbung wäre bei Rasterbildern und mehrfarbigen SVGs unzuverlässig.

## 9. DPI-Bewertung

Im untersuchten Code gibt es derzeit keine DPI-Infrastruktur:

- keine DPI-Awareness-Deklaration,
- kein `GetDpiForWindow`,
- kein `WM_DPICHANGED`,
- keine Skalierung von Item-Breite, Höhe, Eckenradius oder Pens.

`GetDpiForWindow()` liefert abhängig von der DPI-Awareness des Fensters 96 DPI, System-DPI oder Monitor-DPI. [Microsoft-Dokumentation](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getdpiforwindow)

Für echte Per-Monitor-Skalierung muss die Anwendung ihre DPI-Awareness vorzugsweise im Manifest deklarieren. Microsoft empfiehlt das Manifest gegenüber einer nachträglichen API-Einstellung. [DPI-Awareness festlegen](https://learn.microsoft.com/en-us/windows/win32/hidpi/setting-the-default-dpi-awareness-for-a-process)

Daraus folgen zwei Ebenen:

1. **SegmentControl-Erweiterung**

   Den Symbolbereich aus dem aktuellen Fenster-DPI berechnen, zentrieren und bei DPI-Änderungen neu zeichnen beziehungsweise gecachte Rasterbilder verwerfen.

2. **Gesamtes Win32Form**

   Control-Höhen, Item-Breiten, Fonts, Eckenradius und Layout ebenfalls DPI-skalieren. Das ist ein größeres, separates Thema.

Nur das Symbol auf 150 oder 200 Prozent zu vergrößern, während das Control weiterhin 25 Pixel hoch bleibt, ist nicht ausreichend. Die neue Grafikfunktion sollte daher DPI-fähig entworfen werden, aber die Bedeutung bestehender `itemWidth`-Werte nicht stillschweigend verändern.

Falls ältere Systeme als Windows 10 Version 1607 unterstützt werden müssen, wäre für die DPI-Ermittlung ein Fallback über `GetDeviceCaps(LOGPIXELSX)` oder dynamisch aufgelöste APIs nötig.

## 10. CMake- und Deployment-Auswirkungen

Die aktuelle Testanwendung kompiliert `win32form.cpp` direkt in das Executable und besitzt kein eigenständiges Win32Form-Library-Target; siehe `CMakeLists.txt` im Testprojekt.

| Lösung | Zusätzliche Build-Anforderungen |
|---|---|
| reiner HDC-/GDI-Callback | keine neue Win32Form-Abhängigkeit |
| GDI+ | `gdiplus` linken und Prozesslebensdauer initialisieren |
| WIC/PNG | typischerweise `windowscodecs` und COM/OLE-Abhängigkeiten |
| Direct2D/SVG | mindestens `d2d1`, zusätzliche Direct2D-/COM-Infrastruktur |
| Fremd-SVG-Bibliothek | Paket/Vendoring, Lizenz und eventuell Runtime-DLL |
| eingebettete Ressourcen | `.rc` als Target-Quelle und Resource Compiler |

Da Win32Form momentan kein eigenes CMake-Target ist, müssten neue Link-Abhängigkeiten bei jedem konsumierenden Executable ergänzt werden. Ein target-basiertes Win32Form-Library-Target wäre langfristig sauberer, wäre aber eine separate und deutlich größere Umstrukturierung.

## 11. Empfohlene Implementierungsreihenfolge

1. Entscheiden, ob direkt das gemeinsame Inhaltsmodell oder zunächst die Minimal-Setter umgesetzt werden.
2. `SegmentDrawContext` und `SegmentDrawCallback` definieren.
3. Bestehende Text-Überladung unverändert lassen.
4. Intern eine einheitliche Methode für Validierung und Fenstererzeugung verwenden.
5. Zeichnung in „Segmentzustand zeichnen“ und „Segmentinhalt zeichnen“ aufteilen.
6. 16×16-logisches `contentBounds` DPI-abhängig berechnen und zentrieren.
7. Callback mit `SaveDC`/`RestoreDC` und eigener Clip-Region isolieren.
8. Backbuffer und weitere temporäre GDI-Ressourcen mit kleinen internen RAII-Guards absichern.
9. Bei Content-Änderung, DPI-Änderung, Enabled, Fokus, Hover, Auswahl und Größe invalidieren.
10. Ressourcen außerhalb des Paint-Vorgangs laden beziehungsweise rasterisieren und nach DPI/Farbe cachen.
11. Bestehende Textaufrufe sowie gemischte Text-/Grafiksegmente prüfen.

## 12. Risiken und offene Punkte

- **ABI:** Änderungen an `ControlData` sind binär inkompatibel, auch wenn bestehender Quellcode kompatibel bleibt.
- **Callback-Lebensdauer:** `std::function` wird von Win32Form kopiert; Referenzen in Captures bleiben Verantwortung des Aufrufers. Ressourcen sollten per Wert oder `std::shared_ptr` erfasst werden.
- **HDC-Sicherheit:** Ohne `SaveDC`/`RestoreDC` könnte ein Callback Font, Pen, Transform, Clip oder Zeichenmodus des übrigen Controls beschädigen.
- **Exceptions:** Exceptions dürfen nicht durch eine Win32-Window-Procedure entweichen. Gleichzeitig müssen alle GDI-Ressourcen bei einem Fehler sicher freigegeben werden.
- **Paint-Performance:** Dateien, SVGs oder PNGs dürfen nicht bei jedem `WM_PAINT` neu geladen oder dekodiert werden.
- **Kleine Control-Höhe:** 25 Pixel reichen nur bei 96 DPI komfortabel für ein 16-Pixel-Symbol.
- **Theme-Redraw:** Formfarben werden beim Paint gelesen, Änderungen an geerbten Formfarben invalidieren das Child-Control aktuell aber nicht ausdrücklich.
- **SVG-Kompatibilität:** Direct2D unterstützt nur einen Teil von SVG; komplexe Dateien müssen vorab validiert werden.
- **Windows-Version:** Die derzeit unterstützte Mindestversion ist im Projekt nicht dokumentiert.
- **Ressourcendateien:** Das aktuelle Dateilimit verhindert das Hinzufügen einer neuen `.rc`-Datei.
- **Barrierefreiheit:** Das bestehende Custom-Control stellt keine erkennbare segmentweise UI-Automation-Struktur bereit. Für reine Symbole sollte zumindest eine `accessibleName`-Bezeichnung vorgesehen werden.
- **Disabled-Auswahl:** Zu entscheiden ist, ob die bestehende visuelle Unterdrückung der Auswahl im Disabled-Zustand beibehalten werden soll.

## 13. Aufwandsschätzung

| Umfang | Geschätzter Aufwand |
|---|---:|
| Minimal-API mit GDI-Callbacks, DPI-Rechteck und Tests | 1–2 Personentage |
| Gemeinsames Inhaltsmodell mit sauberem Render-Refactoring | 2–3 Personentage |
| zusätzlicher robuster ICO-/PNG-Cache | weitere 1–3 Personentage |
| Direct2D-SVG einschließlich Ressourcen und Caching | weitere 4–7 Personentage |
| grundlegende Per-Monitor-DPI-Nachrüstung für ganz Win32Form | separates Vorhaben, etwa 3–6 Personentage |

## Empfehlung

Das gemeinsame Inhaltsmodell mit `std::variant<SegmentText, SegmentGraphic>` sollte direkt umgesetzt werden. Der Mehraufwand gegenüber den Minimal-Settern ist überschaubar und vermeidet dauerhaft parallele, potenziell inkonsistente Datenstrukturen.

Als erste Grafiktechnik sollte ein HDC-basierter Callback verwendet werden. Er passt unmittelbar in den vorhandenen GDI-Backbuffer, erlaubt sowohl GDI als auch optional GDI+ und verursacht in Win32Form keine neue Grafikabhängigkeit.

SVG sollte erst ergänzt werden, wenn SVG als Eingabeformat tatsächlich zwingend ist. Für kleine Symbole sind direkte GDI-/GDI+-Zeichnung oder gecachte ICO-/PNG-Ressourcen wesentlich einfacher.

## Vor einer Implementierung zu entscheiden

- Gemeinsames Inhaltsmodell oder kurzfristige Setter-Variante?
- Müssen Text und Grafik innerhalb eines Controls mischbar sein? Empfehlung: ja.
- Ist SVG zwingend oder genügt ein Zeichen-Callback?
- Welche minimale Windows-Version wird unterstützt?
- Wer initialisiert GDI+, falls Aufrufer es in Callbacks verwenden?
- Sollen monochrome Symbole die bereitgestellte Vordergrundfarbe verbindlich verwenden?
- Soll die Auswahl im Disabled-Zustand weiterhin visuell verborgen werden?
- Muss jetzt bereits segmentweise Barrierefreiheit berücksichtigt werden?
- Dürfen später `.rc`- und CMake-Dateien ergänzt werden?
- Soll DPI zunächst nur für Symbole oder als separates Win32Form-Gesamtthema umgesetzt werden?
