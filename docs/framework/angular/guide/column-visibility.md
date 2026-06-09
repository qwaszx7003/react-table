---
title: Column Visibility (Angular) Guide
---

## Examples

Want to skip to the implementation? Check out these Angular examples:

- [Column Visibility](../examples/column-visibility)
### Angular Setup

```ts
import { signal } from '@angular/core'
import { injectTable, tableFeatures, columnVisibilityFeature } from '@tanstack/angular-table'

const features = tableFeatures({ columnVisibilityFeature })

export class App {
  readonly data = signal(defaultData)

  readonly table = injectTable(() => ({
    features,
    rowModels: {},
    columns,
    data: this.data(),
  }))
}
```

## Column Visibility (Angular) Guide

The column visibility feature allows table columns to be hidden or shown dynamically. In v9, add `columnVisibilityFeature` to your `features` to enable this. There is a dedicated `columnVisibility` state and APIs for managing column visibility dynamically.

### Column Visibility State

The `columnVisibility` state is a map of column IDs to boolean values. A column will be hidden if its ID is present in the map and the value is `false`. If the column ID is not present in the map, or the value is `true`, the column will be shown.

```ts
import { injectTable, tableFeatures, columnVisibilityFeature } from '@tanstack/angular-table'

readonly columnVisibility = signal({
  columnId1: true,
  columnId2: false, // hide this column by default
  columnId3: true,
})

readonly table = injectTable(() => ({
  features: tableFeatures({ columnVisibilityFeature }),
  rowModels: {},
  //...
  state: {
    columnVisibility: this.columnVisibility(),
    //...
  },
  onColumnVisibilityChange: (updater) =>
    typeof updater === 'function'
      ? this.columnVisibility.update(updater)
      : this.columnVisibility.set(updater),
})
```

Alternatively, if you don't need to manage the column visibility state outside of the table, you can still set the initial default column visibility state using the `initialState` option.

> **Note**: If `columnVisibility` is provided to both `initialState` and `state`, the `state` initialization will take precedence and `initialState` will be ignored. Do not provide `columnVisibility` to both `initialState` and `state`, only one or the other.

```ts
readonly table = injectTable(() => ({
  features: tableFeatures({ columnVisibilityFeature }),
  rowModels: {},
  //...
  initialState: {
    columnVisibility: {
      columnId1: true,
      columnId2: false, // hide this column by default
      columnId3: true,
    },
    //...
  },
})
```

### Disable Hiding Columns

By default, all columns can be hidden or shown. If you want to prevent certain columns from being hidden, you set the `enableHiding` column option to `false` for those columns.

```ts
const columns = [
  {
    header: 'ID',
    accessorKey: 'id',
    enableHiding: false, // disable hiding for this column
  },
  {
    header: 'Name',
    accessor: 'name', // can be hidden
  },
];
```

### Column Visibility Toggle APIs

There are several column API methods that are useful for rendering column visibility toggles in the UI.

- `column.getCanHide` - Useful for disabling the visibility toggle for a column that has `enableHiding` set to `false`.
- `column.getIsVisible` - Useful for setting the initial state of the visibility toggle.
- `column.toggleVisibility` - Useful for toggling the visibility of a column.
- `column.getToggleVisibilityHandler` - Shortcut for hooking up the `column.toggleVisibility` method to a UI event handler.

```html
@for (column of table.getAllLeafColumns(); track column.id) {
  <label>
    <input
      type="checkbox"
      [checked]="column.getIsVisible()"
      [disabled]="!column.getCanHide()"
      (change)="column.getToggleVisibilityHandler()?.($event)"
    />
    {{ column.id }}
  </label>
}
```

### Column Visibility Aware Table APIs

When you render your header, body, and footer cells, there are a lot of API options available. You may see APIs like `table.getAllLeafColumns` and `row.getAllCells`, but if you use these APIs, they will not take column visibility into account. Instead, you need to use the "visible" variants of these APIs, such as `table.getVisibleLeafColumns` and `row.getVisibleCells`.

```html
<table>
  <thead>
    <tr>
      @for (column of table.getVisibleLeafColumns(); track column.id) {
        <th>{{ column.id }}</th>
      }
    </tr>
  </thead>
  <tbody>
    @for (row of table.getRowModel().rows; track row.id) {
      <tr>
        @for (cell of row.getVisibleCells(); track cell.id) {
          <td>
            <ng-container *flexRenderCell="cell; let renderCell">{{ renderCell }}</ng-container>
          </td>
        }
      </tr>
    }
  </tbody>
</table>
```

If you are using the Header Group APIs, they will already take column visibility into account.
