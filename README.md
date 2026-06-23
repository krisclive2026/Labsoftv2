# LabSoft — Build EXE (Virtual Environment)

Steps to package `app.py` into a standalone Windows EXE using a clean virtual
environment, so the build only picks up exactly what LabSoft needs.

## 1. Project folder layout

Make sure these are all in the same folder before you start:

```
LabSoft/
├── app.py
├── requirements.txt
├── lab.ico
├── templates/
│   └── index.html
└── static/
    ├── CSS/
    └── js/
```

(`requirements.txt` is included alongside this README — drop it into the
project folder if it isn't there already.)

## 2. Create the virtual environment

Open Command Prompt / PowerShell in the project folder:

```bash
python -m venv venv
```

## 3. Activate it

```bash
venv\Scripts\activate
```

Your prompt should now show `(venv)` at the start of the line. Every command
below must be run with the venv active.

## 4. Install dependencies

```bash
pip install -r requirements.txt
```

This installs Flask, flask-cors, cryptography, openpyxl, pdfplumber,
reportlab, xhtml2pdf, and PyInstaller — nothing else, keeping the build lean.

## 5. Build the EXE

```bash
pyinstaller --onefile --windowed --name Labsoft --icon=lab.ico --add-data "templates;templates" --add-data "static;static" --collect-all reportlab --collect-all xhtml2pdf --collect-all pdfplumber app.py
```

Flag reference:
| Flag | Why |
|---|---|
| `--onefile` | bundles everything into a single `Labsoft.exe` |
| `--windowed` | hides the console window (background Flask server) |
| `--icon=lab.ico` | sets the EXE icon — must be a real `.ico` file in this folder |
| `--add-data "templates;templates"` | bundles `templates/` (Flask reads it from `sys._MEIPASS` when frozen) |
| `--add-data "static;static"` | bundles `static/` (CSS/JS) the same way |
| `--collect-all reportlab/xhtml2pdf/pdfplumber` | these pull in fonts/resources & dynamic submodules PyInstaller misses by default — skipping this causes blank PDFs or `ModuleNotFoundError` at runtime |

## 6. Get the build

The finished EXE is at:

```
dist\Labsoft.exe
```

`build\` and `Labsoft.spec` are intermediate files — safe to delete, or keep
the `.spec` if you want to tweak and rebuild faster next time
(`pyinstaller Labsoft.spec`).

## 7. First run

On first launch, LabSoft creates its working data folder at:

```
%APPDATA%\.labsoft\
```

This holds `lab_categories_fixed.db`, `patients.db`, and `doctors.db` —
separate from the EXE, so the databases survive when you replace
`Labsoft.exe` with a newer build.

## Troubleshooting

- **App seems to do nothing on launch** — `--windowed` hides errors too.
  Temporarily rebuild without `--windowed` (drop it from the command) to see
  the console and any traceback, then re-add it once it's working.
- **Blank or broken PDF/Excel exports** — usually means one of the
  `--collect-all` flags got dropped. Re-check the build command.
- **Antivirus flags the EXE** — common false positive with PyInstaller
  one-file builds; whitelist the `dist` folder while testing.
- **Port 5000 already in use** — close any other running copy of LabSoft
  (check Task Manager for a lingering `Labsoft.exe` process) before
  relaunching.

## Rebuilding later

You can reuse the same `venv` for future builds — just re-activate it
(step 3) and re-run the `pyinstaller` command (step 5). Only redo steps 2–4
if you start from a fresh machine or delete the `venv` folder.
