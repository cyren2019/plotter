[中文](README_zh-CN.md)

# 📈 Plotter — Time-Series Data Visualization Tool

A browser-based lightweight data visualization tool supporting CSV and Excel file imports. Automatically generates **time-series charts**, **XY scatter plots**, **XYZ 3D scatter plots**, and **FFT spectrum analysis**. Zero build steps, zero runtime dependencies — open and go.

![License](https://img.shields.io/badge/license-MIT-blue)
![Version](https://img.shields.io/badge/version-v1.2.1-blue)

---

## ✨ Features

### Multi-Format Import
- **CSV** files (auto-detects `,` / `;` delimiters, supports quoted field escaping)
- **Excel** `.xls` / `.xlsx` files
- **Log/Text** `.log` / `.txt` files
- Chinese encoding auto-detection (UTF-8 → GB18030/GBK/GB2312 fallback)

### Smart Time Parsing
- Auto-detects the first column as the time axis, supporting **20+ time formats**
- Includes ISO 8601, Excel serial numbers, Chinese dates (`2024年1月1日`), compact formats (`YYYYMMDDHHmmss`), and more
- Millisecond precision and Chinese time expressions (`HH时mm分ss秒`)

### Four Chart Modes

| Mode | Description | Variables |
|------|-------------|-----------|
| **Time-Series · Merged** | All variables in one chart, shared Y-axis | 1–10 |
| **Time-Series · Separate** | Independent subplot per variable, individual Y-axis with area zoom | 1–10 |
| **XY Scatter** | Two-variable 2D scatter | 2 |
| **XYZ 3D** | Three-variable 3D scatter (requires echarts-gl) | 3 |
| **FFT Spectrum** | FFT frequency-domain analysis with harmonic markers | 1 |

### FFT Spectrum Analysis (v1.2.1)
- Real-time FFT of selected time range via dataZoom slider
- **7 window functions**: Rect / Hann / Hamming / Blackman / Blackman-Harris / Flat-top / Kaiser, with automatic coherent-gain compensation
- **Spectrum averaging**: Single / Linear / Exponential / Peak-Hold (Max-Hold), 50% overlapping segments
- **Amplitude calibration**: Peak (Vpk) / RMS (Vrms) / Power Spectral Density (PSD, V²/Hz)
- **Auto-measurement panel**: THD, THD+N, SNR, SFDR, SINAD and noise floor displayed live
- Fundamental frequency auto-detection using harmonic support scoring (handles missing harmonics, e.g. square waves)
- Harmonic marker lines at f₀, 2f₀, 3f₀, …
- Multiple Y-axis modes: Physical (absolute), Per-Unit (normalized), dB (logarithmic), dBc (relative to fundamental)
- Configurable frequency units: Hz, rad/s, deg/s
- Manual sample rate override for non-timestamp data
- Automatic or manual fundamental frequency setting

### Interactions
- 🖱️ Scroll / drag to zoom and pan
- 🔄 Double-click chart to reset zoom
- 📐 Area selection zoom in separate mode
- 🌗 Light/dark theme toggle
- 🔍 Variable search and batch deselect
- ↔️ Draggable sidebar width
- ⏱️ Manual time column switching

### Binary Variable Detection & Bit-Level Visualization
- **Auto-detection**: Automatically identifies positive integer columns (max ≤ 65536) as splittable variables
- **8/16-bit Adaptive**: ≤255 expands to 8 bits, 256–65535 expands to 16 bits
- **DEC/BIN Toggle**: One-click switch between decimal and binary display in separate mode
- **Per-Bit Rows**: Each bit in its own row (bitN-1 → bit0, top to bottom), similar to a logic analyzer
- **Color-Coded**: 8-color cycle assignment for visual distinction between bits
- **Enhanced Tooltip**: Hovering any bit curve shows the complete state of all bits, with the current bit highlighted and bold
- **Merged Mode Protection**: Auto-resets to decimal display when switching to merged mode

### Time-Series Separate Mode
- **Auto-scale Y-axis**: Dynamically adjusts scale based on visible data range (`scale: true`)
- **Fixed Y-axis**: Locks scale based on global data range
- **View Reset**: One-click restore to initial zoom range
- **Zoom Memory**: Preserves current zoom position when switching variables/modes

### ⚡ Performance Optimizations
- **ECharts Instance Reuse**: Chart instance persists across renders — `setOption()` replaces full dispose+reinit (~10-50x faster per interaction)
- **Targeted DOM Updates**: Variable selection and mode toggles use lightweight class/DOM manipulation instead of full `renderApp()` rebuild
- **Search Debouncing**: Variable search input debounced at 150ms with CSS visibility filtering — no DOM recreation on keystroke
- **LTTB Data Sampling**: Large datasets (>10k rows) automatically use Largest-Triangle-Three-Buckets downsampling for smooth rendering
- **Single-Pass CSV Parsing**: Eliminated redundant normalization iteration — ~25% faster file import
- **Cached Y-Axis Ranges**: Min/max values computed once at import, reused across renders in fixed-scale mode

---

## 🚀 Quick Start

### Option 1: Static Server (Recommended)

```bash
cd plotter-app
python -m http.server 8080
# Open http://localhost:8080 in your browser
```

### Option 2: Direct Open

Drag `plotter-app/index.html` directly into your browser (some browsers may restrict file access due to security policies; using a static server is recommended).

### Option 3: Any HTTP Server

```bash
# Node.js
npx serve plotter-app

# PHP
php -S localhost:8080 -t plotter-app

# Python 3
python3 -m http.server 8080 -d plotter-app
```

---

## 📖 Usage

1. **Import Data** — Drag and drop a file onto the page, or click the "Import" button in the top-right corner
2. **Select Time Column** — Use the dropdown at the top of the sidebar to choose the time axis column
3. **Select Variables** — Check the variable names you want to plot
4. **Switch Chart Type** — Use the top tabs to switch between Time-Series / XY / XYZ / FFT
5. **Time-Series Mode** — Toggle between "Merged" and "Separate" display on the right
6. **FFT Analysis** — Select 1 variable, adjust sample rate and fundamental frequency if needed, click Apply
7. **Theme Toggle** — Use the "Light / Dark" button in the top-right corner

---

## 📁 Project Structure

```
plotter/
├── plotter-app/
│   ├── index.html              # App shell (HTML + CSS), loads module scripts in order
│   ├── test-data.csv           # Sample test data (1 kHz, 50 Hz sine / square / harmonics / binary)
│   ├── js/                     # Application modules (no build — plain scripts, ordered)
│   │   ├── app.js              # State, event binding, initialization
│   │   ├── i18n.js             # zh/en dictionaries and language switching
│   │   ├── file-parse.js       # CSV/Excel parsing, time detection, binary-column detection
│   │   ├── fft.js              # FFT math: windows, averaging, measurements, fundamental detection
│   │   └── chart-render.js     # renderApp/renderChart and per-mode ECharts option builders
│   ├── lib/                    # Third-party libraries (offline CDN copies)
│   │   ├── dayjs.min.js                # Date parsing (v1.x)
│   │   ├── customParseFormat.js        # dayjs strict format plugin
│   │   ├── xlsx.full.min.js            # Excel parsing (SheetJS)
│   │   ├── echarts.min.js              # Chart engine (Apache ECharts)
│   │   ├── echarts-gl.min.js           # 3D chart extension
│   │   └── fourier-transform.js        # FFT library (v2.4.1)
│   └── public/                 # Static assets
│       ├── favicon.svg
│       └── icons.svg
├── .gitignore
├── README.md
└── README_zh-CN.md
```

---

## 🏗️ Architecture

| Layer | Approach |
|-------|----------|
| **Architecture** | Single-file SPA (Single Page Application), no build steps |
| **State Management** | Global variables (`data`, `selectedVars`, `plotType`, `timeSeriesMode`, `binaryColumns`, `binaryBits`, `binaryMode`) |
| **Rendering** | `renderApp()` full DOM rebuild for structural changes; targeted class/DOM updates for interactions; ECharts instance reused via `setOption()` |
| **Time Parsing** | Two-stage: dayjs native parsing → 20+ format strict matching → loose fallback |
| **CSV Parsing** | Custom parser with quote escaping, delimiter auto-detection, encoding fallback |
| **Excel Parsing** | SheetJS (`xlsx.full.min.js`), reads the first sheet |
| **FFT Analysis** | 7 window functions → segment averaging (linear/exp/peak) → zero-pad to power-of-2 → RFFT via `fourier-transform.js` → coherent gain compensation → amplitude calibration (Vpk/Vrms/PSD) → harmonic-support fundamental detection → THD/SNR auto-measurements |
| **Chart Engine** | Apache ECharts 5.x + echarts-gl (3D support) |

### Data Flow

```
File Import → Encoding Detection → CSV/Excel Parsing → Time Column Auto-Detection
    → Variable List Rendering → User Selection → ECharts Chart Generation
```

### FFT Data Flow

```
Variable Selection → Zoom Range → Extract Values → Window Weighting → Segment Averaging → Zero-Pad → RFFT
    → Gain Compensation → Amplitude Calibration → Frequency Axis → Fundamental Detection → Spectrum + Harmonics + Auto-Measurements
```

### Encoding Fallback Strategy

```
UTF-8 (strict) → GB18030 (compatible with GBK/GB2312) → UTF-8 (lenient)
```

---

## 🌐 Browser Support

| Chrome | Edge | Firefox | Safari |
|--------|------|---------|--------|
| ✅ 90+ | ✅ 90+ | ✅ 88+ | ✅ 14+ |

---

## 📝 Development Guide

This project requires no build toolchain — edit the source file directly:

```bash
# Edit index.html, then refresh your browser — changes take effect immediately
```

Key functions:
- `parseTime()` — Time string parsing, 20+ format support
- `parseCSV()` / `parseExcel()` — File parsing entry points
- `renderApp()` — Full DOM rendering
- `renderChart()` — ECharts instance creation and configuration
- `buildTimeSeriesOption()` — Time-series chart option generation (merged/separate)
- `buildXYOption()` / `buildXYZOption()` — Scatter plot option generation
- `buildFFTOption()` — FFT spectrum chart option generation
- `computeFFT()` — FFT computation with Hann window and gain compensation
- `autoDetectBaseFreq()` — Fundamental frequency detection via harmonic scoring
- `detectBinaryColumns()` — Binary variable auto-detection
- `getBitColor()` / `setBinaryMode()` — Bit-level color assignment and mode toggle

---

## 📄 License

MIT

---
