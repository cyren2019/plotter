# AGENTS.md

## Repo Overview

This is a minimal static data-visualization app. The entire application lives in a single HTML file with embedded CSS and JavaScript.

- **Language**: Vanilla HTML/CSS/JS (no framework, no build step)
- **UI language**: Chinese (zh-CN)
- **Entry point**: `plotter-app/index.html`
- **No package manager**: There is no `package.json`, `requirements.txt`, or equivalent manifest.

## Project Structure

```
.
├── plotter-app/          <- The only code that matters
│   ├── index.html        <- Entire app (single file, ~1200 lines)
│   ├── lib/              <- Vendored minified libraries
│   │   ├── dayjs.min.js
│   │   ├── customParseFormat.js
│   │   ├── xlsx.full.min.js
│   │   ├── echarts.min.js
│   │   └── echarts-gl.min.js
│   ├── public/           <- Static assets (favicon.svg, icons.svg)
│   ├── test-data.csv     <- Sample input file
│   └── .gitignore        <- Standard Vite-like ignore list (no Vite config actually present)
├── .sisyphus/            <- OpenCode session artifacts
└── .playwright-mcp/      <- Playwright MCP tooling
```

## How to Run

Serve `plotter-app/` as a static directory and open it in a browser:

```bash
# Any static server works
npx serve plotter-app
python -m http.server -d plotter-app 8080
```

Then open the served URL. There is no build step, no dev server, and no hot reload.

## Architecture

- **Single-file app**: All markup, styles, and logic are inside `plotter-app/index.html`.
- **Libraries**: Loaded via `<script src="lib/...">`. Do not attempt to `npm install` anything.
- **Data flow**: User drags/drops or selects a CSV/Excel file → parsed with SheetJS (`xlsx`) or a custom CSV parser → time column auto-detected with `dayjs` → plotted with ECharts (2D or 3D via `echarts-gl`).
- **State management**: Plain global variables (`data`, `selectedVars`, `plotType`, `timeSeriesMode`, `chartInstance`).
- **Rendering**: Imperative DOM manipulation (`document.createElement`, `innerHTML`, `appendChild`). No virtual DOM.

## Conventions & Quirks

- **Time parsing** is custom and extensive. It handles Excel serial numbers, many human-readable formats, and Chinese date strings. See the `TIME_FORMATS` and `TIME_KEYWORDS` arrays in `index.html`.
- **Variable selection limits** are enforced per plot type:
  - Time-series: max 10 variables
  - XY: exactly 2 variables
  - XYZ: exactly 3 variables
- **Theme toggle** switches a `data-theme="dark"` attribute on `<html>`; colors are CSS custom properties.
- **Chart resize**: `echarts.init` is called on every re-render; the old instance is explicitly `.dispose()`-ed first.

## Testing

There is no test framework configured and no test commands.

## Warnings

- **`.sisyphus/notepads/plotter-app/learnings.md` is stale**. It describes a Vite + React + TypeScript setup with `src/` components, Tailwind CSS, and a `package.json`. None of that exists in the working tree. Trust only the files in `plotter-app/`.
- **`node_modules/` and `.vite/` are artifacts without a manifest**. They contain only a few cached packages (`@rolldown`, `@tailwindcss`, `lightningcss-win32-x64-msvc`) but no active build pipeline. Do not treat this as a Node project.

## Git

- Single branch: `main`
- Clean working tree (no uncommitted changes at time of writing)
