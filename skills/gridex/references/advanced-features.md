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
  const { pivotData, pivotColumns, isActive } = usePivotTable(salesData, {
    enabled: true,
    rowFields: ["region", "salesperson"],   // Row grouping fields
    columnField: "quarter",                  // Field whose values become pivot columns
    valueFields: [
      { field: "revenue", aggregation: "sum", header: "Total Revenue" },
      { field: "quantity", aggregation: "avg", header: "Avg Quantity" },
    ],
    showGrandTotal: true,
    showColumnTotals: false,
  });

  return <Gridex data={pivotData} columns={pivotColumns} />;
}
```

### Pivot Configuration

```typescript
interface GridexPivotConfig<TData = unknown> {
  /** Enable pivot mode */
  enabled?: boolean;
  /** Fields to use as row grouping (left-side labels) */
  rowFields: string[];
  /** Field whose unique values become pivot columns */
  columnField: string;
  /** Value fields to aggregate per pivot cell */
  valueFields: GridexPivotValueField[];
  /** Show grand total row at bottom (default: true) */
  showGrandTotal?: boolean;
  /** Show column totals (default: false) */
  showColumnTotals?: boolean;
  /** Server-side pivot callback */
  fetchPivotData?: (params: {
    rowFields: string[];
    columnField: string;
    valueFields: GridexPivotValueField[];
  }) => Promise<{ rows: TData[]; pivotColumns: string[] }>;
}

interface GridexPivotValueField {
  field: string;
  aggregation?: GridexAggregationFnName;   // "sum" | "avg" | "count" | "min" | "max" | ...
  header?: string;
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
    defaultType: "bar",         // "bar" | "line" | "pie"
    defaultLabelColumn: "month",
    defaultValueColumn: "revenue",
    height: 300,
  }}
/>
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

Switch to card or list view on small screens. View switching uses **flat props** on `<Gridex>` — not a `responsive` config object. The `responsive` prop below is a separate feature for adaptive column hiding.

```tsx
<Gridex
  data={data}
  columns={columns}
  responsiveView="auto"             // "table" | "list" | "card" | "auto"
  responsiveBreakpoint={600}        // px — auto mode switches from table to list below this
  renderCard={(row, columns) => (
    <div className="card">
      <h3>{(row as Person).name}</h3>
      <p>{(row as Person).email}</p>
      <span>${(row as Person).amount}</span>
    </div>
  )}
  renderListItem={(row, columns) => (
    <div className="list-item">
      <span>{(row as Person).name}</span>
      <span>${(row as Person).amount}</span>
    </div>
  )}
/>
```

`renderCard` / `renderListItem` receive `(row, columns)` where `columns` is an array of `{ id, label, value, formattedValue }`.

### Adaptive Responsive Column Hiding

Separate feature — hides columns progressively based on container width. Uses the `responsive` object prop and per-column `responsivePriority`.

```tsx
col.accessor("notes", { responsivePriority: 1 });   // Hidden first
col.accessor("email", { responsivePriority: 2 });   // Hidden second
col.accessor("name",  { /* no priority = never auto-hidden */ });

<Gridex
  columns={columns}
  responsive={{
    enabled: true,
    breakpoints: { sm: 480, md: 768, lg: 1024 },   // Width thresholds (px)
  }}
/>
```

Hidden column values remain accessible via a per-row expand button.

## Cell Comments / Notes

Attach notes to individual cells. `cellNotes` is keyed first by **row id** (as returned by `getRowId`), then by column id — not a flat `"rowIndex:columnId"` key. The callback is `onNoteChange(rowId, columnId, note)`.

```tsx
<Gridex
  data={data}
  columns={columns}
  getRowId={(row) => row.id}
  cellNotes={{
    "user-1": {
      name: { text: "Verified by admin", author: "John", createdAt: "2024-01-15" },
    },
    "user-3": {
      status: { text: "Needs review", title: "Important" },
    },
  }}
  onNoteChange={(rowId, columnId, note) => {
    // Save note (note is null when cleared)
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

Save and restore grid state (sort, filters, column order/visibility/sizing, pagination page size) to localStorage:

```tsx
<Gridex
  data={data}
  columns={columns}
  persistence={{
    stateKey: "my-grid-state",     // localStorage key (required)
    storage: sessionStorage,       // Optional — defaults to localStorage
  }}
/>
```

The persisted shape is a fixed set of keys — not configurable per field.

## Sidebar / Tool Panels

Built-in sidebar. Each panel is a `{ id, label, icon, component? }` object (no string shortcuts). The `columns` and `filters` built-ins live inside gridex-mantine / custom slots — pass your own panel objects when registering.

```tsx
<Gridex
  sidebar={{
    panels: [
      { id: "columns", label: "Columns", icon: "☰" },
      { id: "filters", label: "Filters", icon: "▽" },
    ],
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
      { id: "columns", label: "Columns", icon: "☰" },
      { id: "filters", label: "Filters", icon: "▽" },
      { id: "settings", label: "Settings", icon: "⚙", component: MySettingsPanel },
    ],
  }}
/>
```

## Status Bar

Bottom status bar with aggregation panels. Each panel is a `{ id, type, label?, columnId?, render? }` object — `type` is one of `"totalRows" | "filteredRows" | "selectedRows" | "sum" | "avg" | "min" | "max" | "count" | "custom"`.

```tsx
<Gridex
  statusBar={{
    panels: [
      { id: "total", type: "totalRows" },
      { id: "filtered", type: "filteredRows" },
      { id: "selected", type: "selectedRows" },
      { id: "sum-amount", type: "sum", columnId: "amount", label: "Total" },
      { id: "custom", type: "custom", render: ({ table }) => (
        <span>Rows: {table.getRowModel().rows.length}</span>
      )},
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

Replace loading, error, and empty-state overlays with custom React components. The prop is `overlays` (plural), and each entry is a component type — toggling is controlled by `loading` on `<Gridex>`.

```tsx
<Gridex
  loading={isLoading}
  overlays={{
    loading: ({ progress }) => <Spinner progress={progress} label="Loading..." />,
    error: ({ message, onRetry }) => (
      <ErrorBanner message={message} onRetry={onRetry} />
    ),
    noRows: () => <EmptyPlaceholder />,
  }}
/>
```

Import the built-in `<ProgressBar>` helper if you just want progress rendering:

```tsx
import { ProgressBar } from "gridex";
```

## Row Grouping with Aggregation

Grouping controls which columns to group by. Aggregation values (sum/avg/count/min/max) are configured separately on `summary` (or `grouping.showAggregation` to toggle visibility in group rows).

```tsx
<Gridex
  data={data}
  columns={columns}
  grouping={{
    groupBy: ["department", "team"],       // Columns to group by
    showAggregation: true,                 // Show aggregate values in group rows
    onGroupingChange: (next) => {},        // Uncontrolled; use `state` for controlled
  }}
  summary={{
    enabled: true,
    aggregations: {
      salary: "sum",
      age: "avg",
      id: "count",
    },
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
    defaultExpandedDepth: Infinity,  // 0 = collapsed, Infinity = all expanded
    loadChildren: async (row) => {   // Optional async children loader (lazy tree)
      return await api.fetchChildren(row.id);
    },
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
