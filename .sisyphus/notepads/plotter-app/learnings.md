# Plotter App Learnings

## Project Setup
- Vite + React + TypeScript project at `plotter-app/`
- Dependencies: echarts, xlsx, dayjs, tailwindcss
- Type definitions created at `src/types/index.ts`

## Type Contracts
```typescript
interface DataRow {
  [key: string]: string | number | Date | null;
}

interface ParsedData {
  headers: string[];
  rows: DataRow[];
  timeColumn: string;
  timeFormat: string;
}

type PlotType = 'time-series' | 'xy' | 'xyz';

interface PlotConfig {
  type: PlotType;
  xAxis?: string;
  yAxis: string[];
  zAxis?: string;
}
```

## Architecture
- Left panel: Variable selector (checkboxes for headers)
- Right panel: Plot area with mode switcher (time-series/xy/xyz)
- Data flow: File import -> Parse -> State in App -> Pass to components
- Time parsing: Must support multiple formats (ISO, custom, Excel serial, etc.)

## Conventions
- Use Tailwind CSS for styling
- ECharts for plotting (lightweight, powerful)
- xlsx library for Excel parsing
- dayjs for time manipulation

## VariableSelector Component (added 2026-05-03)
- Created \src/components/VariableSelector.tsx\ ¡ª left sidebar for variable selection
- Props: \headers\, \	imeColumn\, \selected\, \onChange\
- Features: search/filter, select-all/deselect-all, selected count badge, custom checkbox with checkmark SVG
- Time column displayed as non-selectable special row with clock icon and "Ê±¼äÖá" badge
- Styling: references existing CSS variables (\--accent\, \--text\, \--border\, etc.) for theme consistency
- Fixed width 260px, scrollable list area, empty states for no data / no search results
- React \+ TypeScript, Tailwind CSS v4 via \@import "tailwindcss"\
- Dependencies installed: react, react-dom, @vitejs/plugin-react, @types/react, @types/react-dom
- tsconfig.json: added \"jsx": "react-jsx"\, removed \"erasableSyntaxOnly"\ (incompatible with JSX)
