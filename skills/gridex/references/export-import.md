# Export & Import

Detailed guide for exporting grid data to CSV, Excel, XLSX, PDF, and importing from files.

## Export API

Access export functions via the `useGridex()` context hook or `exportAPI`:

```tsx
import { useGridex } from "gridex";

function ExportToolbar() {
  const { exportAPI } = useGridex();

  return (
    <div>
      <button onClick={() => exportAPI?.downloadCSV()}>CSV</button>
      <button onClick={() => exportAPI?.downloadExcel()}>Excel (.xls)</button>
      <button onClick={() => exportAPI?.downloadPDF()}>PDF</button>
      <button onClick={() => exportAPI?.copyToClipboard()}>Copy</button>
      <button onClick={() => exportAPI?.copySelectionToClipboard()}>Copy Selection</button>
      <button onClick={() => exportAPI?.importCSV(file)}>Import CSV</button>
      <button onClick={() => exportAPI?.importExcel(file)}>Import Excel</button>
    </div>
  );
}
```

`exportAPI` exposes `downloadCSV`, `downloadExcel`, `downloadPDF`, `copyToClipboard`, `copySelectionToClipboard`, `importCSV`, `importExcel`, and `exportToCSV`. XLSX is only available via the standalone `downloadXlsx`/`generateXlsx` utilities below — there is no `exportAPI.downloadXlsx`.

## CSV Export

### Basic CSV

```tsx
import { generateCSV } from "gridex";

// Generate CSV string
const csvString = generateCSV(table, {
  delimiter: ",",       // Default: ","
  includeHeaders: true, // Default: true
  selectedOnly: false,  // Export only selected rows
  visibleOnly: true,    // Only visible (non-hidden) columns
});

// Download as file
exportAPI?.downloadCSV({
  filename: "export.csv",
  delimiter: ",",
});
```

### Copy to Clipboard

`copyToClipboard` writes the grid rows as tab-separated values (Excel/Sheets compatible). It accepts optional scope options (`selectedOnly`, `visibleOnly`, `includeHeaders`, etc.) — it does **not** accept a format string.

```tsx
import { copyTextToClipboard } from "gridex";

// Default — all filtered rows, visible columns
await exportAPI?.copyToClipboard();

// Selected rows only
await exportAPI?.copyToClipboard({ selectedOnly: true });

// Or use the dedicated selection helper
await exportAPI?.copySelectionToClipboard();

// Direct utility for arbitrary text
await copyTextToClipboard(csvString);
```

## Excel XML Export

Simple Excel export using XML SpreadsheetML format:

```tsx
import { generateExcelXML, downloadExcel } from "gridex";

// Generate XML string
const xml = generateExcelXML(table);

// Download as .xls
downloadExcel(table, "export.xls");

// Via export API
exportAPI?.downloadExcel();
```

## Styled Excel Export

Excel with colors, fonts, borders, and formatting:

```tsx
import { generateStyledExcelXML, downloadStyledExcel } from "gridex";
import type { GridexExcelStyleOptions } from "gridex";

const options: GridexExcelStyleOptions = {
  sheetName: "Sales Report",
  headerStyle: {
    bold: true,
    fontColor: "#FFFFFF",
    backgroundColor: "#1A73E8",
    alignment: "center",
  },
  columnStyles: {
    amount: { numberFormat: "#,##0.00", alignment: "right" },
    status: { bold: true },
  },
  frozenRows: 1,       // Freeze header row
  frozenColumns: 0,
  autoFilter: true,    // Add AutoFilter dropdowns to header
  asExcelTable: true,  // Wrap as an Excel Table object
};

downloadStyledExcel(table, options);
```

### GridexExcelStyleOptions

```typescript
interface GridexExcelStyleOptions {
  headerStyle?: GridexExcelCellStyle;
  columnStyles?: Record<string, GridexExcelCellStyle>;
  frozenColumns?: number;
  frozenRows?: number;                   // Default: 1 (header)
  autoFilter?: boolean;                  // Default: true
  sheetName?: string;                    // Default: "Sheet1"
  protection?: {
    enabled: boolean;
    password?: string;
    editableColumns?: string[];
  };
  asExcelTable?: boolean;
  tableName?: string;                    // Default: "GridexTable"
  useWebWorker?: boolean;
}

interface GridexExcelCellStyle {
  bold?: boolean;
  italic?: boolean;
  fontColor?: string;                    // Hex, e.g. "#FF0000"
  backgroundColor?: string;              // Hex
  numberFormat?: string;                 // e.g. "#,##0.00", "yyyy-mm-dd"
  alignment?: "left" | "center" | "right";
}
```

For web-worker export use `downloadStyledExcelAsync` and set `useWebWorker: true`.

## XLSX Export

Full XLSX format with multiple sheets, formulas, and cell references:

```tsx
import { generateXlsx, downloadXlsx } from "gridex";
import type { XlsxOptions, XlsxSheet } from "gridex";

const options: XlsxOptions & { filename?: string } = {
  filename: "report.xlsx",
  sheets: [
    {
      name: "Sales Data",
      headers: [
        { id: "region", label: "Region" },
        { id: "revenue", label: "Revenue" },
      ],
      rows: salesData,            // Row objects keyed by header id
      frozenRows: 1,
      autoFilter: true,
      headerBold: true,
    },
  ],
};

// Generate as Uint8Array
const data = generateXlsx(options);

// Download as file (filename is passed alongside the options)
downloadXlsx(options);
```

`exportAPI` does **not** expose a `downloadXlsx` method — use the standalone `downloadXlsx`/`generateXlsx` utilities above and build `XlsxSheet` entries from your data.

### XlsxOptions

```typescript
interface XlsxOptions {
  sheets: XlsxSheet[];
}

interface XlsxSheet {
  name: string;
  headers: { id: string; label: string }[];
  rows: Record<string, unknown>[];
  formatters?: Record<string, (value: unknown, row: unknown) => string>;
  frozenRows?: number;
  autoFilter?: boolean;
  headerBold?: boolean;
}
```

## PDF Export

Generate PDF via an HTML print dialog:

```tsx
import { generatePDFHtml, printPDF } from "gridex";
import type { GridexPDFOptions } from "gridex";

// Generate HTML string for PDF
const html = generatePDFHtml(table, {
  title: "Sales Report",
  subtitle: "Q4 totals",
  orientation: "landscape",      // "portrait" | "landscape"
  pageSize: "A4",                // "A4" | "Letter"
  selectedOnly: false,
  currentPageOnly: false,
  columns: ["region", "revenue"],
  footer: "Generated by Gridex",
  filename: "Sales Report",
});

// Open print dialog
printPDF(table, { title: "Sales Report", orientation: "landscape" });

// Via export API
exportAPI?.downloadPDF({ title: "Report", orientation: "landscape" });
```

### GridexPDFOptions

```typescript
interface GridexPDFOptions {
  pageSize?: "A4" | "Letter";
  orientation?: "portrait" | "landscape";
  title?: string;
  subtitle?: string;
  footer?: string;
  selectedOnly?: boolean;
  currentPageOnly?: boolean;
  columns?: string[];            // Column IDs to include (default: all visible)
  filename?: string;             // Used as document title (default: "Export")
}
```

## Export Scope

All export methods support these scope options:

| Option | Description |
|--------|-------------|
| `selectedOnly: true` | Export only selected rows |
| `visibleOnly: true` | Export only visible columns (not hidden) |
| Default (no options) | All rows, all visible columns, current page (for paginated) |

## Import

The simplest import path is the `exportAPI` methods — they already have the active `table` context:

```tsx
const { exportAPI } = useGridex();

const result = await exportAPI?.importCSV(file, {
  onBeforeImport: (raw) => raw,              // Optional mutation/cancel hook
  xlsxParser: myXlsxParser,                  // Only needed for .xlsx files
});

if (result && result.errorCount > 0) {
  console.error("Validation errors:", result.errors);
} else if (result) {
  setData((prev) => [...prev, ...(result.rows as Person[])]);
}
```

### CSV Import (standalone)

The standalone helpers require a TanStack `table` instance so they can map headers to columns and run validators.

```tsx
import { parseCSVText, importCSVFile } from "gridex";

// Parse CSV text directly (no validation)
const parsed = parseCSVText(csvString);   // { headers, rows }

// Import from a File — requires the active table
const handleFileUpload = async (file: File) => {
  const result = await importCSVFile(file, table, {
    onBeforeImport: (raw) => raw,
  });

  if (result.errorCount > 0) console.error(result.errors);
  else setData((prev) => [...prev, ...(result.rows as Person[])]);
};
```

### Excel Import (standalone)

```tsx
import { importExcelFile, parseXlsxBufferAsync, parseCsvFile } from "gridex";

// Import from a File — requires the active table. Provide `xlsxParser` for real .xlsx support.
const result = await importExcelFile(file, table, {
  xlsxParser: async (buffer) => await myParser(buffer),
});

// Parse an XLSX buffer directly — returns GridexImportData
const buffer = await file.arrayBuffer();
const raw = await parseXlsxBufferAsync(buffer);   // { headers: string[], rows: string[][] }

// Parse a CSV file directly — returns GridexImportData
const rawCsv = await parseCsvFile(file);
```

### Import Pipeline (low-level)

`runImportPipeline` runs the map → coerce → validate steps on pre-parsed data. It takes the active `table` so it can look up column validators and meta.

```tsx
import { runImportPipeline } from "gridex";

const result = await runImportPipeline(
  rawData,      // GridexImportData: { headers: string[], rows: string[][] }
  table,        // TanStack Table instance (from useGridex().table)
  { onBeforeImport: (raw) => raw },
);

console.log(`Imported ${result.rowCount} rows, ${result.errorCount} errors`);
```

### GridexImportResult

```typescript
interface GridexImportResult {
  /** Number of rows successfully imported */
  rowCount: number;
  /** Number of rows that had at least one validation error */
  errorCount: number;
  /** Detailed validation errors per cell */
  errors: GridexImportValidationError[];
  /** The parsed & validated rows (rows with errors are still included) */
  rows: Record<string, unknown>[];
}

interface GridexImportValidationError {
  rowIndex: number;         // 0-based row index in the imported file (excluding header)
  columnId: string;         // Grid column id that failed validation
  value: unknown;           // Raw value that failed
  message: string;          // Validation error message
}

interface GridexImportOptions {
  onBeforeImport?: (data: GridexImportData) => GridexImportData | false;
  xlsxParser?: (buffer: ArrayBuffer) =>
    Promise<GridexImportData> | GridexImportData;
}

interface GridexImportData {
  headers: string[];
  rows: string[][];
}
```

## Export Formatters

Custom formatters per column type during export:

```tsx
import { defaultExportFormatters } from "gridex";

// Override default formatters
const customFormatters = {
  ...defaultExportFormatters,
  date: (value: Date) => value.toLocaleDateString("en-US"),
  boolean: (value: boolean) => value ? "Yes" : "No",
  currency: (value: number) => `$${value.toFixed(2)}`,
};
```
