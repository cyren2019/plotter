# AGENTS.md — 时序绘图工具

## 项目性质

- **零依赖运行时**：纯 HTML/CSS/JS 单文件应用，无构建步骤、无 npm、无 Node.js。
- 唯一源码文件：`plotter-app/index.html`（约 1367 行，含 HTML + CSS + JS）。
- 第三方库以 `<script>` 直接引入，文件位于 `plotter-app/lib/`，不通过包管理器安装。

## 开发命令

```bash
# 启动本地静态服务器（必须从 plotter-app 目录运行，否则 lib/ 路径会 404）
cd plotter-app && python -m http.server 8080

# 备选
npx serve plotter-app
```

浏览器打开 `http://localhost:8080`，拖放 CSV / Excel 文件即可验证功能。

## 架构要点

### 状态管理

- 全局可变状态变量（位于 `<script>` 顶部）：
  - `data` — 解析后的数据集 `{ headers, rows, timeColumn }`
  - `selectedVars` — 用户勾选的变量数组
  - `plotType` — `'time-series' | 'xy' | 'xyz'`
  - `timeSeriesMode` — `'merged' | 'separate'`（时序图子模式）
  - `separateAutoScale` — 分图模式 Y 轴自适应开关
  - `restoreZoom` — 跨重绘保留的 dataZoom 状态
  - `chartInstance` — 当前 ECharts 实例（需手动 dispose）

### 渲染模式

- **`renderApp()`** — 全量重建 DOM（侧边栏变量列表、主区域图表容器、顶部标签）。
- **`renderChart()`** — 先 `chartInstance?.dispose()`，再 `echarts.init()` 重建。不要尝试增量更新 ECharts option，全量重建是唯一模式。

### 第三方库加载顺序（index.html `<head>` 底部）

```html
<script src="lib/dayjs.min.js"></script>
<script src="lib/customParseFormat.js"></script>
<script src="lib/xlsx.full.min.js"></script>
<script src="lib/echarts.min.js"></script>
<script src="lib/echarts-gl.min.js"></script>
```

- 若新增库，保持同样的顺序并放入 `lib/`。

### 时间解析（核心逻辑）

- 使用 `dayjs` 双阶段解析：先原生 `dayjs(str)`，再遍历 `TIME_FORMATS` 严格模式，最后宽松模式。
- 支持 Excel 序列号（`excelSerialToDate`）和 20+ 种字符串格式（含中文日期如 `2024年1月1日`）。
- `detectTimeColumn()` 按列名关键字 + 可解析比例自动识别时间列。
- 修改时间解析逻辑时，务必用 `test-data.csv` 和常见中文日期格式验证。

### 主题系统

- CSS 变量通过 `:root` 和 `[data-theme="dark"]` 切换。
- `applyTheme(bool)` 会同步更新 SVG 图标和图表（调用 `renderChart()`）。

## 文件与目录约定

```
plotter-app/
├── index.html          # 唯一源码（HTML + CSS + JS）
├── lib/                # 第三方库（不通过 npm，手动放置）
│   ├── dayjs.min.js
│   ├── customParseFormat.js
│   ├── xlsx.full.min.js
│   ├── echarts.min.js
│   └── echarts-gl.min.js
├── public/             # 静态资源（favicon.svg、icons.svg）
└── test-data.csv       # 示例数据（git 跟踪）
```

- `*.csv` 默认被 `.gitignore` 忽略，只有 `test-data.csv` 和 `test-data/*.csv` 会被跟踪。

## 测试与验证

- **无单元测试、无 CI。**
- 验证方式：浏览器中拖放 `test-data.csv` 或 Excel 文件，检查三种图表模式（时序合并 / 时序分图 / XY / XYZ）是否正常渲染。
- 时序图最多允许勾选 10 个变量，超过会提示并截断。

## 常见陷阱

- **不要在项目根目录运行 `python -m http.server`** — 这会导致 `lib/` 路径 404，因为应用入口在 `plotter-app/index.html`。
- **不要尝试保留 ECharts 实例做增量更新** — 代码约定是每次 `renderChart()` 都 `dispose()` + `init()`。
- **不要引入 npm 构建流程** — 项目设计目标就是零构建、单文件可直接用浏览器打开（从文件系统打开时，仅拖放功能受限，ECharts 渲染正常）。
- **修改 index.html 后无需重启服务器** — 纯静态文件，刷新浏览器即可。
