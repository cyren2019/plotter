# AGENTS.md

## Repo Overview

Minimal static data-visualization app. Single HTML file (~1300 lines) with embedded CSS and JavaScript.

- **Language**: Vanilla HTML/CSS/JS — no framework, no build step, no package manager
- **UI language**: Chinese (zh-CN)
- **Entry point**: `plotter-app/index.html`

## Project Structure

```
.
├── plotter-app/            <- The actual app
│   ├── index.html          <- All markup, styles, and logic (1289 lines)
│   ├── lib/                <- Vendored minified libraries (do NOT npm install)
│   │   ├── dayjs.min.js + customParseFormat.js
│   │   ├── xlsx.full.min.js
│   │   ├── echarts.min.js
│   │   └── echarts-gl.min.js   (3D scatter, scatter3D series)
│   ├── public/             <- Static assets (favicon.svg, icons.svg)
│   ├── test-data.csv       <- 11-line sample: time+temperature+pressure+humidity+voltage
│   └── .gitignore          <- Vite-style ignore list (Vite was never used here)
├── separate-*.png          <- Screenshots (separate mode layout, datazoom, alignment) — reference only, not code
├── .sisyphus/              <- OpenCode session artifacts
└── .playwright-mcp/        <- Playwright MCP tooling
```

**Git note**: `.git/` exists in both the repo root and inside `plotter-app/`. They share the same history. Run git commands from either directory.

## How to Run

Serve `plotter-app/` as a static directory:

```bash
npx serve plotter-app
# or
python -m http.server -d plotter-app 8080
```

No build step, no dev server, no hot reload. After any code change, hard-refresh the browser.

## Architecture

- **Single-file app**: HTML, CSS (`<style>`), and JS (`<script>`) all in `index.html`
- **Libraries**: Loaded via `<script src="lib/...">` — never attempt `npm install`
- **State**: Plain global variables (`data`, `selectedVars`, `plotType`, `timeSeriesMode`, `chartInstance`). No state management library.
- **Rendering**: Imperative DOM via `renderApp()` → clears `mainArea`, rebuilds sidebar + plot section from scratch. Chart re-inited on every render via `requestAnimationFrame(() => renderChart())`.
- **Data flow**:
  1. File dropped/selected (CSV or Excel)
  2. `parseFile()` → `parseCSV()` or `parseExcel()`
  3. Time column auto-detected via `detectTimeColumn()` using keyword matching + parseability scoring
  4. First 3 non-time variables auto-selected
  5. `renderApp()` rebuilds UI, `renderChart()` inits ECharts

## Conventions & Quirks

### Time Parsing
Custom and extensive. Two-pass: `dayjs` strict mode first, then lenient fallback. Handles Excel serial numbers (≥1 → `excelSerialToDate`), 17 format strings, Chinese date strings (`YYYY年M月D日`, etc.). Time keywords for auto-detection: `time`, `date`, `datetime`, `timestamp`, `时间`, `日期`, `時刻`, `日付`.

### Variable Selection Limits
Enforced in `getMaxSelectable()`:
- **Time-series**: max 10
- **XY scatter**: max 2 (chart renders only when exactly 2 selected)
- **XYZ 3D**: max 3 (chart renders only when exactly 3 selected)

Exceeding limit shows a toast message (`varLimitToast`) and blocks selection.

### Plot Types & Modes

| Plot Type | Tab Label | ECharts Series | Requires |
|---|---|---|---|
| `time-series` | 时序图 | `line` | 1-10 vars |
| `xy` | XY图 | `scatter` | exactly 2 vars |
| `xyz` | XYZ图 | `scatter3D` | exactly 3 vars |

**Timeseries sub-modes** (`timeSeriesMode`):
- **`merged`**: All series in one chart, shared Y-axis. Simple grid layout.
- **`separate`**: One subplot per variable, independent Y-axes. Complex multi-grid layout with shared bottom time axis. Hardcoded px values: `rowHeight=200`, `gap=20`, `gridLeft=60`, `gridRight=30`. Container height is pre-computed before `echarts.init`. Changing these constants requires recalculating `totalHeight` in `renderChart()` AND matching the separate-mode height pre-set above it.
- **`separateAutoScale`** (default `true`): Toggle button "自适应" visible only in separate mode. When ON → `scale: true` on yAxis, Y-range auto-adapts to visible data after zoom/pan. When OFF → fixed `min`/`max` calculated from full dataset with 10% padding. Toggling calls `renderChart()` only (no DOM rebuild).

### Rendering Pattern
```js
// Every renderChart() call:
if (chartInstance) chartInstance.dispose();  // Dispose old
chartInstance = echarts.init(container);      // Create new
chartInstance.setOption(option, true);        // Set options
requestAnimationFrame(() => chartInstance.resize());
```

The global `window.resize` listener also calls `chartInstance.resize()`. Be careful not to register duplicate listeners when modifying chart code.

### Theme Toggle
Sets `data-theme="dark"` on `<html>`. All colors are CSS custom properties (`--text`, `--border`, `--bg`, `--accent`, etc.). Chart options reference `isDark` global for inline color values. After theme toggle, `renderChart()` is called to rebuild with correct colors.

### Search Preserved Across Re-renders
`renderApp()` saves the search input value and scroll position before rebuilding the DOM, then restores them after. When adding UI elements, preserve this pattern.

### CSV Parser
Custom (not via SheetJS). Handles quoted fields, detects separator (`,` vs `;`). Excel files use SheetJS (`XLSX.read`). Both return `{ headers, rows, timeColumn, timeFormat }`.

## Testing

No test framework. Manual testing only:
1. Serve `plotter-app/`
2. Drop `test-data.csv` to verify time-series with 3 variables
3. Switch to XY (2 vars) and XYZ (3 vars) modes
4. Toggle merged/separate in time-series
5. Toggle theme
6. Test drag-and-drop, file input button, empty state

## Warnings

- **No build pipeline**. The `.gitignore` references Vite patterns (`node_modules`, `dist`, `dist-ssr`) but there is no `package.json`, `vite.config.*`, or `tsconfig.json`. This is a pure static app.
- **`.sisyphus/notepads/plotter-app/learnings.md` is completely stale**. It describes a Vite + React + TypeScript project that does not exist. Ignore it entirely — trust only the working tree.
- **ECharts 3D** (`echarts-gl`) is loaded but only used for XYZ mode. It must remain in `lib/` even if only 2D plots are used.
- **`renderApp()` destroys and rebuilds the entire DOM**. Any event listeners, animations, or external references to DOM elements are lost on each re-render.
