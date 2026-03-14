# Desktop Automation Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-blue)](https://github.com/openclaw/openclaw)

**⚠️  PRIVACY WARNING**: The macro recorder captures **ALL** keyboard events (including passwords, credit card numbers, private messages) and window titles. **Never record while entering credentials.** Only use for non-sensitive workflows. Store recorded macros securely.

---

## 🎯 **About**

OpenClaw skill to automate desktop interactions on Windows/macOS/Linux using Python. Uses **PyAutoGUI** for basic actions, **OpenCV** for image recognition, and **pytesseract** for OCR.

Perfect for:
- Automating legacy apps without an API
- Filling repetitive forms
- Launching GUI workflows from commands
- Recording and replaying macros

---

## 🔐 **Security & Safety Features**

### Safe Mode
- When **safe mode** is enabled (default), certain risky actions (`type`, `press_key`, `click`, `drag`) are blocked to prevent accidental damage.
- You can temporarily disable safe mode with the `set_safe_mode` action.
- Safe mode also blocks parameters containing dangerous patterns (e.g., `rm `, `del `, paths in `C:\Windows\`, `/etc/`).

### Dry-Run (Sandbox)
- All actions support an optional `dry_run` parameter (boolean). When `true`, the action is only logged and simulated; no real effect occurs.
- Useful for testing macros or conditional monitoring safely.

### Audit Logging
- All actions are logged to a daily rolling file under `~/.openclaw/skills/desktop-automation-logs/` (or `%APPDATA%` on Windows).
- Logs include timestamp, action, parameters, and outcomes.
- Blocked actions by safe mode are also logged.

### Safety Control Action
| Action | Parameters | Description |
|--------|------------|-------------|
| `set_safe_mode` | `{"enabled": false}` | Enable or disable safe mode globally |

---

## 🔍 **Monitor Screen** (Conditional Automation)

---

## 📦 **Installation**

### Prerequisites
- Python 3.10+
- Pip

### Install dependencies
```bash
pip install -r requirements.txt
```

### Enable the skill
Place the folder `desktop-automation-100per100-local` in your OpenClaw skills directory:
- **Windows**: `C:\Users\<YourUsername>\.openclaw\workspace\skills\`
- **Linux/macOS**: `~/.openclaw/workspace/skills/`

Then restart OpenClaw:
```bash
openclaw gateway restart
```

---

## 📋 **Dependencies Explained**

The skill requires several Python packages. Here's what each one does:

### Core (Required)
- **pyautogui** (0.9.54) — Main library for mouse/keyboard control, screenshots
- **pygetwindow** (0.0.9) — Window enumeration and activation
- **Pillow** (10.4.0) — Image processing (used by pyautogui)
- **pynput** (1.7.6) — Keyboard/mouse listeners for macro recording

### Optional — Image & OCR
- **opencv-python** (4.10.0.84) — Required for `find_image` and `wait_for_image` (image recognition)
- **pytesseract** (0.3.10) — Required for `find_text_on_screen` and `extract_screen_data` (OCR). Also requires the Tesseract binary installed on your OS.

### Optional — Clipboard
- **pyperclip** (1.9.0) — Required for `copy_to_clipboard` and `paste_from_clipboard`. Usually works out of the box on Windows/macOS; on Linux you may need `xclip` or `xsel` system packages.

### Optional — Data Integration (Excel/CSV)
- **openpyxl** (3.1.5) — Required for `excel_read` and `excel_write` (Excel .xlsx support)
- **pandas** (2.2.3) — Required for `data_to_csv` (CSV conversion). Depends on `openpyxl` for Excel operations as well.

### Installing only what you need
If you don't need all features, you can install selectively:
```bash
# Minimal (basic actions only)
pip install pyautogui pygetwindow Pillow pynput

# + Image recognition
pip install opencv-python

# + OCR
pip install pytesseract

# + Clipboard
pip install pyperclip

# + Excel/CSV
pip install openpyxl pandas
```

**Note**: The skill will gracefully degrade if optional dependencies are missing; actions requiring them will return an error with a clear message.

---

## 🚀 **Available Actions**

### Basic
| Action | Parameters | Description |
|--------|------------|-------------|
| `click` | `x`, `y`, `button?` ("left"/"right"/"middle") | Click at coordinates |
| `type` | `text`, `interval?` (float) | Type with optional interval |
| `screenshot` | `path?` (default: `~/Desktop/screenshot.png`) | Capture screen |
| `get_active_window` | — | Returns `{title, x, y, width, height}` of active window |
| `list_windows` | — | List all windows (`[{title, x, y, width, height, is_active}]`) |
| `activate_window` | `title_substring` | Activate first matching window |
| `move_mouse` | `x`, `y` | Move cursor |
| `press_key` | `key` (e.g. 'enter', 'tab', 'escape', 'space') | Press a single key |
| `scroll` | `amount` (positive=up, negative=down) | Scroll |
| `copy_to_clipboard` | `text` | Copy to clipboard (requires `pyperclip`) |
| `paste_from_clipboard` | — | Paste (Ctrl+V) |
| `drag` | `start_x`, `start_y`, `end_x`, `end_y`, `duration?`, `button?` | Drag-and-drop |

### Advanced
| Action | Parameters | Description |
|--------|------------|-------------|
| `find_image` | `template_path`, `confidence?` (0.0-1.0) | Find image on screen (OpenCV). Returns `{x, y, confidence}` or `not_found` |
| `wait_for_image` | `template_path`, `timeout?`, `interval?`, `confidence?` | Wait for an image to appear |
| `monitor_screen` | `checks`, `timeout?`, `interval?`, `stop_condition?`, `fallback_confidence?` | Surveille l'écran et exécute des actions quand des conditions sont remplies (voir détails ci-dessous) |
| `find_text_on_screen` | `text`, `lang?` ('fra' default) | OCR: search text on screen (requires Tesseract) |

---

## 📊 **Data Integration**

### Extract Screen Data (OCR)
Extract structured text data from the screen (or a region) using Tesseract OCR. Returns bounding boxes and confidence scores.

```javascript
sessions_spawn({
  task: 'extract_screen_data',
  params: {
    region: {x: 100, y: 100, width: 500, height: 300},  // optional
    output_format: 'json'  // default 'json'
  },
  label: 'desktop-automation-100per100-local'
});
```

**Result**:
```json
{
  "status": "ok",
  "data": [
    {"text": "Hello", "left": 120, "top": 130, "width": 50, "height": 20, "conf": 95.2},
    ...
  ],
  "count": 42
}
```

### Read Excel
Read data from an Excel (.xlsx) file into a list of dictionaries (rows).

```javascript
sessions_spawn({
  task: 'excel_read',
  params: {
    filepath: "C:/data/input.xlsx",
    sheet_name: 0,        // or "Sheet1"
    range: "A1:C10"       // optional, reads entire sheet if omitted
  },
  label: 'desktop-automation-100per100-local'
});
```

**Result**:
```json
{
  "status": "ok",
  "data": [
    {"Name": "Alice", "Age": 30, "City": "Paris"},
    ...
  ],
  "columns": ["Name", "Age", "City"],
  "rows": 100
}
```

### Write Excel
Write a list of dictionaries (or list of lists) to a new Excel file.

```javascript
sessions_spawn({
  task: 'excel_write',
  params: {
    filepath: "C:/data/output.xlsx",
    data: [
      {"Name": "Bob", "Age": 25},
      {"Name": "Charlie", "Age": 35}
    ],
    sheet_name: "Results",
    start_cell: "A1"
  },
  label: 'desktop-automation-100per100-local'
});
```

**Result**:
```json
{"status":"ok","filepath":"C:/data/output.xlsx","rows":2,"columns":2}
```

### Data to CSV
Convert a list of dictionaries to CSV format (string or file).

```javascript
sessions_spawn({
  task: 'data_to_csv',
  params: {
    data: [{"a":1},{"a":2}],
    filepath: "C:/data/out.csv"  // optional; if omitted, returns CSV string
  },
  label: 'desktop-automation-100per100-local'
});
```

---

## 📝 **Usage Examples**

### 1. Click and type
```javascript
sessions_spawn({ task: 'click {"x":100,"y":200}', label: 'desktop-automation-100per100-local' });
sessions_spawn({ task: 'type {"text":"Hello World"}', label: 'desktop-automation-100per100-local' });
```

### 2. Screenshot and list windows
```javascript
sessions_spawn({ task: 'list_windows', label: 'desktop-automation-100per100-local' });
sessions_spawn({ task: 'screenshot {"path":"~/Desktop/my_screen.png"}', label: 'desktop-automation-100per100-local' });
```

### 3. Activate window
```javascript
sessions_spawn({ task: 'activate_window {"title_substring":"Notepad"}', label: 'desktop-automation-100per100-local' });
```

### 4. Image search (with OpenCV)
```javascript
sessions_spawn({
  task: 'find_image {"template_path":"C:/path/button.png","confidence":0.9}',
  label: 'desktop-automation-100per100-local'
});
```

### 5. Full macro (record/play)
```javascript
// Start GUI recording
sessions_spawn({ task: 'record_macro', label: 'desktop-automation-100per100-local' });
// Stop → file generated in recorded_macro/
// Replay
sessions_spawn({
  task: 'play_macro {"macro_path":"C:/.../macro_2026-03-14_22-00-00.json"}',
  label: 'desktop-automation-100per100-local'
});
```

---

## 🔐 **Security & Safety Features**

### Safe Mode
- When **safe mode** is enabled (default), certain risky actions (`type`, `press_key`, `click`, `drag`) are blocked to prevent accidental damage.
- You can temporarily disable safe mode with the `set_safe_mode` action.
- Safe mode also blocks parameters containing dangerous patterns (e.g., `rm `, `del `, paths in `C:\Windows\`, `/etc/`).

### Dry-Run (Sandbox)
- Add `"dry_run": true` to any action parameters to simulate execution without side effects.
- Useful for testing macros or conditional monitoring safely.

### Audit Logging
- All actions are logged to a daily rolling file:
  - **Windows**: `C:\Users\<User>\.openclaw\skills\desktop-automation-logs\automation_YYYY-MM-DD.log`
  - **Linux/macOS**: `~/.openclaw/skills/desktop-automation-logs/automation_YYYY-MM-DD.log`
- Logs include timestamp, action, parameters, and outcomes.
- Blocked actions by safe mode are also logged.

### Actions for Safety Control
| Action | Parameters | Description |
|--------|------------|-------------|
| `set_safe_mode` | `{"enabled": false}` | Enable or disable safe mode globally |
| Any action with `dry_run` param | `dry_run: true` | Execute in simulation mode (no real effect) |

---

## 🔍 **Monitor Screen** (Conditional Automation)

Surveille l'écran et exécute des Actions quand des éléments apparaissent.

### Format de `checks` (liste de conditions)

Chaque condition :
```json
{
  "type": "image" | "text",
  "template_path": "C:/path/to/template.png",   // si type="image"
  "text": "Notification text",                   // si type="text"
  "confidence": 0.9,                             // seuil de confiance (image seulement)
  "lang": "fra",                                 // langue OCR (texte seulement)
  "action": "click",                             // action à exécuter
  "action_params": { "button": "left" }          // paramètres de l'action
}
```

### Paramètres de `monitor_screen`

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| `checks` | array | obligatoire | Liste des conditions à surveiller |
| `timeout` | float | 60 | Durée max de surveillance en secondes |
| `interval` | float | 0.5 | Délai entre les vérifications |
| `stop_condition` | string | "first_match" | "first_match", "all_matched", ou null (continue until timeout) |
| `fallback_confidence` | float \| array | 0.85 | Si l'image n'est pas trouvée avec la confiance initiale, essayer avec des seuils inférieurs automatiquement |

### Comportement

- L'action exécutée reçoit automatiquement les coordonnées `x,y` de l'élément détecté (si présentes).
- Les actions autorisées : `click`, `type`, `press_key`, `scroll`, `move_mouse`, `activate_window`, `drag`, `copy_to_clipboard`, `paste_from_clipboard`.
- Retourne un objet avec `matches` (liste des éléments détectés) et `actions_executed` (résultats des actions).

### Exemple : cliquer sur une notification dès qu'elle apparaît

```javascript
sessions_spawn({
  task: 'monitor_screen',
  params: {
    checks: [
      {
        type: "image",
        template_path: "C:/templates/notification_ok.png",
        confidence: 0.9,
        action: "click",
        action_params: { button: "left" }
      }
    ],
    timeout: 30,
    interval: 0.5,
    stop_condition: "first_match",
    fallback_confidence: 0.85
  },
  label: 'desktop-automation-100per100-local'
});
```

### Exemple : taper du texte quand un champ est détecté par OCR

```javascript
sessions_spawn({
  task: 'monitor_screen',
  params: {
    checks: [
      {
        type: "text",
        text: "Username",
        lang: "eng",
        action: "type",
        action_params: { "text": "my_user", "interval": 0.05 }
      }
    ],
    timeout: 20,
    stop_condition: "first_match"
  },
  label: 'desktop-automation-100per100-local'
});
```

The skill includes a Tkinter GUI to record macros:
- Launch: `python scripts/record_macro.py`
- Records: mouse, clicks, scrolling, keyboard, window changes
- Saves JSON in `recorded_macro/` (timestamped name)
- Playback: `python scripts/play_macro.py <file.json> [speed]`

**CLI example**:
```bash
python scripts/record_macro.py
python scripts/play_macro.py recorded_macro/macro_2026-03-14_22-00-00.json 1.0
```

---

## 🔧 **Configuration**

### OCR (pytesseract)
- Install Tesseract on your system: https://github.com/tesseract-ocr/tesseract
- On Windows: add to PATH or set `pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'`
- Language packs: `fra`, `eng`, etc.

### Pyperclip (copy/paste)
- On Linux: `xclip` or `xsel` required
- On Windows/macOS: usually works out of the box

---

## ⚠️ **Safety & Best Practices**

- **Full control**: the skill controls mouse and keyboard — use carefully
- **No network access**: isolated, no external data
- **Coordinates**: verify before clicking (use `screenshot` for reference)
- **Windows**: `activate_window` assumes a unique title — be precise
- **Macros**: always test in dry-run or slow speed first
- **Permissions**: on Windows, may require admin rights for some apps

---

## 🐛 **Troubleshooting**

| Problem | Solution |
|----------|----------|
| `pyautogui.FailSafeException` | Move mouse to corner (0,0) to disable failsafe, or `pyautogui.FAILSAFE = False` |
| OCR finds nothing | Check Tesseract installation + language. `pytesseract.get_tesseract_version()` |
| `find_image` fails | Ensure template matches exactly (same scale/color). Adjust `confidence` (0.7-0.95) |
| `activate_window` not found | Use `list_windows` to see exact titles |
| ImportError (missing module) | `pip install -r requirements.txt` (check Python 3.10+) |
| Tkinter GUI won't open | On Linux: `apt-get install python3-tk`. On Windows: usually present |

---

## 🤝 **Contributing**

This skill is open-source under the MIT license. Contributions are welcome!

### How to contribute
1. Fork the repository
2. Create a branch (`git checkout -b feature/improvement`)
3. Commit (`git commit -am 'Add X'`)
4. Push (`git push origin feature/improvement`)
5. Open a Pull Request

### Guidelines
- Follow existing code style (PEP 8, 4 spaces)
- Add docstrings in English
- Test locally
- Update README.md if needed

See `CONTRIBUTING.md` for more details.

---

## 📄 **License**

MIT License — see `LICENSE` file.

Copyright (c) 2026 — Jordane Guemara & contributors

---

## 🙏 **Authors**

**Original creator**: Jordane Guemara — https://github.com/JordaneParis

See `AUTHORS.md` for full contributors list.

---

## 🔗 **Links**

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [Clawhub — community skills](https://clawhub.com)
- [GitHub Repository](https://github.com/JordaneParis/desktop-automation-100per100-local)

---

**Version**: 2.0.0 — ultra-robust, thread-safe, with Tkinter logging
