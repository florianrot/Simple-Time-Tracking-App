# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

Pure HTML/CSS/JS (ES6+), no framework, no build process. Hosted via GitHub Pages from `main` branch root.

- `xlsx-js-style` via CDN for Excel export
- Google Apps Script as sync backend
- Service Worker for offline PWA

## Architecture

### Data Flow

```
UI (index.html) → app.js entriesCache (in-memory)
                     ↕ localStorage (zt_entries)
                     ↕ Google Sheets (via Apps Script POST/GET)
```

All state lives in `entriesCache` (array of entry objects). Every mutation (create/edit/delete) immediately writes to localStorage, then fires `syncWrite()` to Google Sheets. On app start, `loadEntries()` reads localStorage, then `syncRead()` merges remote entries by ID (local wins on conflict).

### Key Modules (all in app.js)

- **Sync** (~line 108): `syncRead()` merges remote by ID, `syncWrite()` with dirty-flag retry
- **TimePicker** (~line 307): iOS-style wheel picker, 15-min steps, smart defaults (Von = now-4h, Bis = now)
- **DatePicker** (~line 448): 3-wheel picker (day/month/year), dynamic day count
- **Modals**: Toggle via `.hidden` class, backdrop close via `[data-close]` attribute
- **Excel Export** (~line 675): Uses `excelStyles.js` for formatting config, generates .xlsx with formulas
- **View Navigation**: 3 tabs (Erfassen/Dashboard/Eintraege), CSS `.view.active` toggle

### Google Apps Script (apps_script.gs)

- `doGet`: Reads all monthly sheets, returns entries as JSON
- `doPost`: Clears monthly sheets, rewrites from payload with formulas for totals/wage
- Sheets named by German month: "Januar 2026", "Februar 2026", etc.
- Must be deployed as Web App (Execute as "Me", Access "Anyone")

## Development

### Deploying changes

1. Edit files
2. Bump cache version in `sw.js` (`CACHE_NAME`) AND `index.html` (query strings on script tags) — these must match
3. Commit and push to `main` — GitHub Pages auto-deploys

### Testing sync locally

Open browser console and check for:
- `Sync read error` / `Sync write error` — network/CORS issues
- Toast "Sync fehlgeschlagen" — write failed, dirty flag set for retry
- Verify localStorage: `JSON.parse(localStorage.getItem('zt_entries'))`

### Date handling

Always use `parseDate(str)` (not `new Date(dateStr)`) for entry dates. `new Date("YYYY-MM-DD")` parses as UTC which causes timezone shift bugs with `getMonth()`/`getFullYear()`.

## Conventions

- German UI text throughout (labels, toasts, month names)
- Currency: CHF, locale: de-CH
- Time format: 24h HH:mm, 15-minute increments
- Dark mode only (#0f0f0f background)
- No emojis in code or UI
