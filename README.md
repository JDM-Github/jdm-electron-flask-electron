# JDM | Flask & Electron Template

A minimal template for shipping a **Flask web app as a native desktop application** using Electron. Electron handles the window and lifecycle; Flask handles everything else — routing, UI, and API.

---

## How It Works

```
Electron starts
  └── Finds a free port
  └── Spawns flask_server.exe with FLASK_PORT env var
  └── Shows loading screen while waiting for backend
  └── Loads http://127.0.0.1:{port} once backend is ready
```

Flask serves the full UI (HTML, JS, CSS) and any API routes. Electron is just the shell — it does not serve anything itself.

---

## Structure

```
electron/
--| build/
--| --| icon.ico
--| --| icon.png
--| resources/
--| --| backend/
--| --| --| flask_server.exe
--| loading.html
--| main.js
--| package.json
```

| Path | Purpose |
|---|---|
| `main.js` | Electron entry — spawns backend, manages window lifecycle |
| `loading.html` | Shown while Flask is booting |
| `resources/backend/flask_server.exe` | Compiled Flask server (your actual app) |
| `build/icon.*` | App icons for installer |

---

## Prerequisites

- [Node.js](https://nodejs.org/) (LTS)
- Your compiled `flask_server.exe` placed at `resources/backend/flask_server.exe`

---

## Development

Install dependencies:

```bash
npm install
```

Run in dev mode (expects `flask_server.exe` in `../backend/dist/`):

```bash
npm run dev
```

The dev path for the backend exe is:

```js
// main.js — dev path
path.join(__dirname, '../backend/dist', 'flask_server.exe')
```

Adjust this to wherever your PyInstaller output lands.

---

## Building the Flask Server

Before packaging Electron, compile your Flask app with PyInstaller:

```bash
pyinstaller --onefile app.py --name flask_server
```

Copy the output to `resources/backend/flask_server.exe`.

---

## Building the Desktop App

**Windows** — produces an NSIS installer in `dist/`:
```bash
npm run dist
```

**macOS** — produces a `.dmg`:
```bash
npm run dist:mac
```

**Linux** — produces an `.AppImage`:
```bash
npm run dist:linux
```

Built artifacts go to the `dist/` folder.

---

## Port Handling

Electron finds a free port at runtime using a temporary TCP server, then passes it to Flask via the `FLASK_PORT` environment variable. There is no hardcoded port — this avoids conflicts on any machine.

Your Flask app must read this variable:

```python
import os

port = int(os.environ.get("FLASK_PORT", 5000))

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=port)
```

---

## Crash Handling

If the Flask process exits unexpectedly while the app is running, Electron shows an error dialog and quits cleanly. On Windows, the entire process tree is killed via `taskkill` to avoid orphaned processes.

---

## Customization

| What | Where |
|---|---|
| App name / version | `package.json` → `productName`, `version` |
| Window size | `main.js` → `BrowserWindow` options |
| Loading screen | `loading.html` |
| App icon | `build/icon.ico` (Windows), `build/icon.png` (macOS/Linux) |
| Backend startup timeout | `main.js` → `waitForPort(port, timeout)` |

---

## License

MIT