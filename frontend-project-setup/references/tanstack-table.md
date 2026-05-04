# TanStack Table v8 (react-table) Reference

## Install
```bash
npm install @tanstack/react-table
```

## Key Concept
TanStack Table is **headless** — it manages logic (sorting, filtering, pagination) but renders **no HTML**. You control all markup and styling.

## Basic Typed Table Component
```tsx
// src/components/ui/DataTable.tsx
import {
  flexRender,
  getCoreRowModel,
  getSortedRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  useReactTable,
  type ColumnDef,
  type SortingState,
} from '@tanstack/react-table'
import { useState } from 'react'

interface DataTableProps<TData> {
  data: TData[]
  columns: ColumnDef<TData, any>[]
  pageSize?: number
}

export function DataTable<TData>({
  data,
  columns,
  pageSize = 10,
}: DataTableProps<TData>) {
  const [sorting, setSorting] = useState<SortingState>([])
  const [globalFilter, setGlobalFilter] = useState('')

  const table = useReactTable({
    data,
    columns,
    state: { sorting, globalFilter },
    onSortingChange: setSorting,
    onGlobalFilterChange: setGlobalFilter,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    initialState: { pagination: { pageSize } },
  })

  return (
    <div className="space-y-4">
      {/* Search */}
      <input
        value={globalFilter}
        onChange={(e) => setGlobalFilter(e.target.value)}
        placeholder="Search..."
        className="border rounded px-3 py-1.5 text-sm w-64"
      />

      {/* Table */}
      <div className="overflow-x-auto rounded-lg border">
        <table className="w-full text-sm">
          <thead className="bg-gray-50">
            {table.getHeaderGroups().map((hg) => (
              <tr key={hg.id}>
                {hg.headers.map((header) => (
                  <th
                    key={header.id}
                    className="px-4 py-3 text-left font-medium text-gray-600 cursor-pointer select-none"
                    onClick={header.column.getToggleSortingHandler()}
                  >
                    {flexRender(header.column.columnDef.header, header.getContext())}
                    {{ asc: ' ↑', desc: ' ↓' }[header.column.getIsSorted() as string] ?? ''}
                  </th>
                ))}
              </tr>
            ))}
          </thead>
          <tbody>
            {table.getRowModel().rows.map((row) => (
              <tr key={row.id} className="border-t hover:bg-gray-50">
                {row.getVisibleCells().map((cell) => (
                  <td key={cell.id} className="px-4 py-3">
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* Pagination */}
      <div className="flex items-center justify-between text-sm">
        <span className="text-gray-500">
          Page {table.getState().pagination.pageIndex + 1} of {table.getPageCount()}
        </span>
        <div className="flex gap-2">
          <button
            onClick={() => table.previousPage()}
            disabled={!table.getCanPreviousPage()}
            className="px-3 py-1 border rounded disabled:opacity-40"
          >
            Prev
          </button>
          <button
            onClick={() => table.nextPage()}
            disabled={!table.getCanNextPage()}
            className="px-3 py-1 border rounded disabled:opacity-40"
          >
            Next
          </button>
        </div>
      </div>
    </div>
  )
}
```

## Column Definitions
```tsx
import type { ColumnDef } from '@tanstack/react-table'

interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'user'
  createdAt: string
}

const columns: ColumnDef<User>[] = [
  {
    accessorKey: 'name',
    header: 'Name',
    cell: ({ row }) => <span className="font-medium">{row.original.name}</span>,
  },
  {
    accessorKey: 'email',
    header: 'Email',
  },
  {
    accessorKey: 'role',
    header: 'Role',
    cell: ({ getValue }) => (
      <span className={`px-2 py-0.5 rounded text-xs ${
        getValue() === 'admin' ? 'bg-purple-100 text-purple-700' : 'bg-gray-100 text-gray-600'
      }`}>
        {String(getValue())}
      </span>
    ),
  },
  {
    id: 'actions',
    header: 'Actions',
    cell: ({ row }) => (
      <button onClick={() => console.log('edit', row.original.id)}>Edit</button>
    ),
    enableSorting: false,
  },
]
```

## Usage
```tsx
<DataTable data={users} columns={columns} pageSize={15} />
```

## With antd Table (alternative — no headless setup)
Ant Design's `<Table>` component is simpler for basic use cases. Use TanStack Table when you need:
- Custom rendering and complex logic
- Virtual scrolling (add `@tanstack/react-virtual`)
- Full control over markup/styles
- Reusable headless logic across multiple table UIs

## Notes
- Column `id` is required for columns without `accessorKey`
- `enableSorting: false` disables sort on a specific column
- For row selection: add `getRowId`, `enableRowSelection`, and a checkbox column
- For server-side pagination: set `manualPagination: true` and handle `onPaginationChange`
