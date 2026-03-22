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

```tsx
import { useTheme } from "gridex"; // or your own hook

function App() {
  const systemTheme = useTheme(); // "light" | "dark"
  return <Gridex theme={systemTheme} />;
}
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

## Internationalization (i18n)

### Built-in Locales (35)

```tsx
import { Gridex, ptBR, esES, frFR, deDE, jaJP, zhCN, koKR } from "gridex";

<Gridex data={data} columns={columns} locale={ptBR} />
```

### i18n Provider & Hook

Use translations outside the `<Gridex>` component tree:

```tsx
import { GridexI18nProvider, useTranslations, t } from "gridex";

// Wrap custom components for translation access
<GridexI18nProvider translations={ptBR}>
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
    showing: "Showing {from} to {to} of {total}",
    page: "Page",
    of: "of",
    first: "First",
    previous: "Previous",
    next: "Next",
    last: "Last",
    rowsPerPage: "Rows per page",
  },
  sorting: {
    ascending: "Sort ascending",
    descending: "Sort descending",
    clear: "Clear sort",
  },
  filtering: {
    placeholder: "Filter...",
    clear: "Clear",
    contains: "Contains",
    equals: "Equals",
    // ... all operators
  },
  selection: {
    selectAll: "Select all",
    deselectAll: "Deselect all",
    selected: "{count} selected",
  },
  editing: {
    save: "Save",
    cancel: "Cancel",
    discard: "Discard",
    // ... all editing strings
  },
  // ... other sections
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
