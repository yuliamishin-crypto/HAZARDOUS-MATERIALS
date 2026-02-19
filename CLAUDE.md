# CLAUDE.md — HAZARDOUS-MATERIALS Repository

## Project Overview

This is a **static web application** that serves as a searchable database of hazardous chemical materials. The UI and data are primarily in Hebrew (RTL layout), with chemical property fields in both Hebrew and English.

The application has no build system, no package manager, and no server-side code. It runs entirely in the browser.

---

## Repository Structure

```
HAZARDOUS-MATERIALS/
├── CLAUDE.md       # This file
├── README.md       # Minimal project title only
├── HAZAR.csv       # Primary data file (~200 chemicals, 38 columns, UTF-8 BOM)
└── index.html      # Single-page web viewer for the CSV data
```

---

## Files

### `HAZAR.csv`

- **Encoding**: UTF-8 with BOM (`\uFEFF` prefix on first column header)
- **Rows**: ~200 hazardous chemical substances
- **Columns**: 38 per row (see full list below)
- **Language**: Column headers and most values are in Hebrew; some headers and values are in English

**Column Schema** (in order):

| # | Column | Description |
|---|--------|-------------|
| 1 | שם החומר (עברית) | Chemical name in Hebrew |
| 2 | Chemical Name (English) | Chemical name in English |
| 3 | נוסחה כימית | Chemical formula |
| 4 | מספר CAS | CAS registry number |
| 5 | נקודת הבזק (°C) | Flash point (°C) |
| 6 | נקודת רתיחה (°C) | Boiling point (°C) |
| 7 | LEL (%) | Lower Explosive Limit (%) |
| 8 | UEL (%) | Upper Explosive Limit (%) |
| 9 | טמפרטורת התלקחות עצמית (°C) | Auto-ignition temperature (°C) |
| 10 | לחץ אדים (mmHg @ 20°C) | Vapor pressure (mmHg at 20°C) |
| 11 | צפיפות יחסית (מים=1) | Relative density (water=1) |
| 12 | צפיפות אדים (אוויר=1) | Vapor density (air=1) |
| 13 | מסיסות במים | Water solubility |
| 14 | pH (אם רלוונטי) | pH (if relevant) |
| 15 | קבוצת NFPA 400 ראשית | NFPA 400 primary group |
| 16 | תת-קבוצה NFPA 400 | NFPA 400 subgroup |
| 17 | קבוצות NFPA 400 נוספות | Additional NFPA 400 groups |
| 18 | סיווג NFPA 30 | NFPA 30 classification (Hebrew) |
| 19 | Class NFPA 30 | NFPA 30 classification (English) |
| 20 | LC50 (ppm) | Lethal concentration 50 (ppm) |
| 21 | LD50 (mg/kg) | Lethal dose 50 (mg/kg) |
| 22 | רמת סיכון בריאות NFPA | NFPA health hazard level (0–4) |
| 23 | רמת דליקות NFPA | NFPA flammability level (0–4) |
| 24 | רמת תגובתיות NFPA | NFPA reactivity level (0–4) |
| 25 | סיכונים מיוחדים NFPA | NFPA special hazards (e.g. OX, W, COR, CRYO) |
| 26 | UN Number | UN transport number |
| 27 | Class DOT | DOT hazard class |
| 28 | Packing Group | DOT packing group (I, II, III) |
| 29 | הערות | General notes |
| 30 | MAQ אחסון - מערכת סגורה (ק"ג/ליטר) | MAQ storage, closed system (kg/L) |
| 31 | MAQ אחסון - מערכת פתוחה (ק"ג/ליטר) | MAQ storage, open system (kg/L) |
| 32 | MAQ שימוש - מערכת סגורה (ק"ג/ליטר) | MAQ use, closed system (kg/L) |
| 33 | MAQ שימוש - מערכת פתוחה (ק"ג/ליטר) | MAQ use, open system (kg/L) |
| 34 | MAQ גז - אחסון (מ"ק) | MAQ gas storage (m³) |
| 35 | MAQ גז - שימוש (מ"ק) | MAQ gas use (m³) |
| 36 | Protection Level | Protection level classification |
| 37 | הערות MAQ | MAQ notes |
| 38 | יחידות MAQ | MAQ units (ליטר / מ"ק) |

**MAQ** = Maximum Allowable Quantity (per NFPA 400 / Israeli fire code standards).

### `index.html`

A single-page application in Hebrew that:

- Loads `data.csv` (see **Known Issue** below) via the [PapaParse](https://www.papaparse.com/) library (v5.4.1, loaded from CDN)
- Renders all rows as an HTML `<table>`
- Provides a real-time keyword search input (`<input id="search">`) that filters rows by `innerText`
- Uses RTL page direction (`direction: rtl`)

**Key implementation details:**

- PapaParse is loaded from `https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js`
- The table header is built from `Object.keys(data[0])`
- Row filtering uses case-insensitive `includes()` on the full row text
- There is **no XSS sanitization** on cell values — values are inserted directly via string concatenation into `innerHTML`

---

## Known Issue: CSV Filename Mismatch

`index.html` references `"data.csv"` at line 24:

```js
Papa.parse("data.csv", { ... });
```

The actual data file is named `HAZAR.csv`. The application will fail to load data unless:
- The file is renamed to `data.csv`, or
- The reference in `index.html` is updated to `"HAZAR.csv"`, or
- The file is served via a web server that aliases the path

**Fix**: Change line 24 of `index.html` from `"data.csv"` to `"HAZAR.csv"`.

---

## Development Workflow

### Running Locally

Since this is a static site, open it with any local HTTP server (direct `file://` access will block the CSV fetch due to CORS restrictions):

```bash
# Python
python3 -m http.server 8000

# Node.js (npx)
npx serve .

# Then open: http://localhost:8000
```

### Editing the Data

Edit `HAZAR.csv` directly. Keep the UTF-8 BOM encoding and the comma-delimited format. When adding rows:

- Follow the existing column order exactly (38 columns)
- Use Hebrew for the first column (chemical name in Hebrew)
- Use `לא רלוונטי` for non-applicable numeric fields, not empty
- Use `לא דליק` / `לא מסיס` / `מסיס` for non-numeric descriptive fields
- NFPA levels are integers 0–4
- MAQ units should match the existing pattern: `ליטר` for liquids, `מ"ק` for gases

### Editing the Viewer

`index.html` is a self-contained file. Edit it directly. There is no transpilation, bundling, or compilation step.

---

## Conventions

- **Language**: Hebrew is primary for UI labels and data values; English for chemical names, CAS numbers, and regulatory codes
- **Encoding**: UTF-8 with BOM for CSV (preserve on save)
- **No build system**: Do not add npm, webpack, Vite, or similar tools unless explicitly requested
- **No frameworks**: Plain HTML/CSS/JS only; no React, Vue, etc.
- **CDN dependencies**: PapaParse is loaded from Cloudflare CDN — no local copy is maintained
- **Data integrity**: Do not reorder or rename CSV columns; the HTML viewer relies on column order for display

---

## Regulatory Standards Referenced

- **NFPA 400**: Hazardous Materials Code (groups, MAQ values, protection levels)
- **NFPA 30**: Flammable and Combustible Liquids Code (Class IA/IB/IC/II/IIIA)
- **DOT**: US Department of Transportation hazmat classification (UN numbers, packing groups)
- **Israeli fire regulations**: MAQ thresholds appear to align with Israeli fire safety authority requirements

---

## Branch Information

- `master` / `main`: Primary branches
- `claude/...`: AI assistant working branches
