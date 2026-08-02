[English](README.md)

# 📈 Plotter — 时序数据绘图工具

一个基于浏览器的轻量级数据可视化工具，支持 CSV 与 Excel 文件导入，自动生成**时序图**、**XY 散点图**、**XYZ 3D 散点图**与 **FFT 频谱分析**。零构建、零依赖运行时，打开即用。

![License](https://img.shields.io/badge/license-MIT-blue)
![Version](https://img.shields.io/badge/version-v1.2.2-blue)

---

## ✨ 特性

### 多格式导入
- **CSV** 文件（自动识别 `,` / `;` 分隔符，支持引号字段转义）
- **Excel** `.xls` / `.xlsx` 文件
- **日志/文本** `.log` / `.txt` 文件
- 中文编码自动检测（UTF-8 → GB18030/GBK/GB2312 回退）

### 智能时间解析
- 自动识别第一列为时间轴，支持 **20+ 种时间格式**
- 包括 ISO 8601、Excel 序列号、中文日期（`2024年1月1日`）、紧凑格式（`YYYYMMDDHHmmss`）等
- 支持毫秒级精度与 `HH时mm分ss秒` 中文时间表达

### 四种图表模式

| 模式 | 说明 | 变量数 |
|------|------|--------|
| **时序图 · 合并** | 所有变量绘制在同一图表，共用 Y 轴 | 1–10 |
| **时序图 · 分图** | 每变量独立子图，独立 Y 轴，支持框选缩放 | 1–10 |
| **XY 散点图** | 双变量二维散点 | 2 |
| **XYZ 3D 图** | 三变量三维散点（需 echarts-gl） | 3 |
| **FFT 频谱** | FFT 频域分析，含谐波标记 | 1 |

### FFT 频谱分析 (v1.2.2)
- 通过 dataZoom 滑块实时 FFT 所选时间范围
- **7 种窗函数**：矩形 / Hann / Hamming / Blackman / Blackman-Harris / 平顶 / Kaiser，自动相干增益补偿
- **频谱平均化**：单次 / 线性平均 / 指数平均 / 峰值保持（Max-Hold），50% 重叠分段
- **幅度标定**：峰值 Vpk / 有效值 Vrms / 功率谱密度 PSD（V²/Hz）
- **自动测量面板**：THD、THD+N、SNR、SFDR、SINAD、噪底实时显示
- 基于谐波支撑评分的基频自动检测（支持缺次谐波信号，如方波）
- 谐波标记线（f₀, 2f₀, 3f₀, …）
- 多种 Y 轴模式：物理值（绝对值）、标幺值（归一化）、dB（对数坐标）、dBc（相对基波）
- 频率单位可切换：Hz、rad/s、deg/s、rpm
- 手动采样率覆盖（适用于非时间戳数据）
- 基频支持自动检测或手动设定

### 交互操作
- 🖱️ 滚轮 / 拖拽缩放、平移
- 🔄 双击图表复位缩放
- 📐 分图模式支持框选区域缩放
- 🌗 明暗主题实时切换
- 🔍 变量搜索、批量取消选择
- ↔️ 侧边栏宽度可拖拽调整
- ⏱️ 手动切换时间轴列

### 二进制变量识别与按位拆分
- **自动检测**：自动识别正整数列（max ≤ 65536），标记为可拆分变量
- **8/16 位自适应**：≤255 展开为 8 位，256~65535 展开为 16 位
- **DEC/BIN 切换**：分图模式下可一键切换十进制/二进制显示
- **按位分行**：每个 bit 独立一行（bitN-1 → bit0 从上到下），类似逻辑分析仪
- **彩色编码**：8 色循环赋值，不同 bit 以不同颜色区分
- **增强 tooltip**：悬停任意 bit 曲线时，显示所有 bit 的完整状态，当前 bit 高亮加粗
- **合并模式保护**：切换到合并模式时自动重置为十进制显示

### 时序图分图模式
- **自适应 Y 轴**：自动根据可见数据范围调整刻度（`scale: true`）
- **固定 Y 轴**：基于全局数据范围锁定刻度
- **图像复位**：一键恢复初始缩放范围
- **缩放记忆**：切换变量/模式时保留当前缩放位置

### ⚡ 性能优化
- **ECharts 实例复用**：图表实例跨渲染复用 — `setOption()` 替代每次 dispose+reinit（单次交互提升约 10-50 倍）
- **定向 DOM 更新**：变量勾选和模式切换使用轻量 class/DOM 操作替代全量 `renderApp()` 重建
- **搜索防抖**：变量搜索 150ms 防抖 + CSS visibility 过滤 — 按键不再触发 DOM 重建
- **LTTB 数据采样**：大数据集（>1万行）自动启用 Largest-Triangle-Three-Buckets 降采样，渲染流畅
- **CSV 单次遍历**：消除冗余的归一化遍历 — 文件导入快约 25%
- **Y 轴范围缓存**：导入时一次性计算 min/max，固定刻度模式下复用

---

## 🚀 快速开始

### 方式一：静态服务器（推荐）

```bash
cd plotter-app
python -m http.server 8080
# 浏览器打开 http://localhost:8080
```

### 方式二：直接打开

直接在浏览器中拖入 `plotter-app/index.html` 文件即可使用（部分浏览器可能因安全策略限制文件读取，建议使用静态服务器）。

### 方式三：任意 HTTP 服务器

```bash
# Node.js
npx serve plotter-app

# PHP
php -S localhost:8080 -t plotter-app

# Python 3
python3 -m http.server 8080 -d plotter-app
```

---

## 📖 使用说明

1. **导入数据** — 拖放文件到页面，或点击右上角「导入」按钮
2. **选择时间列** — 左侧栏顶部下拉菜单选择作为时间轴的列
3. **选择变量** — 勾选需要绘制的变量名
4. **切换图表** — 顶部标签栏切换 时序图 / XY图 / XYZ图 / FFT
5. **时序图模式** — 右侧「合并 / 分图」切换显示方式
6. **FFT 分析** — 选择 1 个变量，根据需要调整采样率和基频，点击应用
7. **主题切换** — 右上角「明亮 / 黑暗」按钮

---

## 📁 项目结构

```
plotter/
├── plotter-app/
│   ├── index.html              # 应用外壳（HTML + CSS），按序加载模块脚本
│   ├── test-data.csv           # 示例测试数据（1 kHz，50 Hz 正弦/方波/谐波/二进制）
│   ├── js/                     # 应用模块（无构建，普通 script 按序加载）
│   │   ├── app.js              # 状态管理、事件绑定、初始化
│   │   ├── i18n.js             # 中英文字典与语言切换
│   │   ├── file-parse.js       # CSV/Excel 解析、时间列识别、二进制列检测
│   │   ├── fft.js              # FFT 计算：窗函数、平均化、测量指标、基频检测
│   │   └── chart-render.js     # renderApp/renderChart 及各模式图表 option 构建
│   ├── lib/                    # 第三方库（CDN 离线化）
│   │   ├── dayjs.min.js                # 日期解析 (v1.x)
│   │   ├── customParseFormat.js        # dayjs 严格解析插件
│   │   ├── xlsx.full.min.js            # Excel 解析 (SheetJS)
│   │   ├── echarts.min.js              # 图表引擎 (Apache ECharts)
│   │   ├── echarts-gl.min.js           # 3D 图表扩展
│   │   └── fourier-transform.js        # FFT 库 (v2.4.1)
│   └── public/                 # 静态资源
│       ├── favicon.svg
│       └── icons.svg
├── .gitignore
├── README.md
└── README_zh-CN.md
```

---

## 🏗️ 技术架构

| 层面 | 方案 |
|------|------|
| **架构** | 单文件 SPA（Single Page Application），无构建步骤 |
| **状态管理** | 全局变量（`data`, `selectedVars`, `plotType`, `timeSeriesMode`, `binaryColumns`, `binaryBits`, `binaryMode`） |
| **渲染模式** | 结构性变化时 `renderApp()` 全量重建 DOM；交互操作使用定向 class/DOM 更新；ECharts 实例复用 `setOption()` |
| **时间解析** | 双阶段：dayjs 原生解析 → 20+ 格式严格匹配 → 宽松匹配兜底 |
| **CSV 解析** | 自定义解析器，支持引号转义、分隔符自动检测、编码回退 |
| **Excel 解析** | SheetJS (`xlsx.full.min.js`)，读取首个 Sheet |
| **FFT 分析** | 7 种窗函数 → 分段平均（线性/指数/峰值保持）→ 零填充至 2 的幂 → RFFT → 相干增益补偿 → 幅度标定（Vpk/Vrms/PSD）→ 谐波支撑评分基频检测 → THD/SNR 自动测量 |
| **图表引擎** | Apache ECharts 5.x + echarts-gl（3D 支持） |

### 数据流

```
文件导入 → 编码检测 → CSV/Excel 解析 → 时间列自动识别
    → 变量列表渲染 → 用户选择 → ECharts 图表生成
```

### FFT 数据流

```
变量选择 → 缩放范围 → 提取数值 → 窗函数加权 → 分段平均 → 零填充 → RFFT
    → 增益补偿 → 幅度标定 → 频率轴 → 基频检测 → 频谱图 + 谐波标记 + 自动测量
```

### 编码回退策略

```
UTF-8 (strict) → GB18030 (兼容 GBK/GB2312) → UTF-8 (lenient)
```

---

## 🌐 浏览器支持

| Chrome | Edge | Firefox | Safari |
|--------|------|---------|--------|
| ✅ 90+ | ✅ 90+ | ✅ 88+ | ✅ 14+ |

---

## 📝 开发指南

本项目无需构建工具链，直接编辑源文件即可：

```bash
# 编辑 index.html 后刷新浏览器即生效
```

关键函数：
- `parseTime()` — 时间字符串解析，支持 20+ 格式
- `parseCSV()` / `parseExcel()` — 文件解析入口
- `renderApp()` — DOM 全量渲染
- `renderChart()` — ECharts 实例创建与配置
- `buildTimeSeriesOption()` — 时序图配置生成（合并/分图）
- `buildXYOption()` / `buildXYZOption()` — 散点图配置生成
- `buildFFTOption()` — FFT 频谱图配置生成
- `computeFFT()` — FFT 计算，含 Hann 窗和增益补偿
- `autoDetectBaseFreq()` — 谐波评分基频检测
- `detectBinaryColumns()` — 二进制变量自动检测
- `getBitColor()` / `setBinaryMode()` — 按位拆分颜色与模式切换

---

## 📄 许可证

MIT

---

