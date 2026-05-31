# 二进制变量标识与按位拆分 — 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 自动识别正整数列（max ≤ 65536），在变量选择栏显示 BIN 标识和 DEC/BIN 切换按钮，分图模式下可切换为按位拆分显示。

**Architecture:** 在现有 `index.html` 中新增检测函数、状态变量、UI 渲染逻辑和图表拆分逻辑。所有改动集中在单文件。

**Tech Stack:** Vanilla JS, Apache ECharts 5.x, CSS variables

---

## 文件结构

| 文件 | 操作 | 说明 |
|------|------|------|
| `plotter-app/index.html` | 修改 | 唯一修改文件，预计增加 ~150 行 |

所有改动都在 `index.html` 内，按功能分区：
1. **CSS 区域**（`<style>` 底部）— BIN badge 和 toggle 样式
2. **状态区域**（全局变量声明）— `binaryColumns`, `binaryBits`, `binaryMode`
3. **检测函数**（`isTimeString` 后）— `detectBinaryColumns()`
4. **初始化**（`processFile()` 内）— 检测 + 状态初始化
5. **UI 渲染**（`renderApp()` 的 .var-item 渲染）— 添加 BIN 标识和切换按钮
6. **图表渲染**（`buildTimeSeriesOption()`）— 分图模式下按位拆分
7. **模式重置**（plotType/timeSeriesMode 切换事件）— 合并模式重置所有 BIN 为 DEC

---

## 前置知识

- 项目架构：单文件 SPA，`renderApp()` 全量重建 DOM，`renderChart()` 销毁并重建 ECharts 实例
- 时间解析：`parseTime()` 在 1305-1362 行，支持 20+ 格式
- 数据流：`parseFile()` → `processFile()` → `renderApp()` → `renderChart()`
- 合并模式：`timeSeriesMode === 'merged'`，单图表共享 Y 轴
- 分图模式：`timeSeriesMode === 'separate'`，每个变量独立子图
- 变量选择：`selectedVars` 数组，`renderApp()` 中 `.var-item` 的 click 事件处理
- 现有全局变量声明在 1126-1133 行

---

### Task 1: 全局状态变量

**Files:**
- Modify: `plotter-app/index.html` — 全局变量声明区域

- [ ] **Step 1: 添加全局状态变量**

在现有全局变量声明区域（`let dragCounter = 0;` 之后）添加：

```js
    // ===================== Binary Columns =====================
    let binaryColumns = new Set();      // column names detected as binary
    let binaryBits = new Map();         // column name → bit count (8 or 16)
    let binaryMode = new Map();         // column name → 'dec' | 'bin'
```

- [ ] **Step 2: 提交**

```bash
git add plotter-app/index.html
git commit -m "feat: add binary column state variables"
```

---

### Task 2: 检测函数

**Files:**
- Modify: `plotter-app/index.html` — 在 `isTimeString()` 函数后添加

- [ ] **Step 1: 添加检测函数**

在 `isTimeString()` 函数（约 481-483 行）之后添加：

```js
    function detectBinaryColumns(headers, rows, timeColumn) {
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

- [ ] **Step 2: 提交**

```bash
git add plotter-app/index.html
git commit -m "feat: add detectBinaryColumns function"
```

---

### Task 3: 状态初始化

**Files:**
- Modify: `plotter-app/index.html` — `processFile()` 函数

- [ ] **Step 1: 在 processFile 中添加检测调用**

在 `processFile()` 函数中，找到 `data = parsed;` 之后的代码（约 723 行），在 `selectedVars = nonTime.slice(0, 3);` 之前添加：

```js
        // Detect binary columns
        const { cols, bits } = detectBinaryColumns(parsed.headers, parsed.rows, parsed.timeColumn);
        binaryColumns = cols;
        binaryBits = bits;
        binaryMode = new Map();
        for (const col of cols) {
          binaryMode.set(col, 'dec');
        }
```

- [ ] **Step 2: 提交**

```bash
git add plotter-app/index.html
git commit -m "feat: initialize binary state on file import"
```

---

### Task 4: CSS 样式

**Files:**
- Modify: `plotter-app/index.html` — `<style>` 区域

- [ ] **Step 1: 添加 BIN badge 和 toggle 的 CSS**

在 `:: -webkit-scrollbar` 样式之前（约 362 行），添加：

```css
    /* Binary column indicators */
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
    [data-theme="dark"] .var-item[data-binary] { background: rgba(22, 163, 74, 0.08); }
```

- [ ] **Step 2: 提交**

```bash
git add plotter-app/index.html
git commit -m "style: add binary badge and toggle CSS"
```

---

### Task 5: 切换函数

**Files:**
- Modify: `plotter-app/index.html` — 在 `renderApp()` 函数之前添加

- [ ] **Step 1: 添加 setBinaryMode 函数**

在 `renderApp()` 函数（约 763 行）之前添加：

```js
    function setBinaryMode(varName, mode) {
      binaryMode.set(varName, mode);
      renderApp();
      renderChart();
    }
```

- [ ] **Step 2: 提交**

```bash
git add plotter-app/index.html
git commit -m "feat: add setBinaryMode toggle function"
```

---

### Task 6: 合并模式下的状态重置

**Files:**
- Modify: `plotter-app/index.html` — plotType 和 timeSeriesMode 切换事件

- [ ] **Step 1: 修改 plotType 切换事件**

找到 `plotSection.querySelectorAll('.plot-tabs .btn[data-type]')` 的 click 事件（约 950 行），在 `plotType = btn.dataset.type;` 之后添加状态重置：

```js
          plotType = btn.dataset.type;
          // Reset binary mode to dec when switching to merged mode
          if (plotType === 'time-series' && timeSeriesMode === 'merged') {
            for (const col of binaryColumns) {
              binaryMode.set(col, 'dec');
            }
          }
```

- [ ] **Step 2: 修改 timeSeriesMode 切换事件**

找到 `plotSection.querySelectorAll('.ts-toggle .btn[data-mode]')` 的 click 事件（约 960 行），在 `timeSeriesMode = btn.dataset.mode;` 之后添加：

```js
          timeSeriesMode = btn.dataset.mode;
          // Reset binary mode to dec when switching to merged mode
          if (timeSeriesMode === 'merged') {
            for (const col of binaryColumns) {
              binaryMode.set(col, 'dec');
            }
          }
```

- [ ] **Step 3: 提交**

```bash
git add plotter-app/index.html
git commit -m "feat: reset binary mode when switching to merged"
```

---

### Task 7: UI 渲染 — 变量列表中的 BIN 标识

**Files:**
- Modify: `plotter-app/index.html` — `renderApp()` 中的 `renderVars()` 函数

- [ ] **Step 1: 修改 .var-item 渲染逻辑**

找到 `renderVars()` 函数中创建 `.var-item` 的部分（约 875-906 行），将现有的 `item.innerHTML` 替换为：

```js
        filtered.forEach(v => {
          const item = document.createElement('div');
          const isSelected = selectedVars.includes(v);
          const isBinary = binaryColumns.has(v);
          const binMode = isBinary ? (binaryMode.get(v) || 'dec') : null;
          const isMerged = plotType === 'time-series' && timeSeriesMode === 'merged';

          item.className = 'var-item' + (isSelected ? ' selected' : '');
          if (isBinary) item.setAttribute('data-binary', '');

          // Build right-side content (binary indicator + dot)
          let rightContent = '';
          if (isBinary) {
            // Preview: show binary or decimal value
            const sampleRow = data.rows.find(r => Number(r[v]) !== null && Number(r[v]) !== undefined && Number.isInteger(Number(r[v])));
            let preview = '';
            if (sampleRow) {
              const numVal = Number(sampleRow[v]);
              if (binMode === 'bin') {
                const bits = binaryBits.get(v) || 8;
                preview = numVal.toString(2).padStart(bits, '0');
              } else {
                preview = String(numVal);
              }
            }
            const disabledAttr = isMerged ? 'disabled' : '';
            const decActive = binMode === 'dec' ? 'active' : '';
            const binActive = binMode === 'bin' ? 'active' : '';
            const titleText = isMerged ? '切换至分图模式可用' : '';
            rightContent = `
              <span class="bin-badge" ${isMerged ? 'title="切换至分图模式可用"' : ''}>BIN</span>
              <div class="bin-toggle" onclick="event.stopPropagation()">
                <button class="${decActive}" ${disabledAttr} title="${titleText}"
                  onclick="event.stopPropagation();setBinaryMode('${escapeHtml(v)}','dec')">DEC</button>
                <button class="${binActive}" ${disabledAttr} title="${titleText}"
                  onclick="event.stopPropagation();setBinaryMode('${escapeHtml(v)}','bin')">BIN</button>
              </div>
            `;
          }
          if (isSelected) {
            rightContent += '<div class="dot"></div>';
          }

          item.innerHTML = `
            <div class="checkbox">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><polyline points="20 6 9 17 4 12"/></svg>
            </div>
            <span class="name" title="${escapeHtml(v)}">${escapeHtml(v)}</span>
            ${rightContent}
          `;
          item.addEventListener('click', () => {
```

（`item.addEventListener('click', () => {` 之后的代码保持不变）

同时，需要在 `.selected-count` 的 HTML 模板中，将 `escapeHtml` 函数移到 `renderApp()` 之前可用。检查现有代码，`escapeHtml` 已经定义在全局区域（约 648 行），所以不需要额外处理。

- [ ] **Step 2: 提交**

```bash
git add plotter-app/index.html
git commit -m "feat: render binary indicators in variable list"
```

---

### Task 8: 按位拆分颜色函数

**Files:**
- Modify: `plotter-app/index.html` — 在 `getChartMessage()` 函数附近添加

- [ ] **Step 1: 添加 getBitColor 函数**

在 `getChartMessage()` 函数（约 1129 行）之后添加：

```js
    const BIT_COLORS = [
      '#3b82f6', '#22c55e', '#f59e0b', '#ef4444',
      '#8b5cf6', '#06b6d4', '#ec4899', '#84cc16',
    ];

    function getBitColor(bit) {
      return BIT_COLORS[bit % BIT_COLORS.length];
    }
```

- [ ] **Step 2: 提交**

```bash
git add plotter-app/index.html
git commit -m "feat: add getBitColor function for bit series"
```

---

### Task 9: 图表渲染 — 分图模式按位拆分

**Files:**
- Modify: `plotter-app/index.html` — `buildTimeSeriesOption()` 函数

- [ ] **Step 1: 修改分图模式的变量遍历逻辑**

找到 `buildTimeSeriesOption()` 中 `varsToPlot.forEach((variable, i) => {` 循环内（约 1228 行），在 `series.push({` 之前（约 1300 行），添加二进制检测：

在以下代码位置添加二进制分支：

```js
        const isBinVar = plotType === 'time-series' && timeSeriesMode === 'separate'
          && binaryColumns.has(variable) && binaryMode.get(variable) === 'bin';
        const bitCount = isBinVar ? (binaryBits.get(variable) || 8) : 0;

        if (isBinVar) {
          // Binary mode: split into bit series
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
              itemStyle: { color: getBitColor(bit) },
            });
          }

          // Y-axis fixed for binary mode
          yAxisConfig.min = 0;
          yAxisConfig.max = 1.2;
          yAxisConfig.scale = false;
        } else {
          // Existing decimal logic (the current series.push block)
```

在现有的 `series.push({ ... })` 的 `}` 之后添加 `}` 闭合 else 块。

现有的 `series.push()` 代码（约 1300-1313 行）需要包裹在 else 块中：

```js
        } else {
          series.push({
            name: variable,
            type: 'line',
            data: rows.map(row => {
              const val = row[variable];
              if (val === null || val === undefined || isNaN(Number(val)) || !isFinite(Number(val))) return null;
              return Number(val);
            }),
            symbol: 'none',
            xAxisIndex: i,
            yAxisIndex: i,
            lineStyle: { width: 1.5 },
          });
        }
```

- [ ] **Step 2: 提交**

```bash
git add plotter-app/index.html
git commit -m "feat: split binary vars into bit series in separate mode"
```

---

### Task 10: 验证与测试

**Files:**
- Test: 浏览器手动验证

- [ ] **Step 1: 创建测试数据**

创建 `test-data/binary-test.csv`：

```csv
timestamp,status_flag,temperature,counter,code
2024-01-01 00:00:00,1,23.5,100,42
2024-01-01 01:00:00,0,24.1,101,0
2024-01-01 02:00:00,1,22.8,102,255
2024-01-01 03:00:00,0,25.0,103,1
2024-01-01 04:00:00,1,23.2,104,128
2024-01-01 05:00:00,0,24.5,105,65535
2024-01-01 06:00:00,1,23.9,106,300
2024-01-01 07:00:00,0,24.2,107,15
```

- [ ] **Step 2: 启动服务器并验证**

```bash
cd plotter-app && python -m http.server 8080
```

浏览器打开 http://localhost:8080，导入 `test-data/binary-test.csv`：

1. `status_flag`（值 0/1，max=1 ≤ 255）→ 应显示 BIN 标签，8 位
2. `temperature`（值 23.5 等，非整数）→ 不应显示 BIN 标签
3. `counter`（值 100~107，整数，max=107 ≤ 255）→ 应显示 BIN 标签，8 位
4. `code`（值含 65535，整数，max=65535 ≤ 65536）→ 应显示 BIN 标签，16 位
5. 切换到**分图模式** → DEC/BIN 切换按钮可用
6. 点击 `status_flag` 的 BIN 按钮 → 图表拆分为 8 条线（bit0~bit7）
7. 切换到**合并模式** → 切换按钮置灰，BIN 状态重置为 DEC
8. 切换明暗主题 → BIN 标签和 toggle 颜色适配

- [ ] **Step 3: 提交测试数据**

```bash
git add test-data/binary-test.csv
git commit -m "test: add binary column test data"
```

---

## Self-Review

**Spec coverage:**
- ✅ 识别正整数列 max ≤ 65536 — Task 2 `detectBinaryColumns`
- ✅ ≤255 → 8 位，256~65535 → 16 位 — Task 2
- ✅ BIN 标签 + 切换按钮 — Task 4 (CSS) + Task 7 (UI)
- ✅ 仅分图模式可用 — Task 7 (disabled 属性) + Task 6 (重置)
- ✅ 占 1 个名额 — 不修改 `getMaxSelectable()` 或 `selectedVars` 逻辑
- ✅ 按位拆分 0/1 线 — Task 9
- ✅ Y 轴固定 0~1.2 — Task 9
- ✅ 合并模式重置为 DEC — Task 6
- ✅ 主题适配 — Task 4 (dark mode CSS)

**Placeholder scan:** 无 TBD/TODO，所有步骤都有完整代码

**Type consistency:** 
- `binaryColumns` 是 Set，使用 `.has()` — ✅
- `binaryBits` 是 Map，使用 `.get()` — ✅
- `binaryMode` 是 Map，使用 `.set()`/`.get()` — ✅
- 函数签名 `detectBinaryColumns(headers, rows, timeColumn)` 与调用一致 — ✅
- `getBitColor(bit)` 返回 HEX 颜色字符串 — ✅

Plan is clean.
