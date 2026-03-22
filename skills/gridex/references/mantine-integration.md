# Mantine Integration

Comprehensive guide for the `gridex-mantine` package — Mantine UI integration for Gridex with 15 editors, 21 slot components, theme utilities, icon pack, and utility functions.

## Installation

```bash
npm install gridex-mantine @mantine/core @mantine/dates
# Optional for icons:
npm install @tabler/icons-react
```

**Peer dependencies:** `gridex >= 0.5.0`, `@mantine/core >= 8.0.0`, `@mantine/dates >= 8.0.0` (optional), `@tabler/icons-react >= 3.0.0` (optional)

## MantineGridex Component

Drop-in replacement for `<Gridex>` with full Mantine integration:

```tsx
import { MantineProvider } from "@mantine/core";
import { MantineGridex } from "gridex-mantine";
import type { MantineGridexProps } from "gridex-mantine";

<MantineProvider>
  <MantineGridex
    data={data}
    columns={columns}
    themeOverrides={{                    // Optional CSS variable overrides
      "--gridex-border-radius": "4px",
      "--gridex-font-size": "12px",
    }}
  />
</MantineProvider>
```

**What MantineGridex does automatically:**
1. Detects light/dark mode from `MantineProvider` via `useMantineGridexTheme`
2. Merges all 21 Mantine slot components (user slots take precedence)
3. Maps column types to Mantine editors (only if column has no custom `meta.editor`)
4. Applies the Tabler icon pack (user icons take precedence)
5. Maps 53 CSS variables to Mantine design tokens

## Theme System

### mantineThemePreset

Raw CSS variable mapping object — maps all Gridex variables to Mantine tokens:

```tsx
import { mantineThemePreset } from "gridex-mantine";
// Returns: Record<string, string> with 53 CSS variable mappings
```

### createMantineTheme

Build a theme style object with optional overrides:

```tsx
import { createMantineTheme } from "gridex-mantine";

const style = createMantineTheme({
  "--gridex-border-radius": "4px",
  "--gridex-font-size": "12px",
});
// Returns: CSSProperties object with all 53 variables + your overrides
```

### useMantineGridexTheme

Hook that integrates with MantineProvider's color scheme:

```tsx
import { useMantineGridexTheme } from "gridex-mantine";

function MyGrid() {
  const { theme, style, colorScheme } = useMantineGridexTheme({
    "--gridex-font-size": "13px",  // Optional overrides
  });

  // theme: "light" | "dark" (resolved from Mantine)
  // style: CSSProperties with all CSS variable mappings
  // colorScheme: "light" | "dark" | "auto" (raw from Mantine)

  return <Gridex theme={theme} style={style} />;
}
```

**Color scheme detection:** `"auto"` detects system preference via `window.matchMedia("(prefers-color-scheme: dark)")`.

### CSS Variable Mappings (53 total)

| Gridex Variable | Mantine Token |
|----------------|---------------|
| `--gridex-bg` | `var(--mantine-color-body)` |
| `--gridex-border-color` | `var(--mantine-color-default-border)` |
| `--gridex-border-radius` | `var(--mantine-radius-md)` |
| `--gridex-header-bg` | `var(--mantine-color-gray-light)` |
| `--gridex-header-text` | `var(--mantine-color-text)` |
| `--gridex-header-hover-bg` | `var(--mantine-color-gray-light-hover)` |
| `--gridex-header-font-weight` | `600` |
| `--gridex-cell-text` | `var(--mantine-color-text)` |
| `--gridex-cell-text-secondary` | `var(--mantine-color-dimmed)` |
| `--gridex-row-hover-bg` | `var(--mantine-color-gray-light-hover)` |
| `--gridex-row-striped-bg` | `var(--mantine-color-gray-light)` |
| `--gridex-selected-row-bg` | `var(--mantine-primary-color-light)` |
| `--gridex-selected-row-hover-bg` | `var(--mantine-primary-color-light-hover)` |
| `--gridex-focus-ring-color` | `var(--mantine-primary-color-filled)` |
| `--gridex-empty-text` | `var(--mantine-color-dimmed)` |
| `--gridex-footer-bg` | `var(--mantine-color-gray-light)` |
| `--gridex-skeleton-bg` | `var(--mantine-color-default-border)` |
| `--gridex-skeleton-shine` | `var(--mantine-color-gray-light)` |
| `--gridex-sort-indicator-color` | `var(--mantine-color-dimmed)` |
| `--gridex-sort-indicator-active-color` | `var(--mantine-primary-color-filled)` |
| `--gridex-sort-index-color` | `var(--mantine-color-dimmed)` |
| `--gridex-filter-input-bg` | `var(--mantine-color-body)` |
| `--gridex-resize-handle-color` | `var(--mantine-color-default-border)` |
| `--gridex-cell-padding-x` | `var(--mantine-spacing-md)` |
| `--gridex-cell-padding-y` | `var(--mantine-spacing-xs)` |
| `--gridex-font-family` | `var(--mantine-font-family)` |
| `--gridex-font-size` | `var(--mantine-font-size-sm)` |
| `--gridex-cell-disabled-bg` | `var(--mantine-color-gray-light)` |
| `--gridex-cell-error-border` | `var(--mantine-color-error)` |
| `--gridex-cell-error-bg` | `var(--mantine-color-red-light)` |
| `--gridex-unread-accent` | `var(--mantine-primary-color-filled)` |
| `--gridex-transition-speed` | `0.15s` |
| `--gridex-transition-easing` | `ease-out` |
| `--gridex-range-bg` | `var(--mantine-primary-color-light)` |
| `--gridex-editor-bg` | `var(--mantine-color-body)` |
| `--gridex-dirty-accent` | `var(--mantine-color-yellow-filled)` |

## Icon Pack

22 Tabler SVG icons for all Gridex icon slots:

```tsx
import { mantineIconPack } from "gridex-mantine";
```

| Slot | Tabler Icon | Size |
|------|-------------|------|
| `sortAsc` | IconSortAscending | 16px |
| `sortDesc` | IconSortDescending | 16px |
| `sortAscAction` | IconArrowUp | 16px |
| `sortDescAction` | IconArrowDown | 16px |
| `clearSort` | IconX | 16px |
| `filter` | IconFilter | 16px |
| `menu` | IconDotsVertical | 16px |
| `expand` | IconChevronRight | 14px |
| `collapse` | IconChevronDown | 14px |
| `close` | IconX | 16px |
| `check` | IconCheck | 16px |
| `search` | IconSearch | 16px |
| `chevronLeft` | IconChevronLeft | 16px |
| `chevronRight` | IconChevronRight | 16px |
| `firstPage` | IconChevronsLeft | 16px |
| `lastPage` | IconChevronsRight | 16px |
| `pinLeft` | IconLayoutSidebarLeftCollapse | 16px |
| `pinRight` | IconLayoutSidebarRightCollapse | 16px |
| `unpin` | IconPinnedOff | 16px |
| `hideColumn` | IconEyeOff | 16px |
| `autoSize` | IconArrowsHorizontal | 16px |
| `groupExpand` | IconPlus | 14px |
| `groupCollapse` | IconMinus | 14px |

## Slot Components (21 total)

### Convenience Objects

Import grouped slot collections for selective integration:

```tsx
import {
  mantineSlots,             // All 21 slots combined
  mantineStructuralSlots,   // pagination, empty, loading, toolbar
  mantineInteractiveSlots,  // columnFilter, columnVisibility, advancedFilterDialog
  mantineOverlaySlots,      // popupEditDialog, formEditPanel, sidebar, detailPanel, findBar, tooltip, cellComment
  mantineMenuSlots,         // contextMenu, columnMenu, statusBar, batchEditToolbar, filterBuilder
  mantineViewSlots,         // cardView, listView
} from "gridex-mantine";
```

### Structural Slots

**MantinePagination** — Mantine Pagination + page size Select
- Props: `{pageIndex, pageCount, pageSize, totalRows, goToPage, setPageSize}`
- Page size options: 5, 10, 20, 50, 100

**MantineEmptyState** — Centered "No data to display" with hint text
- Props: `{columnCount}`

**MantineLoadingState** — 6 skeleton rows with Mantine Skeleton
- Props: `{columnCount}`

**MantineToolbar** — TextInput search bar with global filter
- Props: `{table}`

### Interactive Slots

**MantineColumnFilter** — Type-aware column filter input
- Props: `{columnId, filterValue, onFilterChange, filterType}`
- Renders: Select for "select"/"boolean", NumberInput for "number", DateInput for "date", TextInput for "text"

**MantineColumnVisibility** — Menu with Checkbox items for column show/hide
- Props: `{columns, onToggle}`

**MantineAdvancedFilterDialog** — Modal with multi-condition filter builder
- Props: `{columnId, columnType, currentValue, onApply, onClose}`
- Supports AND/OR logic, "between" operator with value + valueTo

### Overlay Slots

**MantinePopupEditDialog** — Modal form editor for popup editing mode
- Props: `{editingRow, columns, formLayout?, renderForm?, onSave, onCancel, onUpdateValue, errors?, rowData}`
- Supports "single" and "twoColumn" form layouts
- Custom `renderForm` support, auto-focuses first input

**MantineFormEditPanel** — Inline form editor rendered as table row
- Props: Same as PopupEditDialog but renders inside `<tr><td colSpan>`

**MantineSidebar** — Paper with Tabs for tool panels
- Props: `{config, table}`
- Built-in panels: "columns" (visibility checkboxes), "filters"
- Custom panels via `config.panels` with custom components

**MantineDetailPanel** — Expansion detail wrapper
- Props: `{columnCount, children}`

**MantineFindBar** — Find/replace bar with navigation
- Props: `{findReplace}`
- Features: match count Badge, next/prev navigation, case sensitivity Switch
- Keys: Enter/Shift+Enter for next/prev, Escape to close

**MantineTooltip** — Cell tooltip using Mantine Tooltip
- Props: `{visible, content, anchorRect?, id?, mousePos?, interactive?, onMouseEnter?, onMouseLeave?}`
- Positioned via mousePos or anchorRect

**MantineCellComment** — Cell note/comment with Popover
- Props: `{comment?, note?, rowId, columnId, onCommentChange?, onNoteChange?, renderNoteEditor?}`
- Blue corner triangle indicator, edit mode with Textarea
- Notes support: text, title, author, createdAt metadata

### Menu Slots

**MantineContextMenu** — Right-click context menu
- Props: `{items, cellValue, rowValues, children}`
- Default items: "Copy Cell", "Copy Row"
- Custom items appended with divider

**MantineColumnMenu** — Column header menu
- Props: `{column, onAutoSize}`
- Features: Sort asc/desc, Pin left/right, Hide, Auto-size

**MantineStatusBar** — Bottom status bar with Badge panels
- Props: `{table, config}`
- Panel types: `"totalRows"`, `"filteredRows"`, `"selectedRows"`, `"sum"` / `"avg"` / `"min"` / `"max"` / `"count"` (with columnId), `"custom"`

**MantineBatchEditToolbar** — Batch edit save/discard toolbar
- Props: `{changeCount, onSave, onDiscard}`
- Shows Badge with pending change count + Save All + Discard buttons
- Hidden when changeCount === 0

**MantineFilterBuilder** — Nested group/condition filter builder
- Props: `{columns, model?, onChange?, onApply?, showApplyButton?, className?}`
- Supports nested groups, AND/OR toggles, add/remove conditions & groups

### View Slots

**MantineCardView** — Responsive card grid layout
- Props: `{table, onRowClick?, renderCard?}`
- SimpleGrid: 1 column base, 2 at sm, 3 at lg
- Selected state: blue outline + "Selected" Badge

**MantineListView** — Responsive list layout
- Props: `{table, onRowClick?, renderListItem?}`
- Stack of Paper items, selected state: blue outline + Badge

## Companion Components

Not slot overrides — standalone helper components:

**MantineSelectionCheckbox** — Mantine Checkbox with indeterminate support
```tsx
import { MantineSelectionCheckbox } from "gridex-mantine";
<MantineSelectionCheckbox checked={true} indeterminate={false} onChange={handler} />
```

**MantineSortIndicator** — Sort direction indicator with multi-sort index
```tsx
import { MantineSortIndicator } from "gridex-mantine";
<MantineSortIndicator direction="asc" sortIndex={1} isMultiSort={true} />
```

**MantineDensitySelector** — Density toggle using SegmentedControl
```tsx
import { MantineDensitySelector } from "gridex-mantine";
<MantineDensitySelector value="comfortable" onChange={setDensity} label="Density" />
// Options: "Compact", "Standard", "Comfortable"
```

## Editors (15 total)

### Editor Collections

```tsx
import {
  mantineAllEditors,           // All 15 editors (type → component map)
  mantineCoreEditors,          // 7 core editors
  mantineSpecializedEditors,   // 8 specialized editors
  mantineEditorMap,            // Same as mantineAllEditors
} from "gridex-mantine";
```

### Column Type → Editor Mapping

| Type | Mantine Editor | Storage Type |
|------|---------------|--------------|
| `text` | MantineTextEditor | string |
| `number` | MantineNumberEditor | number |
| `date` | MantineDateEditor | Date/string "YYYY-MM-DD" |
| `dateTime` | MantineDateTimeEditor | Date |
| `time` | MantineTimeEditor | string "HH:mm" |
| `select` | MantineSelectEditor | string |
| `boolean` | MantineBooleanEditor | boolean |
| `password` | MantinePasswordEditor | string |
| `largeText` | MantineLargeTextEditor | string |
| `phone` | MantinePhoneEditor | string (digits only) |
| `percentage` | MantinePercentageEditor | number (0-100 or 0-1) |
| `duration` | MantineDurationEditor | number (minutes) |
| `masked` | MantineMaskedEditor | string (formatted) |
| `autocomplete` | MantineAutocompleteEditor | string |
| `richSelect` | MantineRichSelectEditor | string or comma-separated |

### Core Editors (7)

**MantineTextEditor** — TextInput with variant="filled", size="xs"
- Keys: Enter/Shift+Enter (navigate down/up), Tab/Shift+Tab (next/prev), Escape (cancel)

**MantineNumberEditor** — NumberInput, hideControls, variant="filled"
- Converts null/"" to null, numbers to Number

**MantineDateEditor** — DatePickerInput with auto-opening Popover
- Extra props: `min?: string | Date`, `max?: string | Date`
- Parses "YYYY-MM-DD" and ISO strings, auto-confirms on selection

**MantineDateTimeEditor** — DateTimePicker with Popover
- Extra props: `min?: string | Date`, `max?: string | Date`
- Auto-confirms on popover close

**MantineTimeEditor** — TimeInput
- Converts to string time format

**MantineSelectEditor** — Select dropdown
- Extra props: `options?: { label: string; value: string }[]`
- Auto-confirms on selection

**MantineBooleanEditor** — Centered Checkbox
- Auto-confirms on change (toggles immediately)

### Specialized Editors (8)

**MantinePasswordEditor** — PasswordInput, autoComplete="off"

**MantineLargeTextEditor** — Textarea in Popover
- Extra props: `largeTextRows?: number` (default: 4), `largeTextCols?: number` (default: 40)
- Keys: Ctrl+Enter/Cmd+Enter to save, Escape to cancel

**MantinePhoneEditor** — Masked phone input
- Extra props: `mask?: string` (default: `"(###) ###-####"`, `#` = digit)
- Stores raw digits, displays formatted

**MantinePercentageEditor** — NumberInput with % suffix
- Extra props: `storageFormat?: "decimal" | "whole"` (default: "whole")
- Displays 0-100, stores as 0-100 (whole) or 0-1 (decimal)

**MantineDurationEditor** — Duration input with format toggle
- Extra props: `format?: "hm" | "colon" | "minutes"` (default: "hm")
- Stores as minutes (number)

**MantineMaskedEditor** — Masked text input
- Extra props: `mask?: string` (default: "")
- Mask patterns: `#` (digit), `A` (letter), `*` (alphanumeric)

**MantineAutocompleteEditor** — Autocomplete with async suggestions
- Extra props: `autocompleteSuggestions?: string[]`, `fetchSuggestions?: (query: string) => Promise<string[]>`, `autocompleteStrict?: boolean`
- 300ms debounced async fetch, strict mode validates on Tab/Enter

**MantineRichSelectEditor** — Searchable dropdown with custom rendering
- Extra props: `richSelectOptions?: MantineRichSelectOption[]`, `richSelectMulti?: boolean`, `loadOptions?: (query: string) => Promise<MantineRichSelectOption[]>`, `renderOption?: (option) => ReactNode`
- `MantineRichSelectOption`: `{value: string, label: string}`
- Multi mode returns comma-separated values

### Editor Styling Pattern

All Mantine editors use:
- `variant="filled"`, `size="xs"`
- Border: `2px solid var(--mantine-primary-color-filled)`
- Font inherits Gridex CSS variables
- Error text at 11px font size

## Utility Functions

### Duration Utilities

```tsx
import { parseDuration, formatDuration } from "gridex-mantine";

// Parse various duration formats to minutes
parseDuration("2h 30m");  // 150
parseDuration("2:30");     // 150
parseDuration("150");      // 150
parseDuration("invalid");  // null

// Format minutes to display string
formatDuration(150, "hm");      // "2h 30m"
formatDuration(150, "colon");   // "2:30"
formatDuration(150, "minutes"); // "150"
formatDuration(null, "hm");     // ""
```

### Mask Utilities

```tsx
import { applyGenericMask, stripMask } from "gridex-mantine";

// Apply mask pattern: # = digit, A = letter, * = alphanumeric
applyGenericMask("AB1234", "AA-####");   // "AB-1234"
applyGenericMask("12345", "(###) ##");   // "(123) 45"

// Strip mask, keeping only matched characters
stripMask("AB-1234", "AA-####");   // "AB1234"
stripMask("(123) 45", "(###) ##"); // "12345"
```

## Column Processing

MantineGridex automatically maps column types to Mantine editors:

```typescript
// For each column:
// 1. Check if meta.type exists
// 2. Check if meta.editor does NOT exist (user editors take precedence)
// 3. Look up mantineEditorMap[meta.type]
// 4. If found, inject as meta.editor

// This means:
col.accessor("name", { type: "text" });
// → automatically gets MantineTextEditor

col.accessor("name", { type: "text", editor: MyCustomEditor });
// → keeps MyCustomEditor (user takes precedence)
```

## Full Example

```tsx
import { MantineProvider } from "@mantine/core";
import { MantineGridex, MantineDensitySelector } from "gridex-mantine";
import { createColumns } from "gridex";
import "@mantine/core/styles.css";
import "@mantine/dates/styles.css";
import "gridex/dist/index.css";

type Product = { name: string; price: number; date: Date; active: boolean };

const columns = createColumns<Product>((col) => [
  col.accessor("name", { header: "Name", editable: true }),
  col.accessor("price", { header: "Price", type: "number", editable: true }),
  col.accessor("date", { header: "Date", type: "date", dateFormat: "medium" }),
  col.accessor("active", { header: "Active", type: "boolean", editable: true }),
]);

function App() {
  const [data, setData] = useState<Product[]>(initialData);
  const [density, setDensity] = useState<GridexDensity>("comfortable");

  return (
    <MantineProvider>
      <MantineDensitySelector value={density} onChange={setDensity} />
      <MantineGridex
        data={data}
        columns={columns}
        density={density}
        editing={{
          enabled: true,
          trigger: "doubleClick",
          onAfterEdit: ({ rowIndex, columnId, newValue }) => {
            setData(prev => {
              const next = [...prev];
              next[rowIndex] = { ...next[rowIndex], [columnId]: newValue };
              return next;
            });
          },
        }}
        filtering={{ showFilterRow: true, globalFilter: true }}
        pagination={{ pageSize: 10 }}
        selection={{ mode: "multiple" }}
        sidebar={{ panels: ["columns", "filters"] }}
        statusBar={{ panels: ["totalRows", "selectedRows"] }}
        enableContextMenu
        enableFind
        striped
      />
    </MantineProvider>
  );
}
```
