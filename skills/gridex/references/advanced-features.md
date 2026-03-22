# Advanced Features

Detailed guide for formulas, pivot tables, charts, conditional formatting, responsive views, and other advanced Gridex features.

## Formula Engine

Enable Excel-like formulas in cells:

```tsx
col.accessor("total", {
  header: "Total",
  type: "number",
  enableFormulas: true,
});
```

Users can type formulas starting with `=`:

```
=SUM(A1:A10)
=AVG(B1:B5)
=IF(C1>100, "High", "Low")
=MIN(D1:D20)
=MAX(E1:E20)
=COUNT(A1:A10)
```

### Supported Functions

| Function | Description |
|----------|-------------|
| `SUM(range)` | Sum of values |
| `AVG(range)` | Average of values |
| `COUNT(range)` | Count of non-empty cells |
| `MIN(range)` | Minimum value |
| `MAX(range)` | Maximum value |
| `IF(condition, trueVal, falseVal)` | Conditional |

### Cell References

```
A1     — Column A, Row 1 (absolute)
$A1    — Fixed column A
A$1    — Fixed row 1
A1:B5  — Range from A1 to B5
A:A    — Entire column A
```

### Additional Functions

| Function | Description |
|----------|-------------|
| `CONCATENATE(args...)` / `CONCAT(args...)` | Join strings |
| `AVERAGE(range)` | Alias for AVG |

### Programmatic Formula Evaluation

```tsx
import { evaluateFormula, isFormula, colLetterToIndex, indexToColLetter, parseCellRef } from "gridex";

// Check if a string is a formula
isFormula("=SUM(A1:A5)"); // true
isFormula("hello");         // false

// Evaluate with a cell value resolver
const result = evaluateFormula("=SUM(A1:A5)", (col, row) => data[row]?.[columns[col]?.id]);

// Cell reference utilities
const colIdx = colLetterToIndex("C"); // 2
const colLetter = indexToColLetter(2); // "C"
const ref = parseCellRef("B3"); // { col: 1, row: 2 }
```

The `evaluateFormula` function accepts a `CellValueResolver` callback `(col: number, row: number) => unknown` for looking up cell values, and supports circular reference detection via an internal `visited` set.

## Pivot Tables

Transform flat data into pivot table views with row/column grouping and aggregations:

```tsx
import { usePivotTable } from "gridex";

function PivotGrid() {
  const { data: pivotData, columns: pivotColumns } = usePivotTable({
    data: salesData,
    rows: ["region", "salesperson"],       // Row grouping fields
    columns: ["quarter"],                   // Column grouping fields
    values: [
      { field: "revenue", aggFn: "sum", label: "Total Revenue" },
      { field: "quantity", aggFn: "avg", label: "Avg Quantity" },
    ],
    showSubtotals: true,
    showGrandTotal: true,
  });

  return <Gridex data={pivotData} columns={pivotColumns} />;
}
```

### Pivot Configuration

```typescript
interface GridexPivotConfig {
  /** Fields to group by as rows */
  rows: string[];
  /** Fields to pivot as columns */
  columns: string[];
  /** Aggregation definitions */
  values: GridexPivotValueField[];
  /** Show subtotals per group (default: true) */
  showSubtotals?: boolean;
  /** Show grand total row (default: true) */
  showGrandTotal?: boolean;
}

interface GridexPivotValueField {
  field: string;
  aggFn: "sum" | "avg" | "count" | "min" | "max" | "distinctCount";
  label?: string;
}
```

## Chart Panel

Integrated chart visualization from grid data:

```tsx
<Gridex
  data={data}
  columns={columns}
  chart={{
    enabled: true,
    type: "bar",            // "bar" | "line" | "pie" | "area" | "scatter"
    categoryField: "month",
    valueFields: ["revenue", "expenses"],
    position: "bottom",     // "bottom" | "right" | "sidebar"
    height: 300,
  }}
/>
```

### Chart Data Hook

```tsx
import { useChartData } from "gridex";

const chartData = useChartData({
  data: data,
  categoryField: "month",
  valueFields: ["revenue", "expenses"],
  aggregation: "sum",
});
// Returns: { labels: string[], datasets: { field, values }[] }
```

### Sparkline Columns

Mini charts inside cells:

```tsx
col.accessor("trend", {
  header: "7-Day Trend",
  sparkline: {
    type: "line",          // "line" | "bar" | "area"
    dataField: "dailyValues", // Array field in row data
    color: "#3b82f6",
    height: 24,
  },
});
```

## Conditional Formatting

Apply dynamic styles to cells based on their values:

### Declarative Rules

```tsx
<Gridex
  data={data}
  columns={columns}
  conditionalFormatting={[
    {
      column: "status",
      condition: (value) => value === "overdue",
      style: { backgroundColor: "#fef2f2", color: "#dc2626", fontWeight: "bold" },
    },
    {
      column: "amount",
      condition: (value) => value > 10000,
      style: { backgroundColor: "#ecfdf5", color: "#059669" },
    },
    {
      column: "score",
      condition: (value) => value < 50,
      style: { color: "#ef4444" },
    },
  ]}
/>
```

### Built-in Formatters

```tsx
import { highlightNegative, heatMap, dataBar, evaluateConditionalFormatting } from "gridex";

// Highlight negative numbers in red
const negativeRule = highlightNegative("amount");

// Color scale from min to max
const heatMapRule = heatMap("temperature", {
  min: { value: 0, color: "#3b82f6" },    // Blue for cold
  max: { value: 100, color: "#ef4444" },   // Red for hot
});

// Data bars (horizontal bar proportional to value)
const dataBarRule = dataBar("progress", {
  color: "#10b981",
  maxValue: 100,
});

// Apply programmatically
const style = evaluateConditionalFormatting(value, row, rules);
```

### Per-Column cellStyle

```tsx
col.accessor("profit", {
  cellStyle: (value, row) => ({
    color: value >= 0 ? "#059669" : "#dc2626",
    fontWeight: value >= 0 ? "normal" : "bold",
  }),
});
```

## Responsive Views

Automatically switch to card or list view on small screens:

```tsx
<Gridex
  data={data}
  columns={columns}
  responsive={{
    breakpoint: 768,            // Switch below this width (px)
    mobileView: "card",         // "card" | "list"
    renderCard: (row) => (
      <div className="card">
        <h3>{row.original.name}</h3>
        <p>{row.original.email}</p>
        <span>${row.original.amount}</span>
      </div>
    ),
    renderListItem: (row) => (
      <div className="list-item">
        <span>{row.original.name}</span>
        <span>${row.original.amount}</span>
      </div>
    ),
  }}
/>
```

### Responsive Column Hiding

Hide columns based on container width:

```tsx
col.accessor("notes", {
  responsivePriority: 1,  // Hidden first (lowest priority)
});
col.accessor("email", {
  responsivePriority: 2,  // Hidden second
});
col.accessor("name", {
  // No responsivePriority = never auto-hidden
});
```

### Adaptive Detail Rows

On small screens, hidden columns appear in a detail row below each data row:

```tsx
<Gridex
  responsive={{
    breakpoint: 768,
    mobileView: "table",  // Keep table but show detail rows for hidden columns
  }}
/>
```

## Cell Comments / Notes

Attach notes to individual cells:

```tsx
<Gridex
  data={data}
  columns={columns}
  cellNotes={{
    "0:name": { text: "Verified by admin", author: "John", createdAt: "2024-01-15" },
    "2:status": { text: "Needs review", title: "Important" },
  }}
  onCellNoteChange={(rowIndex, columnId, note) => {
    // Save note
  }}
/>
```

## Find & Replace

Built-in Ctrl+F find bar:

```tsx
<Gridex
  data={data}
  columns={columns}
  enableFind                // Enable find bar (Ctrl+F)
/>
```

Features:
- Case-sensitive/insensitive search
- Match whole word
- Navigate matches (next/previous)
- Replace single or replace all
- Highlights matching cells

## Context Menu

Right-click context menu:

```tsx
<Gridex
  enableContextMenu
  contextMenuItems={(row, column) => [
    { label: "Copy Cell", action: () => copyCell(row, column) },
    { label: "Edit", action: () => startEdit(row, column) },
    { separator: true },
    { label: "Delete Row", action: () => deleteRow(row), variant: "danger" },
  ]}
/>
```

## State Persistence

Save and restore grid state to localStorage:

```tsx
<Gridex
  data={data}
  columns={columns}
  statePersistence={{
    key: "my-grid-state",          // localStorage key
    include: ["sorting", "filtering", "columnOrder", "columnSizing", "columnVisibility"],
    debounceMs: 500,               // Debounce save
  }}
/>
```

## Multi-Grid Synchronization

Synchronize state across multiple grid instances:

```tsx
import { useGridSync } from "gridex";

function Dashboard() {
  const sync = useGridSync({
    syncSorting: true,
    syncFiltering: true,
    syncSelection: false,
  });

  return (
    <div>
      <Gridex data={salesData} columns={salesColumns} {...sync.getProps("sales")} />
      <Gridex data={inventoryData} columns={inventoryColumns} {...sync.getProps("inventory")} />
    </div>
  );
}
```

## Sidebar / Tool Panels

Built-in sidebar with configurable panels:

```tsx
<Gridex
  sidebar={{
    panels: ["columns", "filters"],   // Built-in panels
    defaultPanel: "columns",
    position: "right",                 // "left" | "right"
    width: 300,
  }}
/>
```

### Custom Sidebar Panels

```tsx
<Gridex
  sidebar={{
    panels: [
      "columns",
      "filters",
      {
        id: "settings",
        label: "Settings",
        icon: SettingsIcon,
        component: MySettingsPanel,
      },
    ],
  }}
/>
```

## Status Bar

Bottom status bar with aggregation panels:

```tsx
<Gridex
  statusBar={{
    panels: [
      "rowCount",          // "Rows: 1,234"
      "selectedCount",     // "Selected: 5"
      "aggregation",       // Shows sum/avg/count of selected cells
      {
        id: "custom",
        component: ({ data }) => <span>Custom: {data.length}</span>,
      },
    ],
  }}
/>
```

## Column Stretching

Stretch columns to fill the container width:

```tsx
<Gridex
  columnConfig={{
    enableStretching: true,
    stretchMode: "all",     // "all" — stretch all columns proportionally
                            // "last" — only stretch the last column
                            // "fit" — stretch to fit, no horizontal scroll
  }}
/>
```

## Overlay / Loading

Custom loading overlay:

```tsx
<Gridex
  overlay={{
    loading: isLoading,
    text: "Loading data...",
    progress: 65,           // Optional progress percentage
  }}
/>
```

## Row Grouping with Aggregation

```tsx
<Gridex
  data={data}
  columns={columns}
  grouping={{
    enabled: true,
    defaultGrouping: ["department", "team"],
    aggregations: {
      salary: "sum",
      age: "avg",
      id: "count",
    },
    expandedByDefault: true,
    showGroupCount: true,
  }}
/>
```

### Aggregation Functions

| Function | Description |
|----------|-------------|
| `sum` | Sum of values |
| `avg` | Average of values |
| `count` | Count of rows |
| `min` | Minimum value |
| `max` | Maximum value |
| `distinctCount` | Count of unique values |

## Tree Data

Hierarchical data with expand/collapse:

```tsx
<Gridex
  data={treeData}
  columns={columns}
  treeData={{
    enabled: true,
    getSubRows: (row) => row.children,
    defaultExpanded: true,          // Expand all by default
    expandOnClick: true,            // Click row to expand (vs toggle button only)
    indentSize: 24,                 // Indent per level (px)
  }}
/>
```

## Keyboard Navigation

Gridex supports full keyboard navigation via roving tabindex:

| Key | Action |
|-----|--------|
| `Arrow keys` | Navigate cells |
| `Tab` / `Shift+Tab` | Move to next/previous cell |
| `Enter` | Start editing / Confirm edit |
| `Escape` | Cancel editing |
| `Space` | Toggle selection (on checkbox) |
| `Ctrl+A` | Select all |
| `Ctrl+C` | Copy selected cells |
| `Ctrl+V` | Paste (when enabled) |
| `Ctrl+X` | Cut (when enabled) |
| `Ctrl+Z` | Undo (with transaction API) |
| `Ctrl+Shift+Z` | Redo |
| `Ctrl+D` | Fill down (bulk operations) |
| `Ctrl+F` | Find bar |
| `Delete` | Clear selected cells |
| `Home` / `End` | First / Last cell in row |
| `Ctrl+Home` / `Ctrl+End` | First / Last cell in grid |
| `Page Up` / `Page Down` | Previous / Next page |

## Accessibility

Gridex includes comprehensive ARIA support:

- `role="grid"` on the table container
- `role="row"`, `role="columnheader"`, `role="gridcell"` on elements
- `aria-sort` on sortable headers
- `aria-selected` on selected rows/cells
- `aria-expanded` on expandable rows
- `aria-colindex`, `aria-rowindex` for virtual scrolling
- `LiveRegion` component announces sort/filter/selection changes to screen readers
- Roving tabindex for keyboard navigation
- Focus management during editing

## Slot Customization

Replace any built-in component:

```tsx
<Gridex
  slots={{
    // Cell rendering
    cell: MyCell,
    headerCell: MyHeaderCell,
    row: MyRow,

    // UI components
    pagination: MyPagination,
    emptyState: MyEmptyState,
    loadingState: MyLoadingState,
    columnVisibilityToggle: MyColumnToggle,

    // Editing
    popupEditDialog: MyEditDialog,
    formEditPanel: MyFormPanel,
    batchEditToolbar: MyBatchToolbar,

    // Filters
    advancedFilterDialog: MyFilterDialog,
    filterBuilder: MyFilterBuilder,

    // Advanced
    sidebar: MySidebar,
    statusBar: MyStatusBar,
    contextMenu: MyContextMenu,
    columnMenu: MyColumnMenu,
    findBar: MyFindBar,
    tooltip: MyTooltip,
    cellComment: MyCellComment,
    cardView: MyCardView,
    listView: MyListView,
    detailPanel: MyDetailPanel,
  }}
/>
```

Each slot receives typed props (e.g., `CellSlotProps`, `PaginationSlotProps`) with all the context needed to render.
