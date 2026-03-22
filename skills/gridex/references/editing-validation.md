# Editing & Validation

Detailed guide for all editing modes, validation, clipboard operations, and transaction management in Gridex.

## Editing Modes

### Cell Editing (Default)

Edit one cell at a time inline:

```tsx
<Gridex
  data={data}
  columns={columns}
  editing={{
    enabled: true,
    trigger: "doubleClick",  // "click" | "doubleClick" | "keypress"
    mode: "cell",
    onAfterEdit: ({ rowIndex, columnId, oldValue, newValue, row }) => {
      console.log(`Cell [${rowIndex}, ${columnId}] changed: ${oldValue} → ${newValue}`);
    },
  }}
/>
```

### Row Editing

Edit all cells in a row simultaneously:

```tsx
<Gridex
  editing={{
    enabled: true,
    mode: "row",
    onRowSave: async ({ rowIndex, changes, row }) => {
      await api.updateRow(row.id, changes);
    },
    onRowCancel: ({ rowIndex, row }) => {
      console.log("Row edit cancelled");
    },
  }}
/>
```

### Batch Editing

Accumulate changes and save all at once:

```tsx
<Gridex
  editing={{
    enabled: true,
    mode: "batch",
    onBatchSave: async (changes) => {
      await api.batchUpdate(changes);
    },
    onBatchDiscard: () => {
      console.log("All changes discarded");
    },
  }}
/>
```

Batch mode shows a toolbar with Save/Discard buttons and highlights dirty cells.

### Form Editing

Modal form for complex row editing:

```tsx
<Gridex
  editing={{
    enabled: true,
    mode: "form",
    formLayout: "twoColumn",  // "single" | "twoColumn"
    renderForm: ({ row, values, onChange, onSave, onCancel, errors }) => (
      <div>
        <input value={values.name} onChange={(e) => onChange("name", e.target.value)} />
        {errors.name && <span className="error">{errors.name}</span>}
        <button onClick={onSave}>Save</button>
        <button onClick={onCancel}>Cancel</button>
      </div>
    ),
  }}
/>
```

### Popup Editing

Dialog-based editing — similar to form but in a popup overlay:

```tsx
<Gridex
  editing={{
    enabled: true,
    mode: "popup",
    formLayout: "single",
  }}
/>
```

## Edit Triggers

| Trigger | Behavior |
|---------|----------|
| `"click"` | Single click enters edit mode |
| `"doubleClick"` | Double-click enters edit mode (default) |
| `"keypress"` | Start typing to enter edit mode on focused cell |

## Per-Column Editability

```tsx
col.accessor("name", { editable: true });
col.accessor("id", { editable: false });
col.accessor("status", { editable: (row) => row.role === "admin" }); // Conditional
```

## Per-Row Editability

```tsx
<Gridex
  editing={{
    enabled: true,
    isRowEditable: (row) => !row.locked, // Lock specific rows
  }}
/>
```

## Validation

### Synchronous Validation

```tsx
col.accessor("email", {
  editable: true,
  validate: (value) => {
    if (!value) return "Email is required";
    if (!value.includes("@")) return "Invalid email";
    return true; // Valid
  },
});
```

### Asynchronous Validation

```tsx
col.accessor("username", {
  editable: true,
  validate: async (value) => {
    const exists = await api.checkUsername(value);
    return exists ? "Username taken" : true;
  },
});
```

### Accessing Validation Errors

```tsx
const { validationErrors } = useGridex();
// validationErrors: Map<string, string> — key is "rowIndex:columnId"
```

## Edit Lifecycle Callbacks

```tsx
<Gridex
  editing={{
    enabled: true,
    // Before entering edit mode — return false to prevent
    onBeforeEdit: async ({ rowIndex, columnId, value }) => {
      return canEdit(rowIndex, columnId);
    },
    // After a cell edit is confirmed
    onAfterEdit: ({ rowIndex, columnId, oldValue, newValue, row }) => {
      setData(prev => {
        const next = [...prev];
        next[rowIndex] = { ...next[rowIndex], [columnId]: newValue };
        return next;
      });
    },
    // When edit is cancelled (Escape key)
    onCancelEdit: ({ rowIndex, columnId, value }) => {},
    // Batch save
    onSave: async (changes) => {
      await api.batchUpdate(changes);
    },
  }}
/>
```

## Custom Editors

Create custom editors by implementing the `EditorProps` interface:

```tsx
import type { EditorProps } from "gridex";

function ColorEditor({ value, onChange, onCommit, onCancel }: EditorProps) {
  return (
    <input
      type="color"
      value={value as string}
      onChange={(e) => onChange(e.target.value)}
      onBlur={onCommit}
      onKeyDown={(e) => {
        if (e.key === "Enter") onCommit();
        if (e.key === "Escape") onCancel();
      }}
      autoFocus
    />
  );
}

col.accessor("color", { editable: true, editor: ColorEditor });
```

## Built-in Editors

| Editor | Import | Usage |
|--------|--------|-------|
| `TextEditor` | Auto | Default for `text` type |
| `NumberEditor` | Auto | Default for `number` type |
| `DateEditor` | `"gridex"` | Default for `date` type |
| `DateTimeEditor` | `"gridex"` | Default for `dateTime` type |
| `TimeEditor` | `"gridex"` | Default for `time` type |
| `SelectEditor` | Auto | Default for `select` type |
| `BooleanEditor` | Auto | Default for `boolean` type |
| `PasswordEditor` | `"gridex"` | For `password` type |
| `PhoneEditor` | `"gridex"` | For `phone` type |
| `DurationEditor` | `"gridex"` | For `duration` type |
| `PercentageEditor` | `"gridex"` | For `percentage` type |
| `MaskedEditor` | `"gridex"` | For `masked` type |
| `LargeTextEditor` | `"gridex"` | Multi-line text (textarea) |
| `RichSelectEditor` | `"gridex"` | Searchable dropdown with custom rendering |
| `AutocompleteEditor` | `"gridex"` | Text input with suggestions |

## Clipboard Operations

### Copy/Cut

Enabled by default when cell range selection is active. Copies as TSV.

```tsx
<Gridex
  editing={{
    enabled: true,
    clipboardFormat: "tsv",  // "tsv" | "csv" | "html"
    enableCut: true,         // Ctrl+X / Cmd+X (default: true)
    customClipboardWrite: (data) => {
      // Override clipboard behavior
      navigator.clipboard.writeText(data);
    },
  }}
/>
```

### Paste

```tsx
<Gridex
  editing={{
    enabled: true,
    enableClipboardPaste: true,  // Ctrl+V / Cmd+V
    onBeforePaste: (data) => {
      // Return modified data or false to cancel
      return data;
    },
    processClipboardData: (cells) => {
      // Transform 2D string array before applying
      return cells.map(row => row.map(cell => cell.trim()));
    },
  }}
/>
```

## Bulk Operations

Enable Excel-like bulk cell operations on selected ranges:

```tsx
<Gridex
  editing={{
    enabled: true,
    enableBulkOperations: true,  // Ctrl+D, Ctrl+Enter, Delete
    enableFillHandle: true,       // Drag-fill handle
    onCellFill: ({ sourceCell, filledCells }) => {
      console.log(`Filled ${filledCells.length} cells from [${sourceCell.rowIndex}, ${sourceCell.colIndex}]`);
    },
  }}
  selection={{
    mode: "multiple",
    enableCellRangeSelection: true,  // Required for bulk ops
  }}
/>
```

| Shortcut | Action |
|----------|--------|
| `Ctrl+D` | Fill down — copy top cell value to all selected cells |
| `Ctrl+Enter` | Fill range — fill all selected cells with active cell value |
| `Delete` | Clear — set all selected cells to empty |
| Drag fill handle | Copy source cell value in drag direction |

## Auto-Save

Automatically save changes after a debounce period:

```tsx
<Gridex
  editing={{
    enabled: true,
    autoSave: {
      enabled: true,
      debounceMs: 2000,  // Save 2s after last edit (default: 1000)
      onSave: async (changes) => {
        await api.batchUpdate(changes);
      },
      onError: (error) => {
        toast.error("Auto-save failed");
      },
    },
  }}
/>
```

## Unsaved Changes Guard

Warn users before leaving with unsaved changes:

```tsx
<Gridex
  editing={{
    enabled: true,
    unsavedChangesGuard: true,  // Browser beforeunload warning
    onUnsavedNavigation: async () => {
      // Custom navigation guard — return false to prevent
      return window.confirm("You have unsaved changes. Leave anyway?");
    },
  }}
/>
```

## Transaction API (Undo/Redo)

```tsx
import { useTransactionAPI } from "gridex";

function MyGrid() {
  const { data, setData, undo, redo, canUndo, canRedo, history } = useTransactionAPI(initialData);

  return (
    <div>
      <button onClick={undo} disabled={!canUndo}>Undo</button>
      <button onClick={redo} disabled={!canRedo}>Redo</button>
      <Gridex
        data={data}
        columns={columns}
        editing={{
          enabled: true,
          onAfterEdit: ({ rowIndex, columnId, newValue }) => {
            setData(prev => {
              const next = [...prev];
              next[rowIndex] = { ...next[rowIndex], [columnId]: newValue };
              return next;
            });
          },
        }}
      />
    </div>
  );
}
```

## CRUD Toolbar

Built-in toolbar for row add/delete operations:

```tsx
<Gridex
  data={data}
  columns={columns}
  selection={{ mode: "multiple" }}
  toolbar={{
    showAddRow: true,
    showDeleteSelected: true,
    position: "top",  // "top" | "bottom"
    createNewRow: () => ({ id: uuid(), name: "", age: 0 }),
    newRowPosition: "bottom",  // "top" | "bottom"
    onAddRow: (newRow) => {
      setData(prev => [...prev, newRow]);
    },
    onBeforeDelete: async (rows) => {
      return window.confirm(`Delete ${rows.length} rows?`);
    },
    onDelete: (rows) => {
      setData(prev => prev.filter(r => !rows.includes(r)));
    },
    // Custom toolbar actions
    actions: [
      { id: "export", label: "Export", onClick: () => exportData(), variant: "secondary" },
      { id: "refresh", label: "Refresh", onClick: () => refetch() },
    ],
  }}
/>
```
