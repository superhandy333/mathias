# CAD-Klassenhierarchie – Public API (Eigenschaften & Methoden)

## CadPoint

### Eigenschaften

|name|beschreibuxng|
|-|-|
|Name||
|OnRedrawNeeded||

### Methoden
| name | beschreibung |
|---|---|
| CadPoint() |  |
| ~CadPoint() |  |
| get_backcolor() const |  |
| get_color() const |  |
| get_linewidth() const |  |
| get_location() const |  |
| get_usedParentBackcolor() const |  |
| get_usedParentColor() const |  |
| get_usedParentLinewidth() const |  |
| get_visible() const |  |
| getRect() const |  |
| operator=(CadPoint &&) |  |
| operator=(CadPoint const &) |  |
| paint(Gdiplus::Graphics &, ParentValues const &, HWND) |  |
| set_backcolor(tools::SystemColor) |  |
| set_color(tools::SystemColor) |  |
| set_linewidth(int) |  |
| set_location(CadCoord const &) |  |
| set_location(int, int) |  |
| set_usedParentBackcolor(bool) |  |
| set_usedParentColor(bool) |  |
| set_usedParentLinewidth(bool) |  |
| set_visible(bool) |  |

---

## CadNode

### Eigenschaften
| name | beschreibung |
|---|---|
| Name |  |
| OnRedrawNeeded |  |

### Methoden
| name | beschreibung |
|---|---|
| CadNode() |  |
| ~CadNode() |  |
| add_circle(CadCoord const &, int const) |  |
| add_circle(int, int, int) |  |
| add_line() |  |
| add_line(int, int, int, int) |  |
| add_node(CadCoord const &) |  |
| add_node(int, int) |  |
| add_rect(CadCoord const &, int, int) |  |
| add_rect(int, int, int, int) |  |
| add_text(CadCoord const &, std::string const &) |  |
| add_text(int, int, std::string const &) |  |
| add_triangle() |  |
| add_triangle(CadCoord const &, CadCoord const &, CadCoord const &) |  |
| add_triangle(int, int, int, int, int, int) |  |
| get_backcolor() const |  |
| get_color() const |  |
| get_linewidth() const |  |
| get_location() const |  |
| get_mirror_horizontal() const |  |
| get_mirror_vertical() const |  |
| get_rotation() const |  |
| get_scale() const |  |
| get_usedParentBackcolor() const |  |
| get_usedParentColor() const |  |
| get_usedParentLinewidth() const |  |
| get_visible() const |  |
| getRect() const |  |
| getSvg() const |  |
| operator=(CadNode &&) |  |
| operator=(CadNode const &) |  |
| paint(Gdiplus::Graphics &, ParentValues const &, HWND) |  |
| set_backcolor(tools::SystemColor) |  |
| set_color(tools::SystemColor) |  |
| set_linewidth(int) |  |
| set_location(CadCoord const &) |  |
| set_location(int, int) |  |
| set_mirror_horizontal(bool) |  |
| set_mirror_vertical(bool) |  |
| set_rotation(float) |  |
| set_scale(float) |  |
| set_usedParentBackcolor(bool) |  |
| set_usedParentColor(bool) |  |
| set_usedParentLinewidth(bool) |  |
| set_visible(bool) |  |

---

## CadPrimitive

### Eigenschaften
| name | beschreibung |
|---|---|
| Name |  |
| OnRedrawNeeded |  |

### Methoden
| name | beschreibung |
|---|---|
| CadPrimitive() |  |
| ~CadPrimitive() |  |
| get_backcolor() const |  |
| get_color() const |  |
| get_filled() const |  |
| get_linewidth() const |  |
| get_location() const |  |
| get_usedParentBackcolor() const |  |
| get_usedParentColor() const |  |
| get_usedParentLinewidth() const |  |
| get_visible() const |  |
| getSvg() const |  |
| operator=(CadPrimitive &&) |  |
| operator=(CadPrimitive const &) |  |
| paint(Gdiplus::Graphics &, ParentValues const &, HWND) |  |
| set_backcolor(tools::SystemColor) |  |
| set_color(tools::SystemColor) |  |
| set_filled(bool) |  |
| set_linewidth(int) |  |
| set_location(CadCoord const &) |  |
| set_location(int, int) |  |
| set_usedParentBackcolor(bool) |  |
| set_usedParentColor(bool) |  |
| set_usedParentLinewidth(bool) |  |
| set_visible(bool) |  |

---

## CadLine

### Eigenschaften
| name | beschreibung |
|---|---|
| Name |  |
| OnRedrawNeeded |  |

### Methoden
| name | beschreibung |
|---|---|
| CadLine() |  |
| ~CadLine() |  |
| get_backcolor() const |  |
| get_color() const |  |
| get_end() const |  |
| get_filled() const |  |
| get_linewidth() const |  |
| get_location() const |  |
| get_start() const |  |
| get_usedParentBackcolor() const |  |
| get_usedParentColor() const |  |
| get_usedParentLinewidth() const |  |
| get_visible() const |  |
| getRect() const |  |
| getSvg() const |  |
| operator=(CadLine &&) |  |
| operator=(CadLine const &) |  |
| paint(Gdiplus::Graphics &, ParentValues const &, HWND) |  |
| set_backcolor(tools::SystemColor) |  |
| set_color(tools::SystemColor) |  |
| set_end(CadCoord const &) |  |
| set_end(int, int) |  |
| set_filled(bool) |  |
| set_linewidth(int) |  |
| set_location(CadCoord const &) |  |
| set_location(int, int) |  |
| set_start(CadCoord const &) |  |
| set_start(int, int) |  |
| set_usedParentBackcolor(bool) |  |
| set_usedParentColor(bool) |  |
| set_usedParentLinewidth(bool) |  |
| set_visible(bool) |  |

---

## CadCircle

### Eigenschaften
| name | beschreibung |
|---|---|
| Name |  |
| OnRedrawNeeded |  |

### Methoden
| name | beschreibung |
|---|---|
| CadCircle() |  |
| ~CadCircle() |  |
| get_backcolor() const |  |
| get_color() const |  |
| get_filled() const |  |
| get_linewidth() const |  |
| get_location() const |  |
| get_radius() const |  |
| get_usedParentBackcolor() const |  |
| get_usedParentColor() const |  |
| get_usedParentLinewidth() const |  |
| get_visible() const |  |
| getRect() const |  |
| getSvg() const |  |
| operator=(CadCircle &&) |  |
| operator=(CadCircle const &) |  |
| paint(Gdiplus::Graphics &, ParentValues const &, HWND) |  |
| set_backcolor(tools::SystemColor) |  |
| set_color(tools::SystemColor) |  |
| set_filled(bool) |  |
| set_linewidth(int) |  |
| set_location(CadCoord const &) |  |
| set_location(int, int) |  |
| set_radius(int) |  |
| set_usedParentBackcolor(bool) |  |
| set_usedParentColor(bool) |  |
| set_usedParentLinewidth(bool) |  |
| set_visible(bool) |  |

---

## CadRect

### Eigenschaften
| name | beschreibung |
|---|---|
| Name |  |
| OnRedrawNeeded |  |

### Methoden
| name | beschreibung |
|---|---|
| CadRect() |  |
| ~CadRect() |  |
| get_backcolor() const |  |
| get_color() const |  |
| get_filled() const |  |
| get_height() const |  |
| get_linewidth() const |  |
| get_location() const |  |
| get_usedParentBackcolor() const |  |
| get_usedParentColor() const |  |
| get_usedParentLinewidth() const |  |
| get_visible() const |  |
| get_width() const |  |
| getRect() const |  |
| getSvg() const |  |
| operator=(CadRect &&) |  |
| operator=(CadRect const &) |  |
| paint(Gdiplus::Graphics &, ParentValues const &, HWND) |  |
| set_backcolor(tools::SystemColor) |  |
| set_color(tools::SystemColor) |  |
| set_filled(bool) |  |
| set_height(int) |  |
| set_linewidth(int) |  |
| set_location(CadCoord const &) |  |
| set_location(int, int) |  |
| set_usedParentBackcolor(bool) |  |
| set_usedParentColor(bool) |  |
| set_usedParentLinewidth(bool) |  |
| set_visible(bool) |  |
| set_width(int) |  |

---

## CadTriangle

### Eigenschaften
| name | beschreibung |
|---|---|
| Name |  |
| OnRedrawNeeded |  |

### Methoden
| name | beschreibung |
|---|---|
| CadTriangle() |  |
| ~CadTriangle() |  |
| get_backcolor() const |  |
| get_color() const |  |
| get_filled() const |  |
| get_linewidth() const |  |
| get_location() const |  |
| get_point1() const |  |
| get_point2() const |  |
| get_point3() const |  |
| get_usedParentBackcolor() const |  |
| get_usedParentColor() const |  |
| get_usedParentLinewidth() const |  |
| get_visible() const |  |
| getRect() const |  |
| getSvg() const |  |
| operator=(CadTriangle &&) |  |
| operator=(CadTriangle const &) |  |
| paint(Gdiplus::Graphics &, ParentValues const &, HWND) |  |
| set_backcolor(tools::SystemColor) |  |
| set_color(tools::SystemColor) |  |
| set_filled(bool) |  |
| set_linewidth(int) |  |
| set_location(CadCoord const &) |  |
| set_location(int, int) |  |
| set_point1(CadCoord const &) |  |
| set_point1(int, int) |  |
| set_point2(CadCoord const &) |  |
| set_point2(int, int) |  |
| set_point3(CadCoord const &) |  |
| set_point3(int, int) |  |
| set_usedParentBackcolor(bool) |  |
| set_usedParentColor(bool) |  |
| set_usedParentLinewidth(bool) |  |
| set_visible(bool) |  |

---

## CadText

### Eigenschaften
| name | beschreibung |
|---|---|
| Name |  |
| OnRedrawNeeded |  |

### Methoden
| name | beschreibung |
|---|---|
| CadText() |  |
| ~CadText() |  |
| get_alignment() const |  |
| get_backcolor() const |  |
| get_bold() const |  |
| get_color() const |  |
| get_filled() const |  |
| get_fontfamily() const |  |
| get_italic() const |  |
| get_linewidth() const |  |
| get_location() const |  |
| get_strikeout() const |  |
| get_text() const |  |
| get_textsize() const |  |
| get_underline() const |  |
| get_usedParentBackcolor() const |  |
| get_usedParentColor() const |  |
| get_usedParentLinewidth() const |  |
| get_valignment() const |  |
| get_visible() const |  |
| getRect() const |  |
| getSvg() const |  |
| operator=(CadText &&) |  |
| operator=(CadText const &) |  |
| paint(Gdiplus::Graphics &, ParentValues const &, HWND) |  |
| set_alignment(TextAlignment) |  |
| set_backcolor(tools::SystemColor) |  |
| set_bold(bool) |  |
| set_color(tools::SystemColor) |  |
| set_filled(bool) |  |
| set_fontfamily(SystemFontFamily) |  |
| set_italic(bool) |  |
| set_linewidth(int) |  |
| set_location(CadCoord const &) |  |
| set_location(int, int) |  |
| set_strikeout(bool) |  |
| set_text(std::string const &) |  |
| set_textsize(int) |  |
| set_underline(bool) |  |
| set_usedParentBackcolor(bool) |  |
| set_usedParentColor(bool) |  |
| set_usedParentLinewidth(bool) |  |
| set_valignment(TextVAlignment) |  |
| set_visible(bool) |  |