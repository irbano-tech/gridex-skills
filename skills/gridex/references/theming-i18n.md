# Theming & Internationalization

Detailed guide for themes, CSS variables, presets, dark mode, and internationalization in Gridex.

## Theme System

Gridex uses CSS custom properties for theming. All visual properties can be overridden.

### Built-in Presets

| Preset | Description |
|--------|-------------|
| `material` | Google Material Design inspired |
| `alpine` | Clean, nature-inspired (AG Grid-like) |
| `quartz` | Modern, minimal with soft shadows |
| `minimal` | Bare-bones, no decoration |
| `corporate` | Professional, traditional |
| `highContrast` | WCAG AAA accessible |

### Using Presets

```tsx
import { Gridex } from "gridex";

// By name
<Gridex data={data} columns={columns} themePreset="alpine" />

// With dark mode
<Gridex data={data} columns={columns} themePreset="alpine" theme="dark" />
```

### Importing Presets

```tsx
import {
  PRESETS,
  materialPreset,
  alpinePreset,
  quartzPreset,
  minimalPreset,
  corporatePreset,
  highContrastPreset,
  toInlineStyle,
} from "gridex";

// Access by name
const preset = PRESETS["alpine"];

// Convert to inline style object
const style = toInlineStyle(preset.light); // or preset.dark
```

## Custom Themes

### createTheme

Build a custom theme from scratch or by extending a preset:

```tsx
import { createTheme } from "gridex";

const myTheme = createTheme({
  preset: "alpine",         // Start from a preset (optional)
  overrides: {
    "--gridex-bg": "#fafafa",
    "--gridex-header-bg": "#1a73e8",
    "--gridex-header-color": "#ffffff",
    "--gridex-border-color": "#e0e0e0",
    "--gridex-row-hover-bg": "#e8f0fe",
    "--gridex-selection-bg": "#d2e3fc",
    "--gridex-font-family": "'Inter', sans-serif",
    "--gridex-font-size": "13px",
  },
  dark: {
    "--gridex-bg": "#1e1e1e",
    "--gridex-header-bg": "#2d2d2d",
    "--gridex-header-color": "#e0e0e0",
    "--gridex-border-color": "#404040",
    "--gridex-row-hover-bg": "#2d2d2d",
  },
});

<Gridex data={data} columns={columns} theme={myTheme} />
```

### CSS Variable Map

Pass a plain object of CSS variables:

```tsx
<Gridex
  data={data}
  columns={columns}
  theme={{
    "--gridex-bg": "#ffffff",
    "--gridex-header-bg": "#f5f5f5",
    "--gridex-border-color": "#ddd",
  }}
/>
```

## CSS Variables Reference

### Colors

| Variable | Description | Default (Light) |
|----------|-------------|-----------------|
| `--gridex-bg` | Table background | `#ffffff` |
| `--gridex-color` | Text color | `#1f2937` |
| `--gridex-header-bg` | Header background | `#f9fafb` |
| `--gridex-header-color` | Header text color | `#374151` |
| `--gridex-border-color` | Border color | `#e5e7eb` |
| `--gridex-row-hover-bg` | Row hover background | `#f3f4f6` |
| `--gridex-row-stripe-bg` | Striped row background | `#f9fafb` |
| `--gridex-selection-bg` | Selected row background | `#dbeafe` |
| `--gridex-selection-color` | Selected row text | inherit |
| `--gridex-cell-focus-outline` | Focused cell outline | `#3b82f6` |
| `--gridex-editing-bg` | Editing cell background | `#eff6ff` |
| `--gridex-editing-border` | Editing cell border | `#3b82f6` |
| `--gridex-error-color` | Validation error color | `#ef4444` |
| `--gridex-error-bg` | Validation error background | `#fef2f2` |
| `--gridex-link-color` | Link color | `#2563eb` |
| `--gridex-dirty-indicator` | Dirty cell indicator color | `#f59e0b` |

### Typography

| Variable | Description | Default |
|----------|-------------|---------|
| `--gridex-font-family` | Font family | `system-ui, sans-serif` |
| `--gridex-font-size` | Base font size | `14px` |
| `--gridex-header-font-size` | Header font size | `13px` |
| `--gridex-header-font-weight` | Header font weight | `600` |

### Spacing

| Variable | Description | Default |
|----------|-------------|---------|
| `--gridex-cell-padding-x` | Cell horizontal padding | `12px` |
| `--gridex-cell-padding-y` | Cell vertical padding | `8px` |
| `--gridex-header-padding-x` | Header horizontal padding | `12px` |
| `--gridex-header-padding-y` | Header vertical padding | `10px` |
| `--gridex-border-width` | Border width | `1px` |
| `--gridex-border-radius` | Container border radius | `8px` |

### Density Presets

| Density | Cell Padding Y | Row Height |
|---------|---------------|------------|
| `compact` | `4px` | ~32px |
| `comfortable` | `8px` | ~40px |
| `spacious` | `12px` | ~48px |

```tsx
<Gridex density="compact" />   // Tight rows
<Gridex density="spacious" />  // Roomy rows
```

## Dark Mode

### Via theme prop

```tsx
<Gridex theme="dark" />
```

### Via data attribute

Gridex scopes dark mode via `[data-gridex-theme="dark"]` data attribute on the grid container. This allows multiple grids with different themes on the same page.

### System theme detection

`useTheme` is an internal hook and is **not** exported from the `gridex` package. Pass `theme="auto"` — Gridex detects system preference automatically — or roll your own `window.matchMedia` hook:

```tsx
import { useEffect, useState } from "react";

function useSystemTheme(): "light" | "dark" {
  const [theme, setTheme] = useState<"light" | "dark">("light");
  useEffect(() => {
    const mql = window.matchMedia("(prefers-color-scheme: dark)");
    const update = () => setTheme(mql.matches ? "dark" : "light");
    update();
    mql.addEventListener("change", update);
    return () => mql.removeEventListener("change", update);
  }, []);
  return theme;
}

// Or just let Gridex handle it:
<Gridex theme="auto" />
```

### CSS-only dark mode

Override CSS variables under the dark mode scope:

```css
[data-gridex-theme="dark"] {
  --gridex-bg: #1e1e1e;
  --gridex-color: #e0e0e0;
  --gridex-header-bg: #2d2d2d;
  --gridex-border-color: #404040;
}
```

### Turbopack / Lightning CSS Compatibility

Gridex's CSS Modules are fully compatible with Turbopack (Next.js). Dark mode uses a local `.darkTheme` CSS class internally — no root-level `:global()` selectors that would break Lightning CSS parsing. No `--webpack` flag is needed.

## Icon Pack

Replace all built-in icons with custom components:

```tsx
import { IconPackProvider, useIconPack, defaultIconPack } from "gridex";
import type { GridexIconPack, GridexIconProps } from "gridex";

const myIcons: Partial<GridexIconPack> = {
  sortAsc: (props) => <MyIcon name="arrow-up" {...props} />,
  sortDesc: (props) => <MyIcon name="arrow-down" {...props} />,
  filter: (props) => <MyIcon name="funnel" {...props} />,
};

<IconPackProvider icons={myIcons}>
  <Gridex data={data} columns={columns} />
</IconPackProvider>
```

### All Icon Slots (22)

Each icon receives `GridexIconProps`: `{ "aria-label"?: string; className?: string; style?: CSSProperties }`

| Slot | Purpose |
|------|---------|
| `sortAsc` | Sort ascending indicator (▲) |
| `sortDesc` | Sort descending indicator (▼) |
| `sortAscAction` | Sort ascending menu action (↑) |
| `sortDescAction` | Sort descending menu action (↓) |
| `clearSort` | Clear sort menu action (✕) |
| `filter` | Filter icon in filter row / column menu |
| `menu` | Column menu trigger icon (⋮) |
| `expand` | Row expand icon (▶) |
| `collapse` | Row collapse icon (▼) |
| `close` | Close / cancel icon (✕) |
| `check` | Checkmark icon (✓) |
| `search` | Search icon |
| `chevronLeft` | Previous page (‹) |
| `chevronRight` | Next page (›) |
| `firstPage` | Go to first page («) |
| `lastPage` | Go to last page (»)  |
| `pinLeft` | Pin column left |
| `pinRight` | Pin column right |
| `unpin` | Unpin column |
| `hideColumn` | Hide column menu action |
| `autoSize` | Auto-size column menu action (↔) |
| `groupExpand` | Group expand indicator (+) |
| `groupCollapse` | Group collapse indicator (−) |

### useIconPack Hook

Access the current icon pack from any child component:

```tsx
import { useIconPack } from "gridex";

function MyComponent() {
  const icons = useIconPack();
  const SortIcon = icons.sortAsc;
  return <SortIcon aria-label="Sorted ascending" />;
}
```

### defaultIconPack

The built-in icon pack uses Unicode symbols. Import it to extend rather than replace:

```tsx
import { defaultIconPack } from "gridex";

const myIcons = {
  ...defaultIconPack,
  sortAsc: MyCustomSortAscIcon,  // Override only specific icons
};
```

## Config Provider (Shared Defaults)

`GridexConfigProvider` lets you define shared configuration once and have all nested `<Gridex>` instances inherit it. Any prop set directly on `<Gridex>` takes precedence.

```tsx
import { GridexConfigProvider, ptBR } from "gridex";
import type { GridexConfig } from "gridex";

<GridexConfigProvider
  config={{
    theme: "dark",
    themePreset: "alpine",
    density: "compact",
    striped: true,
    locale: ptBR,
    enableFind: true,
    showRowNumbers: true,
    enableContextMenu: true,
  }}
>
  <Gridex data={data1} columns={cols1} />
  <Gridex data={data2} columns={cols2} />
  {/* Per-instance override */}
  <Gridex data={data3} columns={cols3} theme="light" density="spacious" />
</GridexConfigProvider>
```

### Shareable Props

| Prop | Type | Description |
|------|------|-------------|
| `theme` | `"light" \| "dark" \| "auto" \| CSSVariableMap` | Theme mode |
| `themePreset` | `"material" \| "alpine" \| "quartz" \| "minimal" \| "corporate" \| "highContrast"` | Visual preset |
| `density` | `"compact" \| "comfortable" \| "spacious"` | Row density |
| `striped` | `boolean` | Alternating row colors |
| `rtl` | `boolean` | Right-to-left layout |
| `locale` | `GridexTranslations \| GridexLocaleOverrides` | i18n translations |
| `iconPack` | `GridexIconPack` | Custom icon overrides |
| `tooltipConfig` | `GridexTooltipConfig` | Tooltip behavior |
| `dragConfig` | `GridexDragConfig` | Drag behavior |
| `enableContextMenu` | `boolean` | Right-click context menu |
| `enableTouchOptimization` | `boolean` | Touch-friendly interactions |
| `enableRowAnimation` | `boolean` | Row add/remove animation |
| `enableFind` | `boolean` | Find bar (Ctrl+F) |
| `enableColumnHoverHighlight` | `boolean` | Column hover highlight |
| `showRowNumbers` | `boolean` | Row number column |
| `layout` | `"normal" \| "autoHeight"` | Layout mode |
| `responsiveView` | `"table" \| "list" \| "card" \| "auto"` | Responsive view mode |
| `responsiveBreakpoint` | `number` | Auto-mode breakpoint (px) |
| `loadingRowCount` | `number` | Skeleton rows when loading |
| `defaultAlign` | `Align` | Default data cell alignment for all columns |
| `defaultHeaderAlign` | `Align` | Default header alignment for all columns (independent of `defaultAlign`) |

### Nesting Providers

Inner providers merge with outer ones — inherits unset props, overrides set ones:

```tsx
<GridexConfigProvider config={{ theme: "dark", density: "compact" }}>
  <Gridex data={data1} columns={cols1} />  {/* dark + compact */}

  <GridexConfigProvider config={{ locale: esES, striped: true }}>
    <Gridex data={data2} columns={cols2} />  {/* dark + compact + Spanish + striped */}
  </GridexConfigProvider>
</GridexConfigProvider>
```

### Dynamic Config Switching

```tsx
const [config, setConfig] = useState<GridexConfig>({
  theme: "light",
  density: "comfortable",
  locale: enUS,
});

<GridexConfigProvider config={config}>
  <button onClick={() => setConfig(prev => ({ ...prev, theme: "dark" }))}>
    Toggle Dark
  </button>
  <Gridex data={data} columns={columns} />
</GridexConfigProvider>
```

### useGridexConfig Hook

Access the current shared config from any child component:

```tsx
import { useGridexConfig } from "gridex";

function MyComponent() {
  const config = useGridexConfig();
  // config.theme, config.density, config.locale, etc.
}
```

## Internationalization (i18n)

### Built-in Locales (35)

```tsx
import { Gridex, ptBR, esES, frFR, deDE, jaJP, zhCN, koKR } from "gridex";

// Per-instance locale
<Gridex data={data} columns={columns} locale={ptBR} />

// Or set once via provider (recommended for multi-grid apps)
<GridexConfigProvider config={{ locale: ptBR }}>
  <Gridex data={data1} columns={cols1} />
  <Gridex data={data2} columns={cols2} />
</GridexConfigProvider>
```

### i18n Provider & Hook

Set locale for all nested grids, or access translations in custom components:

```tsx
import { GridexI18nProvider, useTranslations, t } from "gridex";

// Locale-only provider (also works via GridexConfigProvider)
<GridexI18nProvider translations={ptBR}>
  <Gridex data={data1} columns={cols1} />
  <MyCustomToolbar />
</GridexI18nProvider>

// Access translations in any child component
function MyCustomToolbar() {
  const translations = useTranslations();
  return <span>{t(translations.pagination.showing, { from: 1, to: 10, total: 100 })}</span>;
}
```

**Available locales:** `enUS`, `ptBR`, `esES`, `frFR`, `deDE`, `jaJP`, `zhCN`, `koKR`, `itIT`, `arSA`, `ruRU`, `hiIN`, `nlNL`, `trTR`, `plPL`, `csCZ`, `daDK`, `fiFI`, `nbNO`, `svSE`, `elGR`, `huHU`, `roRO`, `skSK`, `ukUA`, `bgBG`, `hrHR`, `srRS`, `slSI`, `ltLT`, `lvLV`, `etEE`, `thTH`, `viVN`, `idID`

### Partial Overrides

Override specific strings without replacing the entire locale:

```tsx
import { deepMerge, ptBR } from "gridex";

const myLocale = deepMerge(ptBR, {
  pagination: {
    showing: "Mostrando {from} a {to} de {total}",
  },
  filtering: {
    placeholder: "Buscar...",
  },
});

<Gridex locale={myLocale} />
```

### Template Interpolation

Locale strings support `{variable}` interpolation:

```tsx
import { t } from "gridex";

// In locale definition:
// pagination: { showing: "Showing {from} to {to} of {total}" }

const text = t(locale.pagination.showing, { from: 1, to: 10, total: 100 });
// → "Showing 1 to 10 of 100"
```

### Custom Locale

Create a complete custom locale:

```tsx
import type { GridexTranslations } from "gridex";

const myLocale: GridexTranslations = {
  pagination: {
    showing: "Showing {start}-{end} of {total}",
    noRows: "No rows",
    page: "Page {current} of {total}",
    perPage: "{count} / page",
    firstPage: "First page",
    previousPage: "Previous page",
    nextPage: "Next page",
    lastPage: "Last page",
  },
  filtering: {
    filterPlaceholder: "Filter...",
    searchPlaceholder: "Search all columns...",
    all: "All",
    yes: "Yes",
    no: "No",
    advancedFilter: "Filter",
    addCondition: "Add condition",
    removeCondition: "Remove condition",
    apply: "Apply",
    clear: "Clear",
    and: "AND",
    or: "OR",
    selectAll: "Select All",
    deselectAll: "Deselect All",
    searchValues: "Search values...",
    noMatchingValues: "No matching values",
    nValues: "{count} values selected",
    clearFilter: "Clear filter",
    findPlaceholder: "Find...",
    noMatches: "No matches",
    addGroup: "Add Group",
    operators: {
      equals: "Equals",
      notEquals: "Not equals",
      contains: "Contains",
      notContains: "Does not contain",
      startsWith: "Starts with",
      endsWith: "Ends with",
      greaterThan: "Greater than",
      greaterThanOrEqual: "Greater than or equal",
      lessThan: "Less than",
      lessThanOrEqual: "Less than or equal",
      between: "Between",
      isEmpty: "Is empty",
      isNotEmpty: "Is not empty",
    },
  },
  selection: {
    selectAll: "Select all rows",
    selectRow: "Select row {index}",
    selected: "Selected",
  },
  columns: {
    toggleVisibility: "Columns",
    hideColumn: "Hide Column",
    autoSize: "Auto-size",
  },
  sorting: {
    sortAscending: "Sorted ascending",
    sortDescending: "Sorted descending",
    clearSort: "Clear Sort",
  },
  empty: {
    noData: "No data to display",
    noDataHint: "Try adjusting your filters or adding new records.",
  },
  overlay: {
    loading: "Loading...",
    error: "An error occurred while loading data.",
    noRows: "No rows to display",
    retry: "Retry",
  },
  loading: {
    loadingMore: "Loading more...",
  },
  editing: {
    editTitle: "Edit",
    save: "Save",
    cancel: "Cancel",
    saveAll: "Save All",
    discard: "Discard",
    copyCell: "Copy Cell",
    copyRow: "Copy Row",
    edit: "Edit",
    pendingChanges: "{count} pending change(s)",
  },
  statusBar: {
    totalRows: "Total Rows",
    filteredRows: "Filtered Rows",
    selectedRows: "Selected Rows",
    sum: "Sum",
    avg: "Avg",
    min: "Min",
    max: "Max",
    count: "Count",
  },
  sidebar: {
    columns: "Columns",
    filters: "Filters",
    showAll: "Show All",
    hideAll: "Hide All",
    pinLeft: "Pin Left",
    pinRight: "Pin Right",
    unpin: "Unpin",
    filterHint: "Use column header filters to filter data.",
  },
  accessibility: {
    gridLabel: "Data grid",
    resizeColumn: "Resize {column} column",
    sortedBy: "Sorted by {column} {direction}",
    sortCleared: "Sort cleared",
    ascending: "ascending",
    descending: "descending",
    filteredTo: "Filtered to {count} of {total} rows",
    filtersCleared: "Filters cleared",
  },
};
```

### RTL Support

```tsx
<Gridex rtl locale={arSA} />
```

The `rtl` prop flips the entire grid layout, including pinned columns, scroll direction, and text alignment.

### Per-Column Locale

Override formatting locale for specific columns:

```tsx
col.accessor("price", {
  type: "currency",
  formatLocale: "de-DE",  // Format as German currency regardless of grid locale
});

col.accessor("date", {
  type: "date",
  dateFormat: "long",
  formatLocale: "ja-JP",  // Format as Japanese date
});
```
