
# Dokumentation der Klassen `NanaForm` und `NanaDialog`

## Übersicht

Diese Klassen kapseln die GUI-Elemente der [Nana C++ GUI Library](http://nanapro.org/en-us/) und ermöglichen eine einfache Erstellung und Verwaltung von Formularen und Steuerelementen.

---

## Klasse: `NanaForm`

### Konstruktoren

```cpp
NanaForm();
NanaForm(FormProperties const & prop);
```

### Destruktor

```cpp
~NanaForm();
```

### Form-Eigenschaften

- `getNanaForm()` – Gibt eine Referenz auf das interne `nana::form`-Objekt zurück.
- `setFormCaption(caption)` – Setzt die Fensterüberschrift.
- `setFormPosition(left, top)` – Positioniert das Formular.
- `setFormSize(width, height)` – Setzt die Fenstergröße.
- `setFormWidth(width)` / `setFormHeight(height)` – Einzelne Dimensionen ändern.
- `getFormPosition()` / `getFormSize()` – Gibt Position/Größe zurück.
- `getFormWidth()` / `getFormHeight()` – Gibt jeweilige Einzelwerte zurück.

### Steuerung (Controls)

Setzen von Eigenschaften:
- `setControlCaption(name, caption)`
- `setControlPosition(name, left, top)`
- `setControlSize(name, width, height)`
- `setControlVisible(name, visible)`
- `setControlEnabled(name, enabled)`
- `setControlForegroundColor(name, color)`
- `setControlBackgroundColor(name, color)`

Abfragen von Eigenschaften:
- `getControlCaption(name)`
- `getControlPosition(name)`
- `getControlSize(name)`
- `isControlVisible(name)`
- `isControlEnabled(name)`

### Label

- `createLabel(name, caption, nameParentPanel)` – Erstellt ein Label.
- `getLabel(name)` – Gibt ein Label zurück.

### Button

- `createButton(name, caption, nameParentPanel)` – Erstellt einen Button.
- `getButton(name)` – Gibt den Button zurück.
- `setButtonClicked(name, callback)` – Setzt Klick-Callback.

### TextBox

- `createTextBox(name, text, nameParentPanel)`
- `getTextBox(name)`
- `setTextBoxText(name, text)`
- `setTextBoxEditable(name, editable)`
- `setTextBoxMultiline(name, multiline)`
- `setTextBoxWordwrap(name, wordwrap)`
- `getTextBoxText(name)`
- `isTextBoxEditable(name)`
- `isTextBoxMultiline(name)`
- `isTextBoxWordwrap(name)`
- `setTextBoxTextChanged(name, callback)`

### ComboBox

- `createComboBox(name, items, nameParentPanel)`
- `getComboBox(name)`
- `getComboBoxSelected(name)`
- `setComboBoxSelected(name, item)`

### CheckBox

- `createCheckBox(name, caption, nameParentPanel)`
- `getCheckBox(name)`
- `isCheckBoxChecked(name)`

### ListBox

- `createListBox(name, items, nameParentPanel)`
- `getListBox(name)`
- `setListBoxData(name, data)`
- `setListBoxDoubleClick(name, callback)`

### Panel

- `createPanel(name)`

### Form-Anzeigen & Interaktion

- `Show()` – Zeigt das Formular (nicht virtuell überschreibbar).
- `ShowModal()` – Zeigt das Formular modal.
- `Close()` – Schließt das Formular.
- `ShowMessage(message, title, level)` – Zeigt eine Info-Nachricht an.
- `AskMessage(frage)` – Fragt Benutzer mit Ja/Nein zurück.

---

## Klasse: `NanaDialog`

Erbt von `NanaForm` und verwendet standardmäßig Dialog-Eigenschaften.

### Konstruktor

```cpp
NanaDialog();
```

### Destruktor

```cpp
~NanaDialog();
```

---

## Private Member (nur intern)

- `nanaform` – Instanz des Nana-Fensters.
- Verschiedene Maps (`labels`, `buttons`, …) zur Verwaltung der Controls.
- `isControlNameExist(name)` – Prüft Existenz eines Controls.
- `getParentWindow(nameParentPanel)` – Holt übergeordnetes Panel-Fenster.
