# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file browser-based analytics dashboard (`index.html`) for analyzing NSE (National Stock Exchange) stock delivery percentage data. No build process, no package manager, no server required.

## Running the App

Open `index.html` directly in a browser. No build or install steps needed.

For local development with live reload, any static file server works:
```bash
python3 -m http.server 8080
# or
npx serve .
```

## Architecture

The entire application lives in `index.html` (~455 lines). It uses React 18 via CDN with Babel Standalone for in-browser JSX compilation — there is no transpile step.

**Data flow:**
1. User picks a local folder via the File System Access API (falls back to `<input type="file">`)
2. SheetJS parses Excel/CSV files from the folder into JSON arrays
3. `sector.xlsx` provides NSE code → Sector/Industry mapping
4. Multiple `useMemo` hooks filter, aggregate, and sort data for display
5. Two views: a Recharts bar chart with drill-down (Sector → Industry → Stocks) and a sortable table

**Key processing functions inside the single React component:**
- `processFiles` — reads folder contents and parses files with SheetJS
- `mapRowToStock` — normalizes raw row objects to a consistent stock schema
- `processedStocks` — joins stock data with sector mapping
- `filteredStocks` — applies delivery % thresholds and timeframe filters
- `chartData` — builds hierarchical drill-down counts for the bar chart

## Input Data Format

The user selects a folder containing:
- `sector.xlsx` — required mapping file with columns: `NSE code`/`Symbol`, `Sector`, `Industry`
- One or more date-named files (`YYYY-MM-DD.xlsx` or `.csv`) with columns:
  - `NSE code` / `Symbol` / `NSE Code`
  - `Stock Name`
  - `Delivery% volume end of day`
  - `Delivery% volume Avg Week`
  - `Delivery% volume Avg Month`
  - `Delivery% volume Avg 6Month`

## CDN Dependencies (no npm install)

All loaded at runtime from CDN:
- React 18 + ReactDOM 18 (production UMD)
- Babel Standalone (JSX in-browser)
- SheetJS v0.20.1 (XLSX)
- Recharts 2.12.7
- Tailwind CSS
- Lucide icons
- PropTypes 15.8.1
- Google Fonts (Inter)

## Development Notes

- All component logic is in one `<script type="text/babel">` block inside `index.html`
- Chart orientation switches automatically: horizontal for ≤8 bars, vertical for more
- The drill-down state is tracked via `drilldownPath` (array of strings)
- Custom date range filtering uses `customStartDate` / `customEndDate` state
