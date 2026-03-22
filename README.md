# gridex-skills

Agent skill for [Gridex](https://github.com/irbano-tech/gridex) — a comprehensive React data grid library with all enterprise features, built on TanStack Table v8.

## Install

```bash
npx skills add irbano-tech/gridex-skills
```

## What This Skill Does

When installed, this skill gives your AI coding agent deep knowledge of the Gridex library, including:

- Complete `<Gridex>` component API with 60+ props
- Column definition system (`createColumns`, types, value pipeline)
- All editing modes (cell, row, batch, form, popup) with validation
- Server-side data fetching (`dataSource` with caching)
- Sorting, filtering (8 filter types), and pagination
- Row selection, expansion, grouping, and tree data
- Virtualization for large datasets (100k+ rows)
- Export (CSV, Excel XML, XLSX, PDF) and import utilities
- Theme system (6 presets + custom themes via CSS variables)
- i18n (35 built-in locales)
- Advanced features: formulas, pivot tables, charts, conditional formatting
- Mantine UI integration (`gridex-mantine` package)
- Slot customization system (50+ replaceable components)
- Context API (`useGridex`) and imperative Grid API
- Event emitter system for grid lifecycle events

## When It Activates

The skill activates automatically when you ask about:

- Gridex component usage and configuration
- Data grid, table, or spreadsheet features in React
- Column definitions, sorting, filtering, pagination
- Cell editing, validation, or batch operations
- Server-side data fetching with grids
- Exporting grid data to CSV, Excel, or PDF
- Theming or internationalizing a data grid
- Mantine integration with Gridex

## Structure

```
skills/gridex/
├── SKILL.md                              # Core skill with API overview and quick start
└── references/
    ├── column-system.md                  # createColumns, types, value pipeline, filter builder, type helpers
    ├── editing-validation.md             # Editing modes, validation, transactions, clipboard
    ├── server-side.md                    # dataSource, server grouping, server tree, caching
    ├── export-import.md                  # CSV, Excel, XLSX, PDF export and import
    ├── theming-i18n.md                   # Themes, presets, CSS variables, icon pack, locales
    ├── advanced-features.md              # Formulas, pivot tables, charts, conditional formatting
    └── mantine-integration.md            # gridex-mantine: 15 editors, 21 slots, theme, icons, utilities
```

## License

MIT
