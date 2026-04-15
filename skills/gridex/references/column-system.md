# Column System

Detailed guide for defining columns, types, value pipeline, and column options in Gridex.

## createColumns

Type-safe column builder using a fluent API:

```tsx
import { createColumns } from "gridex";

type Person = { name: string; age: number; department: string; hireDate: Date };

const columns = createColumns<Person>((col) => [
  // Accessor columns — bound to data
  col.accessor("name", {
    header: "Full Name",
    type: "text",
    editable: true,
  }),
  col.accessor("age", {
    header: "Age",
    type: "number",
    size: 80,
  }),
  col.accessor("department", {
    header: "Department",
    type: "select",
    filterSelectOptions: [
      { label: "Engineering", value: "engineering" },
      { label: "Marketing", value: "marketing" },
    ],
  }),
  col.accessor("hireDate", {
    header: "Hire Date",
    type: "date",
    dateFormat: "medium",
  }),

  // Display columns — no data binding, custom render only
  col.display("actions", {
    header: "Actions",
    cell: ({ row }) => <button onClick={() => edit(row.original)}>Edit</button>,
    size: 100,
  }),

  // Group columns — visual header grouping
  col.group("personal", "Personal Info", [
    col.accessor("name", { header: "Name" }),
    col.accessor("age", { header: "Age" }),
  ]),
]);
```

## Auto-Generated Columns

Infer columns automatically from data shape:

```tsx
import { Gridex, autoGenerateColumnsFromData } from "gridex";

const columns = autoGenerateColumnsFromData(data[0]); // Infers from first row
<Gridex data={data} columns={columns} />
```

Or use the `autoGenerateColumns` prop:

```tsx
<Gridex data={data} autoGenerateColumns={{ enabled: true, inferTypes: true }} />
```

## Column Types

| Type | Filtering | Sorting | Editor | Formatter |
|------|-----------|---------|--------|-----------|
| `text` | TextFilter (contains, equals, startsWith, endsWith) | Alphabetical | TextEditor | As-is |
| `number` | NumberFilter (equals, gt, lt, between) | Numeric | NumberEditor | Locale-formatted |
| `date` | DateFilter (equals, before, after, between) | Chronological | DateEditor | Intl.DateTimeFormat |
| `dateTime` | DateFilter | Chronological | DateTimeEditor | Intl.DateTimeFormat |
| `time` | TextFilter | String sort | TimeEditor | HH:MM format |
| `boolean` | BooleanFilter (true/false toggle) | Boolean | Checkbox toggle | Checkbox display |
| `select` | SelectFilter (dropdown) | Alphabetical | SelectEditor | Label from options |
| `richSelect` | SelectFilter | Alphabetical | RichSelectEditor | Custom rendering |
| `autocomplete` | TextFilter | Alphabetical | AutocompleteEditor | As-is |
| `password` | None | None | PasswordEditor | Masked (•••) |
| `phone` | TextFilter | String | PhoneEditor | Formatted phone |
| `duration` | NumberFilter | Numeric | DurationEditor | hm/colon/minutes |
| `percentage` | NumberFilter | Numeric | PercentageEditor | XX% format |
| `currency` | NumberFilter | Numeric | NumberEditor | Locale currency |
| `bigint` | NumberFilter | Numeric | NumberEditor | Grouped digits |
| `masked` | TextFilter | String | MaskedEditor | Masked display |

## Custom Data Types

Define reusable custom types:

```tsx
import { defineDataType, getDataTypeConfig } from "gridex";

defineDataType("rating", {
  type: "number",
  valueFormatter: (v) => "⭐".repeat(v),
  editor: RatingEditor,
  filterType: "number",
});

// Use in columns
col.accessor("rating", { ...getDataTypeConfig("rating"), header: "Rating" });
```

## Value Pipeline

The data flows through a transformation pipeline:

```
Read:  row[key] → valueGetter(row) → valueFormatter(value) → cell(context)
Write: editorValue → valueSetter({ row, newValue, columnId }) → row[key]
```

### valueGetter

Extract or compute values from row data:

```tsx
col.accessor("fullName", {
  header: "Full Name",
  valueGetter: (row) => `${row.firstName} ${row.lastName}`,
});
```

### valueFormatter

Format the value for display (string output):

```tsx
col.accessor("price", {
  header: "Price",
  type: "number",
  valueFormatter: (value) => `$${value.toFixed(2)}`,
});
```

### cell

Full custom render (React component):

```tsx
col.accessor("status", {
  header: "Status",
  cell: ({ getValue }) => {
    const status = getValue();
    return <Badge color={status === "active" ? "green" : "red"}>{status}</Badge>;
  },
});
```

### valueSetter

Transform edited value before writing back:

```tsx
col.accessor("price", {
  valueSetter: ({ row, newValue, columnId }) => {
    // Return the updated row
    return { ...row, price: parseFloat(newValue), updatedAt: new Date() };
  },
});
```

## All Accessor Options

| Option | Type | Description |
|--------|------|-------------|
| `header` | `string` | Column header label |
| `type` | `GridexColumnType` | Column type (affects filtering, sorting, editing, formatting) |
| `valueGetter` | `(row) => any` | Extract/compute value from row data |
| `valueFormatter` | `(value) => string` | Format value for display |
| `cell` | `ColumnDef["cell"]` | Custom cell renderer (React) |
| `footer` | `ColumnDef["footer"]` | Custom footer renderer |
| `editable` | `boolean \| (row) => boolean` | Enable editing (can be per-row) |
| `editor` | `ComponentType<EditorProps> \| "largeText"` | Custom editor component |
| `validate` | `(value) => true \| string \| Promise<...>` | Validation (sync or async) |
| `valueSetter` | `({ row, newValue, columnId }) => unknown` | Write transformer |
| `cellStyle` | `(value, row) => CSSProperties` | Dynamic cell styling |
| `enableSorting` | `boolean` | Enable/disable sorting |
| `enableFiltering` | `boolean` | Enable/disable filtering |
| `enableResizing` | `boolean` | Enable/disable resizing |
| `enablePinning` | `boolean` | Enable/disable pinning |
| `enableHiding` | `boolean` | Enable/disable hiding |
| `size` | `number` | Default column width (px) |
| `minSize` | `number` | Minimum width |
| `maxSize` | `number` | Maximum width |
| `filterFn` | `FilterFn` | Custom filter function |
| `sortingFn` | `SortingFn` | Custom sorting function |
| `filterType` | `GridexFilterType` | Override filter UI type |
| `filterSelectOptions` | `{ label, value }[]` | Options for select filter |
| `fetchUniqueValues` | `() => Promise<{ value, count }[]>` | Server-side set filter values |
| `tooltip` | `(value, row) => ReactNode` | Custom cell tooltip |
| `headerTooltip` | `string \| ReactNode` | Header tooltip |
| `headerAnnotation` | `string \| ReactNode` | Text below header (e.g., units) |
| `enableFormulas` | `boolean` | Enable formula evaluation |
| `sparkline` | `{ type, dataField?, color?, height? }` | Mini chart config |
| `colSpan` | `number \| (row) => number` | Column spanning |
| `rowSpan` | `number \| (row, rowIndex) => number` | Row spanning |
| `align` | `"left" \| "center" \| "right" \| "justify"` | Horizontal text alignment for data cells |
| `headerAlign` | `"left" \| "center" \| "right" \| "justify"` | Horizontal text alignment for the header (independent of `align`) |
| `verticalAlign` | `"top" \| "middle" \| "bottom"` | Vertical alignment |
| `responsivePriority` | `number` | Lower = hidden first on small screens |
| `dependentColumns` | `string[]` | Re-render these columns on edit |
| `dateFormat` | `"short" \| "medium" \| "long" \| "full" \| Intl.DateTimeFormatOptions` | Date display format |
| `dateTimeFormat` | Same as dateFormat | DateTime display format |
| `formatLocale` | `string` | BCP 47 locale override (e.g., "pt-BR") |
| `dateMin` / `dateMax` | `string` | Min/max date for editors |
| `dateTimeStep` | `number` | Step in seconds for dateTime picker |
| `phoneMask` | `string` | Phone mask (default: "(###) ###-####") |
| `durationFormat` | `"hm" \| "colon" \| "minutes"` | Duration display format |
| `percentageStorageFormat` | `"decimal" \| "whole"` | How percentage is stored |
| `inputMask` | `string` | Mask pattern for masked editor |
| `richSelectOptions` | `RichSelectOption[]` | Options for rich select |
| `loadOptions` | `(query) => Promise<RichSelectOption[]>` | Async options loader |
| `richSelectMulti` | `boolean` | Enable multi-select |
| `renderOption` | `(option) => ReactNode` | Custom option renderer |
| `autocompleteSuggestions` | `string[]` | Static suggestions |
| `fetchSuggestions` | `(query) => Promise<string[]>` | Async suggestions |
| `autocompleteStrict` | `boolean` | Only accept suggestions |
| `largeTextRows` / `largeTextCols` | `number` | LargeText editor size |
| `cellRendererSelector` | `(row) => ComponentType \| null` | Dynamic cell renderer |
| `cellEditorSelector` | `(row) => ComponentType \| null` | Dynamic editor |
| `editorConfig` | `GridexEditorConfig` | Grouped editor configuration |

## Dynamic Cell Rendering

Select renderer or editor based on row data:

```tsx
col.accessor("value", {
  cellRendererSelector: (row) => {
    if (row.type === "image") return ImageCell;
    if (row.type === "link") return LinkCell;
    return null; // Default renderer
  },
  cellEditorSelector: (row) => {
    if (row.type === "select") return SelectEditor;
    return null; // Default editor
  },
});
```

## Filter Types

| Filter | UI | Operators |
|--------|----|-----------|
| `TextFilter` | Text input | contains, equals, startsWith, endsWith, notContains, notEquals |
| `NumberFilter` | Number input | equals, notEquals, greaterThan, greaterThanOrEqual, lessThan, lessThanOrEqual, between |
| `DateFilter` | Date picker | equals, before, after, between |
| `SelectFilter` | Dropdown | equals from options |
| `BooleanFilter` | Toggle | true / false |
| `SetFilter` | Checkbox list | Multi-select from unique values |
| `AdvancedFilterDialog` | Modal | Multi-condition with AND/OR |
| `GlobalFilter` | Search bar | Matches across all columns |

## Inferring Column Types

```tsx
import { inferColumnType, inferColumnTypes } from "gridex";

const type = inferColumnType(sampleValue); // Returns GridexColumnType
const types = inferColumnTypes(dataArray);  // Returns Record<string, GridexColumnType>
```

## Data Type Formatter

Get the formatter registered for a custom data type:

```tsx
import { getDataTypeFormatter } from "gridex";

const formatter = getDataTypeFormatter("rating");
// Returns the valueFormatter function registered via defineDataType
```

## Type Helpers

Convenience types to reduce generic boilerplate:

```tsx
import type { GridexColumnDef, GridexRowData, GridexCellValue } from "gridex";

// GridexColumnDef<TData> — Alias for ColumnDef<TData, any>
// Avoids needing eslint-disable for the second TanStack generic
type Columns = GridexColumnDef<Person>[];

// GridexRowData<TColumns> — Extract row data type from column definitions
type Data = GridexRowData<typeof columns>; // Infers Person

// GridexCellValue<TData, TKey> — Extract cell value type from data + key
type NameValue = GridexCellValue<Person, "name">; // string
```

## Filter Builder

Build complex filter UIs with nested AND/OR conditions:

```tsx
import {
  FilterBuilder,
  createEmptyFilterBuilderModel,
  isFilterBuilderGroup,
  evaluateFilterBuilderModel,
} from "gridex";
import type { FilterBuilderModel, FilterBuilderGroup, FilterBuilderCondition } from "gridex";

// Create an empty model with a default condition targeting a column
const model = createEmptyFilterBuilderModel("name");

// Check if an item is a group vs. a condition
if (isFilterBuilderGroup(item)) {
  // item is FilterBuilderGroup — has logic: "and" | "or" and nested items
} else {
  // item is FilterBuilderCondition — has columnId, operator, value, valueTo?
}

// Evaluate a filter builder model against table data
const matchesRow = evaluateFilterBuilderModel(model, row, columns);

// Standalone component
<FilterBuilder
  columns={columns}
  model={model}
  onChange={(newModel) => setModel(newModel)}
  onApply={(model) => applyToGrid(model)}
  showApplyButton
/>
```

### Filter Builder Types

```typescript
interface FilterBuilderModel {
  rootGroup: FilterBuilderGroup;
}

interface FilterBuilderGroup {
  id: string;
  logic: "and" | "or";
  items: Array<FilterBuilderCondition | FilterBuilderGroup>; // Recursive nesting
}

interface FilterBuilderCondition {
  id: string;
  columnId: string;
  operator: FilterOperator;
  value: string;
  valueTo?: string;  // For "between" operator
}
```

## Advanced Filter Utilities

```tsx
import {
  advancedFilterFn,        // Custom filter function for advanced filter values
  isAdvancedFilterValue,   // Type guard for AdvancedFilterValue
  isSetFilterValue,        // Type guard for SetFilterValue
  evaluateOperator,        // Evaluate a single operator against a value
  getOperatorsForType,     // Get available operators for a column type
} from "gridex";

// Get operators available for a column type
const ops = getOperatorsForType("number");
// → ["equals", "notEquals", "greaterThan", "greaterThanOrEqual", "lessThan", "lessThanOrEqual", "between"]

// Evaluate an operator
const matches = evaluateOperator("greaterThan", 42, "10"); // true
```

## Filter Utility Functions

```tsx
import { stripDiacritics, fuzzyMatch, normalizeForFilter, parseDateLocal } from "gridex";

stripDiacritics("café");              // "cafe"
fuzzyMatch("hllo", "hello");          // true (fuzzy)
normalizeForFilter("  Café ");        // "cafe" (trim + lowercase + strip diacritics)
parseDateLocal("2024-01-15");         // Date object (local timezone, no UTC shift)
```
