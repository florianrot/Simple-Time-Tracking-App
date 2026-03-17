# Zeiterfassung Zimmerei

Mobile-First PWA zur Zeiterfassung, entwickelt als Ersatz fuer manuelle Google Sheets Eingabe. Optimiert fuer den schnellen Einsatz auf der Baustelle — App oeffnen, Zeit eintragen, fertig.

## Features

### Zeit erfassen

- **Automatisches Datum** — Beim Oeffnen ist das heutige Datum vorausgefuellt. Aenderbar ueber den Wheel-Picker.
- **Wheel-Picker fuer Uhrzeit** — Scrollbare Raeder (wie iOS), kein Tippen noetig. Minuten in 15-Minuten-Schritten (00, 15, 30, 45).
- **Smart Defaults** — "Von"-Feld startet automatisch bei aktuelle Uhrzeit minus 4 Stunden (typischer Arbeitsbeginn). "Bis"-Feld startet bei der aktuellen Uhrzeit. Beide auf 15 Minuten gerundet.
- **Live-Berechnung** — Sobald "Von" und "Bis" gesetzt sind, wird die Stundenanzahl sofort angezeigt.
- **Mitternachts-Erkennung** — Arbeitszeiten ueber Mitternacht (z.B. 22:00 - 02:00) werden korrekt als 4 Stunden berechnet.

### Eintraege verwalten

- **Monatsansicht** — Alle Eintraege gruppiert nach Monat, navigierbar mit Pfeiltasten.
- **Eintrag bearbeiten** — Antippen oeffnet den Bearbeitungsmodus mit Datum, Von, Bis und Live-Berechnung.
- **Eintrag loeschen** — Direkt im Bearbeitungsmodus oder ueber Multi-Select.
- **Multi-Select** — Checkbox-Icon in der Kopfzeile aktiviert den Mehrfachauswahl-Modus. Eintraege anwaehlen und gesammelt loeschen.
- **Wochentag-Anzeige** — Jeder Eintrag zeigt den Wochentag (MO, DI, MI, ...) vor dem Datum.

### Dashboard

- **Monats-Stunden** — Gesamtstunden des aktuell angezeigten Monats.
- **Lohn netto** — Berechnet aus Stunden x Stundenlohn (CHF). Der Stundenlohn ist in den Einstellungen konfigurierbar (Standard: CHF 38.00).
- **Direktlinks** — "Eintraege anzeigen" und "Excel exportieren" direkt vom Dashboard aus.

### Excel-Export

- **Monatsweise** — Beim Export waehlt man den gewuenschten Monat aus einem Dropdown.
- **Fertig formatiert** — Die `.xlsx`-Datei enthaelt:
  - Firmen-Header ("Zimmerei") + Monatsname
  - Tabelle mit Datum, Von, Bis, Stunden
  - Laufende Total-Spalte (kumulierte Stunden)
  - Lohn-Spalte (kumulierter Netto-Lohn in CHF)
  - Total-Zeile am Ende mit Gesamtsummen
- **Sauberes Layout** — Kein Gitternetz, gesetzte Spaltenbreiten, passende Zahlenformate.

### Google Sheets Sync

- **Automatisch** — Jeder neue Eintrag, jede Bearbeitung und jedes Loeschen wird automatisch nach Google Sheets synchronisiert.
- **Merge-Logik** — Beim App-Start werden lokale und Remote-Daten per ID zusammengefuehrt. Lokale Eintraege gehen nie verloren.
- **Monatliche Blaetter** — Google Sheets erstellt automatisch ein Tabellenblatt pro Monat ("Januar 2026", "Februar 2026", ...) mit Formeln fuer Totals und Lohn.
- **Fehlermeldung** — Bei fehlgeschlagenem Sync erscheint ein Toast "Sync fehlgeschlagen". Ungesyncte Daten werden beim naechsten App-Start nachgeholt.

### Einstellungen

- **Stundenlohn** — Frei konfigurierbar (CHF, Standard: 38.00). Wird fuer Dashboard-Berechnung, Excel-Export und Google Sheets verwendet.
- **Google Script URL** — Die Deploy-URL des Google Apps Scripts. Ohne URL funktioniert die App rein lokal (localStorage).

### PWA / Offline

- **Installierbar** — Kann auf iPhone und Android als App auf dem Homescreen installiert werden. Laeuft dann im Standalone-Modus ohne Browser-Leisten.
- **Offline-faehig** — Service Worker cached alle App-Dateien. Bei schlechtem Netz greift nach 3 Sekunden automatisch der Cache.
- **Dark Mode** — Durchgehend dunkles Design (#0f0f0f Hintergrund).

## Tech Stack

- HTML5, CSS3, JavaScript (ES6+) — kein Framework, kein Build-Prozess
- [xlsx-js-style](https://github.com/gitbrent/xlsx-js-style) fuer Excel-Export (via CDN)
- Google Apps Script als Sync-Backend
- Service Worker fuer Offline-Cache
- Gehostet via GitHub Pages

## Setup

### 1. GitHub Pages aktivieren

Repository Settings > Pages > Source: `main` Branch, Root `/`

### 2. Google Sheets Backend einrichten

1. Neues Google Sheet erstellen
2. Extensions > Apps Script oeffnen
3. Gesamten Inhalt von `apps_script.gs` dort einfuegen
4. Funktion `setup` einmal ausfuehren (erteilt die noetige Berechtigung)
5. Deploy > New Deployment > Type: Web App
   - Execute as: "Me"
   - Who has access: "Anyone"
6. Deploy-URL kopieren

### 3. App konfigurieren

1. App oeffnen (GitHub Pages URL oder lokal)
2. Tab "Eintraege" > Zahnrad-Icon oben rechts
3. Google Script URL einfuegen
4. Stundenlohn anpassen
5. Speichern

Ab jetzt synchronisiert die App automatisch mit dem Google Sheet.

## Dateistruktur

```
.
├── index.html          App-Struktur, alle Modals und Picker
├── style.css           Komplettes Styling, Dark Mode, Responsive
├── app.js              Gesamte App-Logik: Eintraege, Sync, Picker, Export
├── excelStyles.js      Excel-Export Formatierung und Spalten-Definitionen
├── sw.js               Service Worker mit Offline-Cache und Timeout
├── apps_script.gs      Google Apps Script Backend (in Sheets einfuegen)
├── manifest.json       PWA-Manifest
├── icon.png            App-Icon (192x192)
└── LICENSE             MIT
```

## Datenstruktur

Jeder Zeiteintrag:

```json
{
  "id": "m1abc2def3",
  "date": "2026-03-15",
  "from": "07:00",
  "to": "16:30",
  "hours": 9.5
}
```

- `id` — Generiert aus Timestamp + Zufallsstring, eindeutig pro Eintrag
- `date` — ISO-Format (YYYY-MM-DD)
- `from` / `to` — 24h-Format (HH:mm), in 15-Minuten-Schritten
- `hours` — Berechnete Differenz als Dezimalzahl

Gespeichert in `localStorage` unter dem Key `zt_entries` und synchronisiert mit Google Sheets.

## Lizenz

MIT
