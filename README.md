# 数据绘图工具

一个基于浏览器的轻量数据可视化工具，支持 CSV 与 Excel 文件导入，自动生成时序图、XY 散点图与 XYZ 3D 图。

## 特性

- **零依赖运行时**：纯 HTML/CSS/JS 单文件应用，无需构建、无需 Node.js
- **多格式支持**：CSV（自动识别 `,` / `;` 分隔符）、Excel `.xls` / `.xlsx`
- **智能时间解析**：自动识别 20+ 种时间格式（含 Excel 序列号、中文日期如 `2024年1月1日` 等）
- **三种图表模式**
  - **时序图**：支持合并视图（单图多线）与分图视图（每变量独立子图）
  - **XY 散点图**：双变量二维散点
  - **XYZ 3D 图**：三变量三维散点（需 echarts-gl）
- **交互操作**
  - 滚轮 / 拖拽缩放、平移
  - 双击图表复位
  - 框选区域缩放（分图模式）
  - 明暗主题切换
- **变量管理**：侧边栏搜索、批量取消选择、数量限制提示

## 快速开始

```bash
# 进入应用目录并启动静态服务器
cd plotter-app
python -m http.server 8080

# 或
npx serve plotter-app
```

浏览器打开 `http://localhost:8080`，拖放或点击导入数据文件即可。

## 项目结构

```
plotter-app/
├── index.html          # 完整应用（HTML + CSS + JS，约 1300 行）
├── lib/                # 第三方库（直接引用，不通过 npm）
│   ├── dayjs.min.js + customParseFormat.js
│   ├── xlsx.full.min.js
│   ├── echarts.min.js
│   └── echarts-gl.min.js
├── public/             # 静态资源（favicon、图标）
└── test-data.csv       # 示例数据
```

## 使用说明

1. **导入数据**：拖放文件到页面，或点击右上角「导入」按钮
2. **选择变量**：左侧栏勾选需要绘制的变量（时序图最多 10 个）
3. **切换图表**：顶部标签切换 时序图 / XY图 / XYZ图
4. **时序图模式**：
   - **合并**：所有变量绘制在同一图表，共用 Y 轴
   - **分图**：每个变量独立子图，独立 Y 轴，支持框选缩放
   - **自适应**：自动根据可见数据范围调整 Y 轴刻度
   - **图像复位**：一键恢复初始缩放范围
5. **主题切换**：右上角「明亮 / 黑暗」按钮

## 技术说明

- **无构建步骤**：直接编辑 `index.html`，刷新浏览器即可生效
- **状态管理**：全局变量（`data`, `selectedVars`, `plotType`, `timeSeriesMode`）
- **渲染模式**：`renderApp()` 全量重建 DOM，`renderChart()` 销毁并重建 ECharts 实例
- **时间解析**：双阶段解析（dayjs 严格模式 → 宽松模式），支持 20+ 格式与中文日期
- **浏览器支持**：现代浏览器（Chrome、Edge、Firefox、Safari）

## 许可证

MIT

## 致谢

- [openCode](https://github.com/anomalyco/opencode)
- [oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)
