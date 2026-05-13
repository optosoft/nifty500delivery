# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start

**Running the application:**
- Open `index.html` directly in a web browser (no build process required)
- The app is self-contained with all dependencies loaded from CDNs

**Developing:**
- Edit `index.html` directly and refresh the browser to see changes
- No npm, no build tools, no compilation step
- Console (F12 developer tools) shows any errors

## Application Overview

**Nifty 500 Alpha Analytics** is a client-side React dashboard for analyzing stock delivery patterns and identifying trading opportunities in the NSE Nifty 500 index.

The application has no backend—all processing happens in the browser. Users provide:
1. Daily stock data files (CSV/XLSX with columns like NSE Code, Stock Name, Delivery %, Price, SMA200, SMA50, etc.)
2. A `sector.xlsx` mapping file that associates stock codes with sector/industry classifications

The app then visualizes patterns across multiple strategies and time frames.

## Architecture

### Core Data Model
The application manages two datasets:
- **rawData**: Array of stock records, each containing: `{date, nseCode, stockName, deliveryEod, deliveryAvgWeek, deliveryAvgMonth, deliveryAvg6Month, price, sma200, sma50, priceChange, marketCap, pe, indPe}`
- **sectorIndustryMap**: Object mapping NSE codes to `{sector, industry}` pairs

### Component Structure
The entire UI is a single `<App>` component with these major functional areas:

1. **Data Loading** (lines 99-163)
   - `processFiles()`: Parses XLSX/CSV files using SheetJS
   - Special handling for `sector.xlsx` (mapping file)
   - Date extraction from filename patterns (YYYY-MM-DD format)
   - Fuzzy key matching for CSV headers (handles column name variations)

2. **Filtering & Computation** (lines 165-219)
   - `filteredData`: Memoized computed state applying all active filters
   - Time-frame modes: `all`, `date`, `month`, `quarter`, `year`, `custom`
   - Strategy scoring: Each stock gets `mom` (momentum), `rev` (reversion), `vol` (volume), `comp` (composite)
   - Base filter: `deliveryEod > deliveryAvg6Month && deliveryAvgWeek >= minDeliveryPct && deliveryAvgMonth >= minDeliveryPct`

3. **Visualization Layers** (lines 225-275)
   - `drillChartData`: Hierarchical drill-down (Sectors → Industries → Stocks)
   - `globalFreqData`: Top 30 stocks by frequency across dataset
   - `breadthInsights`: Sector health (% of stocks above SMA 200)
   - `scatterData`: Delivery % vs. price change (bubble size = market cap)
   - `valuationData`: Stock PE vs. Industry PE comparison
   - `sortedTable`: Sortable stock table with all metrics

4. **Tab Navigation** (lines 345-529)
   - **Visual Dashboard**: Drill-down charts and frequency analysis
   - **Advanced Analytics**: Breadth, valuation, scatter plots
   - **Stock Explorer**: Searchable, sortable data table

### Strategy Scoring Logic (lines 165-182)
Each strategy has specific thresholds:
- **Momentum** (mom): Trend-following (price > SMA, positive change)
- **Reversion** (rev): Mean reversion (price < SMA despite high delivery)
- **Volume** (vol): Unusual delivery concentration
- **Composite** (comp): Weighted blend (40% momentum + 40% reversion + 20% volume)

Strategies are applied as additional filters once base delivery criteria are met.

## Input File Formats

### Daily Data Files (CSV/XLSX)
Required columns (fuzzy-matched, case-insensitive):
- `NSE Code` / `Symbol` / `Ticker` / `Code` — stock identifier
- `Stock Name` / `Name` / `Company Name` — company name
- `Delivery% Volume End of Day` / `Delivery %` / `Delivery Percentage` — day's delivery %
- `Delivery% Volume Avg Week` / `Avg Week Delivery` — weekly average
- `Delivery% Volume Avg Month` / `Avg Month Delivery` — monthly average
- `Delivery% Volume Avg 6Month` / `Avg 6 Month Delivery` — 6-month average
- `Current Price` / `Price` / `LTP` / `Close` — stock price
- `Day SMA200` / `SMA 200` / `SMA200` — 200-day moving average
- `Day SMA50` / `SMA 50` / `SMA50` — 50-day moving average
- `Day Change %` / `Change %` / `Chg %` — price change percentage
- `Market Capitalization` / `Market Cap` / `MCAP` — market cap in Cr.
- `PE TTM Price to Earnings` / `PE` / `P/E` — stock P/E ratio
- `Industry PE TTM` / `Industry PE` / `Ind PE` — industry average P/E

Filename date format: extracts from patterns like `2026-05-08`, `2026/05/08`, etc.

### Sector Mapping File (`sector.xlsx`)
Required columns:
- `NSE Code` / `Symbol` / `Ticker` / `Code` / `NSE_Code` — stock code
- `Sector` / `Sector_Name` / `Sector Name` — sector name
- `Industry` / `Industry Name` / `Industry_Name` — industry classification

Codes are normalized to uppercase for matching.

## Key Technical Decisions

**Single HTML file, no build tools**: The entire app fits in one file (~550 lines) because:
- React is loaded from CDN (development build)
- Babel standalone transpiles JSX in-browser
- Recharts and Tailwind are CDN-hosted
- No complex module dependencies

**Client-side processing only**: 
- No backend API calls
- Users select data folder directly (browser File API)
- All filtering and visualization computed in React state
- Leverages `useMemo` to avoid unnecessary recalculations

**Fuzzy header matching**:
- CSV/XLSX files from different sources have inconsistent column names
- Headers are trimmed, lowercased, and matched against expected variations
- Gracefully defaults to 0 if a field is missing

**Memoization strategy**:
- `filteredData`, `drillChartData`, `globalFreqData`, etc. are memoized
- Recalculate only when dependencies change
- Critical for performance with large datasets

## Development Notes

### Common Modifications

**Adding a new metric field**:
1. Add to `mapRowToStock()` extraction logic
2. Add to the data object returned (line 82-96)
3. Reference in filters or visualizations as needed

**Adding a new strategy**:
1. Update `calculateScores()` with new scoring logic
2. Add filtering condition in `filteredData` memoization
3. Add strategy button in the UI grid (lines 322-339)

**Adding a new visualization**:
1. Derive a memoized dataset from `filteredData`
2. Add a Recharts component in the relevant tab
3. Wire it to active state with `activeTab`

**Modifying filter logic**:
- Core filter logic: lines 184-219 in the `filteredData` memoization
- Time-frame logic: lines 189-196
- Strategy thresholds: lines 202-205

### Testing Changes
- Open DevTools (F12) to inspect React state
- Use browser's localStorage inspection if session state needs persistence
- The app degrades gracefully if files are missing columns

### Performance Considerations
- Large datasets (100K+ records) may lag on column filters
- Memoization deps are critical—missing a dep causes stale renders
- Recharts re-renders on data changes; consider if drill-down should be paginated

## UI Component Reference

**Icon components** (lines 40-46): SVG icons for navigation (ChevronRight, Back, Refresh, Folder, Sort)

**Strategy buttons** (lines 322-339): Four toggle buttons for strategy filtering

**Time-frame controls** (lines 301-318): Mode selector with conditional inputs (date picker, month/quarter/year dropdowns, date range)

**Tabs** (lines 346-352): Visual Dashboard, Advanced Analytics, Stock Explorer

## Common Debugging

**No data appearing after file selection**: 
- Check DevTools console for errors in `processFiles()`
- Verify CSV/XLSX column headers match expected patterns
- Ensure `sector.xlsx` exists and has matching stock codes

**Sector showing "Pending Mapping"**:
- Indicates `sector.xlsx` hasn't been loaded yet
- Or stock code in data file doesn't match mapping file codes

**Chart not updating after filter change**:
- Check if dependency is missing from memoization deps array
- Verify the filter UI is calling the correct state setter

**Performance lag on large datasets**:
- Use browser DevTools Performance tab to profile
- Check if memoization is working (avoid recalculating same data)
- Consider limiting visible rows in table
