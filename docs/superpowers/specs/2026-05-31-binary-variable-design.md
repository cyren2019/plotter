# 二进制变量标识与按位拆分 — 设计文档

## 背景

用户在导入数据时，某些列的值实际上是位标志（如状态码、报警码等十进制整数），在图表中作为连续数值曲线显示缺乏可读性。本功能自动识别这类列，在变量选择栏中允许用户在十进制和二进制之间切换，切换后在分图模式下按位拆分为多条 0/1 线。

## 需求汇总

- 识别列中所有值为 ≥0 的整数、且最大值 ≤ 65536 的列
- ≤255 展开为 8 位，256~65535 展开为 16 位
- 变量列表中显示 BIN 标签和 DEC/BIN 切换按钮（方案 A：行内标识 + 切换）
- **仅分图模式可用二进制切换**，合并模式始终显示十进制
- 二进制变量在 10 个变量限制中占 1 个名额
- 按位拆分后每条线显示为 `变量名 · bitN`，Y 轴固定 0~1.2

## 架构

### 全局状态

在 `<script>` 顶部现有全局变量旁新增：

```js
let binaryColumns = new Set();      // 列名集合
let binaryBits = new Map();         // 列名 → 位数 (8 或 16)
let binaryMode = new Map();         // 列名 → 'dec' | 'bin'
```

### 检测函数

```js
function detectBinaryColumns(headers, rows) {
  const timeColumn = data.timeColumn; // 需要传入
  const cols = new Set();
  const bits = new Map();

  for (const h of headers) {
    if (h === timeColumn) continue;

    let maxVal = 0;
    let isPositiveInt = true;

    for (const row of rows) {
      const v = row[h];
      if (v === null || v === undefined) continue;
      const n = Number(v);
      if (!Number.isInteger(n) || n < 0 || !Number.isFinite(n)) {
        isPositiveInt = false;
        break;
      }
      if (n > maxVal) maxVal = n;
    }

    if (isPositiveInt && maxVal <= 65536) {
      cols.add(h);
      bits.set(h, maxVal <= 255 ? 8 : 16);
    }
  }

  return { cols, bits };
}
```

### 状态初始化与重置

在 `processFile()` 中，文件解析成功后：

```js
const { cols, bits } = detectBinaryColumns(data.headers, data.rows);
binaryColumns = cols;
binaryBits = bits;
binaryMode = new Map(); // 清空上次的模式选择
for (const col of cols) {
  binaryMode.set(col, 'dec'); // 默认十进制
}
```

在 `plotType` 切换为 `time-series` 且当前为合并模式时，确保所有 `binaryMode` 为 `'bin'` 的重置为 `'dec'`（`renderApp()` 中 plotType 切换处理处添加）。

### UI 渲染（renderApp 中的 .var-item）

每个变量的渲染逻辑修改如下：

```js
const isBinary = binaryColumns.has(v);
const mode = isBinary ? (binaryMode.get(v) || 'dec') : null;
const isMerged = plotType === 'time-series' && timeSeriesMode === 'merged';

// 在变量名之后添加二进制标识
if (isBinary) {
  // BIN 标签（始终显示，绿色小圆角 badge）
  const badge = `<span class="bin-badge">BIN</span>`;

  // 合并模式：切换按钮置灰
  // 分图模式：切换按钮可用
  const disabled = isMerged;
  const toggle = `
    <div class="bin-toggle" onclick="event.stopPropagation()">
      <button class="${mode === 'dec' ? 'active' : ''}" ${disabled ? 'disabled' : ''}
              onclick="setBinaryMode('${escapeHtml(v)}', 'dec')">DEC</button>
      <button class="${mode === 'bin' ? 'active' : ''}" ${disabled ? 'disabled' : ''}
              onclick="setBinaryMode('${escapeHtml(v)}', 'bin')">BIN</button>
    </div>
  `;
  // 拼接到 .var-item 的 innerHTML 中
}
```

**新增 CSS**：

```css
.bin-badge {
  font-size: 9px; padding: 1px 5px;
  border-radius: 9999px;
  background: #22c55e; color: white; font-weight: 500;
  flex-shrink: 0;
}
[data-theme="dark"] .bin-badge { background: #16a34a; }
.bin-toggle { display: flex; background: var(--border); border-radius: 6px; padding: 1px; gap: 1px; flex-shrink: 0; }
.bin-toggle button {
  padding: 2px 8px; font-size: 10px; border: none;
  background: transparent; color: var(--text); cursor: pointer;
  border-radius: 4px; transition: all 0.1s;
}
.bin-toggle button.active { background: var(--accent); color: white; }
.bin-toggle button:disabled { opacity: 0.35; cursor: not-allowed; }
.bin-toggle button:not(:disabled):not(.active):hover { opacity: 0.7; }
.var-item[data-binary] { background: #f0fdf4; }
[data-theme="dark"] .var-item[data-binary] { background: #14532d22; }
```

### 切换函数

```js
function setBinaryMode(varName, mode) {
  binaryMode.set(varName, mode);
  renderApp();   // 重建 UI
  renderChart(); // 重建图表
}
```

### 图表渲染（buildTimeSeriesOption）

在 `buildTimeSeriesOption()` 的 `varsToPlot` 遍历中，对每个变量检查：

**合并模式**（现有逻辑不变）：始终按十进制绘制

**分图模式**：

```js
const isBinVar = binaryColumns.has(variable) && binaryMode.get(variable) === 'bin';
const bitCount = isBinVar ? (binaryBits.get(variable) || 8) : 0;

if (isBinVar) {
  // 按位拆分为多条线
  for (let bit = 0; bit < bitCount; bit++) {
    series.push({
      name: `${variable} · bit${bit}`,
      type: 'line',
      data: rows.map(row => {
        const val = Number(row[variable]);
        if (val === null || val === undefined || isNaN(val) || !isFinite(val)) return null;
        return (val >> bit) & 1;
      }),
      symbol: 'none',
      lineStyle: { width: 1 },
      xAxisIndex: i,
      yAxisIndex: i,
      // 颜色：同一变量的不同 bit 使用同一色系
      itemStyle: { color: getBitColor(bit, bitCount) },
    });
  }

  // Y 轴固定 0~1.2
  yAxisConfig.min = 0;
  yAxisConfig.max = 1.2;
  yAxisConfig.scale = false; // 禁用自适应
} else {
  // 现有十进制逻辑不变
}
```

**颜色函数** `getBitColor(bit, bitCount)`：

- 使用固定色板循环分配：`['#3b82f6','#22c55e','#f59e0b','#ef4444','#8b5cf6','#06b6d4','#ec4899','#84cc16']`（8 色循环，16 位时前 8 色重复）

### 模式切换时的状态重置

在 plotType 切换事件和 timeSeriesMode 切换事件中：

```js
// 切换为合并模式时，重置所有 bin 模式为 dec
if (plotType === 'time-series' && timeSeriesMode === 'merged') {
  for (const col of binaryColumns) {
    binaryMode.set(col, 'dec');
  }
}
```

## 修改文件

- `plotter-app/index.html` — 唯一修改文件（~1500 行 → 预计增加 ~150 行）

## 验证

1. 导入 `test-data.csv`，确认无二进制列被错误识别
2. 导入包含 0/1 列的文件，确认显示 BIN 标签
3. 切换到分图模式，点击 BIN → DEC 切换按钮，确认图表按位拆分
4. 切换到合并模式，确认切换按钮置灰
5. 确认 ≤255 显示 8 位，>255 显示 16 位
6. 确认主题切换后 BIN 标签颜色适配暗色主题
