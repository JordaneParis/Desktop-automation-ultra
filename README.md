# Desktop Automation Skill for OpenClaw

**⚠️  PRIVACY WARNING**: The macro recorder captures **ALL** keyboard events (including passwords, credit card numbers, private messages) and window titles. **Never record while entering credentials.** Only use for non-sensitive workflows. Store recorded macros securely.

Full desktop automation: control mouse, keyboard, windows, OCR, image recognition — all local.

- **PyAutoGUI** for basic actions
- **OpenCV** for image recognition
- **pytesseract** for OCR

---

## 🔐 Security & Privacy

### ⚠️ Keyboard Recording Warning
The **macro recorder** captures **ALL** keyboard events, including passwords, credit card numbers, and personal data. **Never** record macros while entering credentials. Use only for non-sensitive workflows. Recorded macro files contain raw keystrokes — store them securely.

**Summary**:
- No network access — completely local
- No credential storage (unless you record them)
- All dependencies are standard Python packages
- You are responsible for macro file security

---

## Installation

### Prerequisites
- Python 3.10+
- Pip

### Install dependencies
```bash
pip install -r requirements.txt
```

### Enable skill
Place the folder `desktop-automation-100per100-local` in your OpenClaw skills directory:
- **Windows**: `C:\Users\<YourUsername>\.openclaw\workspace\skills\`
- **Linux/macOS**: `~/.openclaw/workspace/skills/`

Then restart OpenClaw:
```bash
openclaw gateway restart
```

---

## Available Actions

### Basic Actions
- `click` — click at coordinates
- `type` — type text
- `screenshot` — capture screen
- `get_active_window` — get active window info
- `list_windows` — list all windows
- `activate_window` — activate window by title substring
- `move_mouse` — move cursor
- `press_key` — press a single key
- `scroll` — scroll screen
- `copy_to_clipboard` — copy text to clipboard
- `paste_from_clipboard` — paste from clipboard
- `drag` — drag from start to end

### Advanced Actions
- `find_image` — locate image on screen (OpenCV)
- `wait_for_image` — wait until image appears
- `monitor_screen` — conditional monitoring: watch for images/text and trigger actions
- `find_text_on_screen` — OCR text search
- `extract_screen_data` — extract structured text data (OCR) from screen or region
- `excel_read` — read data from an Excel file into list of dicts
- `excel_write` — write list of dicts/lists to an Excel file
- `data_to_csv` — convert list of dicts to CSV (string or file)

---

## Examples

```javascript
// Click and type
sessions_spawn({ task: 'click {"x":100,"y":200}', label: 'desktop-automation-100per100-local' });
sessions_spawn({ task: 'type {"text":"Hello World"}', label: 'desktop-automation-100per100-local' });

// Take screenshot
sessions_spawn({ task: 'screenshot {"path":"~/Desktop/screen.png"}', label: 'desktop-automation-100per100-local' });

// Activate Notepad
sessions_spawn({ task: 'activate_window {"title_substring":"Notepad"}', label: 'desktop-automation-100per100-local' });

// Find an image
sessions_spawn({
  task: 'find_image {"template_path":"C:/button.png","confidence":0.9}',
  label: 'desktop-automation-100per100-local'
});
```

---

## Macro Recording

### GUI Recorder
Run the Tkinter GUI to record macros:
```bash
python scripts/record_macro.py
```
Records: mouse moves, clicks, scrolling, keyboard, window switches.
Saves JSON files to `recorded_macro/`.

### CLI Player
Replay a recorded macro:
```bash
python scripts/play_macro.py recorded_macro/macro_2026-03-14_22-00-00.json 1.0
```
Speed factor optional (1.0 = normal).

---

## 🔍 Monitor Screen (Conditional Automation)

The `monitor_screen` action watches the screen for specific visual elements (images or text) and executes actions when they appear.

### Parameters

```json
{
  "checks": [
    {
      "type": "image" | "text",
      "template_path": "path/to/template.png",
      "confidence": 0.9,
      "text": "string to find",
      "lang": "fra",
      "action": "click",
      "action_params": { "button": "left" }
    }
  ],
  "timeout": 60,
  "interval": 0.5,
  "stop_condition": "first_match",
  "fallback_confidence": 0.85
}
```

- `checks`: array of conditions to monitor
- `timeout`: max seconds to monitor (default 60)
- `interval`: check every N seconds (default 0.5)
- `stop_condition`:
  - `"first_match"`: stop after first condition matches (default)
  - `"all_matched"`: stop when all conditions have matched at least once
  - `null` or omitted: run until timeout
- `fallback_confidence`: if image not found at initial confidence, automatically retry with lower thresholds (0.85, 0.8, 0.75, ...). Set to `null` to disable.

### Example: Click a pop-up as soon as it appears

```javascript
sessions_spawn({
  task: 'monitor_screen',
  params: {
    checks: [
      {
        type: "image",
        template_path: "C:/templates/ok_button.png",
        confidence: 0.9,
        action: "click"
      }
    ],
    timeout: 30,
    stop_condition: "first_match"
  },
  label: 'desktop-automation-100per100-local'
});
```

### Example: Type credentials when fields appear

```javascript
sessions_spawn({
  task: 'monitor_screen',
  params: {
    checks: [
      {
        type: "text",
        text: "Email",
        lang: "eng",
        action: "type",
        action_params: { "text": "user@example.com", "interval": 0.05 }
      },
      {
        type: "text",
        text: "Password",
        lang: "eng",
        action: "type",
        action_params: { "text": "secret123", "interval": 0.05 }
      }
    ],
    timeout: 20,
    stop_condition: "all_matched"
  },
  label: 'desktop-automation-100per100-local'
});
```

---

## Configuration

### OCR (pytesseract)
- Install [Tesseract](https://github.com/tesseract-ocr/tesseract) on your OS.
- On Windows, add to PATH or set:
  ```python
  pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'
  ```
- Install language packs (`fra`, `eng`, etc.)

### Pyperclip
- Linux: install `xclip` or `xsel`
- Windows/macOS: usually works out of the box

---

## 📊 Data Integration

### extract_screen_data
Extract structured text data from the screen (or a specific region) using Tesseract OCR. Returns bounding boxes, text, and confidence scores.

```javascript
sessions_spawn({
  task: 'extract_screen_data',
  params: {
    region: {x: 0, y: 0, width: 800, height: 600},  // optional
    output_format: 'json'  // default
  },
  label: 'desktop-automation-100per100-local'
});
```

**Response**:
```json
{
  "status": "ok",
  "data": [
    {"text": "Total", "left": 100, "top": 200, "width": 50, "height": 15, "conf": 98.5},
    {"text": "$123.45", "left": 200, "top": 200, "width": 60, "height": 15, "conf": 99.1}
  ],
  "count": 2
}
```

### excel_read
Read an Excel file (.xlsx) and return rows as an array of objects (keys = column headers).

```javascript
sessions_spawn({
  task: 'excel_read',
  params: {
    filepath: "C:/data/input.xlsx",
    sheet_name: 0,      // or "Sheet1"
    range: "A1:C10"     // optional; reads whole sheet if omitted
  },
  label: 'desktop-automation-100per100-local'
});
```

**Response**:
```json
{
  "status": "ok",
  "data": [
    {"Name": "Alice", "Age": 30},
    {"Name": "Bob", "Age": 25}
  ],
  "columns": ["Name", "Age"],
  "rows": 2
}
```

### excel_write
Write an array of objects (or array of arrays) to an Excel file.

```javascript
sessions_spawn({
  task: 'excel_write',
  params: {
    filepath: "C:/data/output.xlsx",
    data: [
      {"Product": "Widget", "Price": 9.99},
      {"Product": "Gadget", "Price": 19.99}
    ],
    sheet_name: "Results",
    start_cell: "A1"
  },
  label: 'desktop-automation-100per100-local'
});
```

**Response**:
```json
{"status":"ok","filepath":"C:/data/output.xlsx","rows":2,"columns":2}
```

### data_to_csv
Convert a list of dictionaries to CSV format. Returns CSV string or writes to a file.

```javascript
sessions_spawn({
  task: 'data_to_csv',
  params: {
    data: [{"a":1,"b":2},{"a":3,"b":4}],
    filepath: "C:/data/out.csv"  // optional; if omitted, CSV string is returned
  },
  label: 'desktop-automation-100per100-local'
});
```

**Response** (without filepath):
```json
{
  "status": "ok",
  "csv": "a,b\n1,2\n3,4\n",
  "rows": 2,
  "columns": 2
}
```

---

## Safety & Best Practices

⚠️ **This skill controls mouse and keyboard. Use with extreme caution.**

### Safe Mode (enabled by default)
- Blocks risky actions (`type`, `press_key`, `click`, `drag`) to prevent accidental damage.
- Disable with: `sessions_spawn({ task: 'set_safe_mode', params: {enabled: false} })`
- Also blocks dangerous patterns (e.g., `rm `, `del `, system paths).

### Dry-Run (Sandbox)
Add `"dry_run": true` to any action to simulate without side effects. Great for testing.

### Audit Logging
All actions are logged to daily files under:
- **Windows**: `C:\Users\<User>\.openclaw\skills\desktop-automation-logs\`
- **Linux/macOS**: `~/.openclaw/skills/desktop-automation-logs/`

Use logs to debug or audit activity.

### General
- Verify coordinates before clicking.
- Test macros in dry-run or slow speed first.
- `activate_window` requires exact or partial title match.
- No network access — completely local.
- May require admin rights for certain applications on Windows.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `pyautogui.FailSafeException` | Move mouse to corner (0,0) to disable failsafe, or set `pyautogui.FAILSAFE = False` |
| OCR not working | Ensure Tesseract is installed and in PATH. Check language pack. |
| `find_image` fails | Template must match screen exactly (scale, colors). Lower confidence if needed (0.7–0.95). |
| `activate_window` can't find window | Use `list_windows` to get exact titles. |
| Missing modules | Run `pip install -r requirements.txt` |
| Tkinter GUI not opening | Linux: `sudo apt-get install python3-tk`. Windows: usually already installed. |

---

## License

MIT © 2026 Jordane Guemara & contributors

See `LICENSE` file.

---

## Contributing

See `CONTRIBUTING.md` to contribute to this skill.

---

## Author

Created by **Jordane Guemara** — [@JordaneParis](https://github.com/JordaneParis)
