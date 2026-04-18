# Server-Side Data

Detailed guide for server-side data fetching, caching, grouping, tree navigation, and the dataSource API in Gridex.

## dataSource Prop

Replace `data` with `dataSource` to enable server-side mode. The grid manages sorting, filtering, and pagination state internally and calls your `fetchData` function whenever state changes.

```tsx
<Gridex
  columns={columns}
  dataSource={{
    fetchData: async ({ sorting, filters, globalFilter, pageIndex, pageSize }) => {
      const params = new URLSearchParams({
        page: String(pageIndex),
        size: String(pageSize),
        sort: JSON.stringify(sorting),
        filters: JSON.stringify(filters),
        search: globalFilter,
      });
      const res = await fetch(`/api/data?${params}`);
      const { data, totalRows } = await res.json();
      return { data, totalRows };
    },
    pageSize: 25,
    debounceMs: 300,  // Debounce filter/sort changes (default: 300)
    showFilterRow: true,
  }}
/>
```

## GridexDataSource Interface

```typescript
interface GridexDataSource<TData = any> {
  /** Server fetch function — called on every sort/filter/pagination change */
  fetchData: (request: GridexServerRequest) => Promise<GridexServerResponse<TData>>;
  /** Pagination mode (default: "offset") */
  paginationMode?: "offset" | "cursor";
  /** Initial page size (default: 10) */
  pageSize?: number;
  /** Available page size options (default: [10, 25, 50, 100]) */
  pageSizeOptions?: number[];
  /** Debounce delay in ms for filter/sort changes (default: 300) */
  debounceMs?: number;
  /** Show filter row (default: false) */
  showFilterRow?: boolean;
  /** Show global filter (default: false) */
  globalFilter?: boolean;
  /** Fetch grouped data when a group row is expanded */
  fetchGroupData?: (request: GridexServerGroupRequest) =>
    Promise<{ data: TData[]; hasMore?: boolean }>;
  /** Fetch children for a tree node (lazy loading) */
  fetchChildren?: (parentId: string) => Promise<TData[]>;
  /** Loading mode (default: "pagination") */
  loadingMode?: "pagination" | "infinite" | "viewport";
  /** Enable response caching */
  cache?: boolean | { ttl: number };
  /** Periodic auto re-fetch interval (ms) */
  revalidateMs?: number;
}

interface GridexServerRequest {
  sorting: SortingState;
  filters: ColumnFiltersState;
  globalFilter: string;
  pageIndex: number;         // Used in offset mode
  pageSize: number;
  cursor?: string;           // Used in cursor mode
}

interface GridexServerResponse<TData> {
  data: TData[];
  totalRows: number;
  nextCursor?: string;       // Required in cursor mode
  previousCursor?: string;
}
```

There is no `defaultSort` on `GridexDataSource`. Pass `sorting.defaultSort` on `<Gridex>` instead — the sorting config still applies in server-side mode and Gridex forwards the sort state to `fetchData`.

## Caching

Enable response caching to avoid refetching identical requests:

```tsx
<Gridex
  columns={columns}
  dataSource={{
    fetchData: myFetchFn,
    cache: true,                  // Cache indefinitely
    // OR
    cache: { ttl: 30000 },       // Cache for 30 seconds
  }}
/>
```

Cache is automatically invalidated when the serialized request params change.

## useDataSource Hook

Use the hook directly for more control:

```tsx
import { useDataSource } from "gridex";

function MyGrid() {
  const ds = useDataSource({
    fetchData: myFetchFn,
    pageSize: 25,
    cache: { ttl: 60000 },
  });

  return (
    <div>
      {ds.isLoading && <Spinner />}
      {ds.error && <Error message={ds.error.message} />}
      <Gridex
        data={ds.data}
        columns={columns}
        // Pass server-side state to grid
        sorting={{ state: ds.sorting, onSortingChange: ds.onSortingChange }}
        filtering={{ state: ds.filters, onFilteringChange: ds.onFiltersChange }}
        pagination={{
          state: ds.pagination,
          onPaginationChange: ds.onPaginationChange,
          pageCount: ds.pageCount,
          totalRows: ds.totalRows,
        }}
      />
      <button onClick={ds.refetch}>Refresh</button>
    </div>
  );
}
```

## Server-Side Grouping

Server-side grouping is wired through `dataSource.fetchGroupData` — Gridex calls it when a group row is expanded and passes the path of group keys. Top-level rows still come from `fetchData`.

```tsx
<Gridex
  columns={columns}
  dataSource={{
    fetchData: async (req) => fetchFlatData(req),
    fetchGroupData: async ({ groupKeys, sorting, filters }) => {
      const rows = await api.fetchChildren(groupKeys, { sorting, filters });
      return { data: rows, hasMore: false };
    },
    pageSize: 50,
  }}
  grouping={{
    groupBy: ["department"],      // Columns to group by
    showAggregation: true,
  }}
/>
```

### Server Group Request

```typescript
interface GridexServerGroupRequest {
  /** Path of expanded group keys (e.g., ["Engineering", "Frontend"]) */
  groupKeys: string[];
  /** Current sorting state */
  sorting: SortingState;
  /** Current column filter state */
  filters: ColumnFiltersState;
}
```

Return `{ data: TData[]; hasMore?: boolean }` — `data` contains the child rows for the expanded group path.

## Server-Side Tree Data

Load tree nodes lazily. For a non-dataSource tree, set `treeData.loadChildren`. In server-side (`dataSource`) mode, provide `dataSource.fetchChildren(parentId)` — Gridex calls it by row id when a tree row is expanded.

```tsx
// Standalone lazy tree (client-side data, async children)
<Gridex
  data={rootRows}
  columns={columns}
  treeData={{
    enabled: true,
    getSubRows: (row) => row.children,
    loadChildren: async (row) => {
      const res = await fetch(`/api/nodes/${row.id}/children`);
      return res.json();
    },
    defaultExpandedDepth: 0,
  }}
/>

// dataSource-driven lazy tree
<Gridex
  columns={columns}
  dataSource={{
    fetchData: async (req) => fetchFlatData(req),
    fetchChildren: async (parentId) => {
      const res = await fetch(`/api/nodes/${parentId}/children`);
      return res.json();
    },
  }}
  treeData={{ enabled: true, getSubRows: (row) => row.children }}
/>
```

## Server-Side Set Filter

Fetch unique values for a column's set filter from the server:

```tsx
col.accessor("status", {
  type: "select",
  fetchUniqueValues: async () => {
    const res = await fetch("/api/statuses");
    const statuses = await res.json();
    return statuses.map((s) => ({ value: s.name, count: s.count }));
  },
});
```

## Viewport-Based Loading

Load data based on the visible viewport (for virtual scrolling):

```tsx
<Gridex
  columns={columns}
  virtualization={{ height: 600 }}
  dataSource={{
    fetchData: async ({ pageIndex, pageSize }) => {
      // pageIndex/pageSize correspond to visible viewport rows
      return fetchViewportData(pageIndex, pageSize);
    },
    pageSize: 100,
  }}
/>
```

## Controlled Server-Side State

For full control, manage all state externally:

```tsx
function ControlledServerGrid() {
  const [sorting, setSorting] = useState<SortingState>([]);
  const [filters, setFilters] = useState<ColumnFiltersState>([]);
  const [pagination, setPagination] = useState({ pageIndex: 0, pageSize: 25 });
  const [data, setData] = useState([]);
  const [totalRows, setTotalRows] = useState(0);

  useEffect(() => {
    fetchData({ sorting, filters, ...pagination }).then(res => {
      setData(res.data);
      setTotalRows(res.totalRows);
    });
  }, [sorting, filters, pagination]);

  return (
    <Gridex
      data={data}
      columns={columns}
      sorting={{ state: sorting, onSortingChange: setSorting }}
      filtering={{ state: filters, onFilteringChange: setFilters }}
      pagination={{
        state: pagination,
        onPaginationChange: setPagination,
        pageCount: Math.ceil(totalRows / pagination.pageSize),
        totalRows,
      }}
    />
  );
}
```

## Architecture Note

When `dataSource` is provided, Gridex bypasses the intermediate `useSortingState`/`useFilteringState`/`usePaginationState` hooks entirely and uses `useDataSource` values directly. This prevents infinite re-render loops caused by object reference identity changes.

The `useDataSource` hook uses `JSON.stringify(sorting)` and template literal strings as `useEffect` dependencies instead of object references — this is intentional to avoid infinite loops while still triggering refetches on actual value changes.
