# 03 – Dataset Components (Grids and Views)

Dataset components replace or enhance Dataverse views/grids.
They access a collection of records via `context.parameters.<datasetName>`.

---

## Manifest for a dataset component

```xml
<manifest>
  <control namespace="MyNS" constructor="MyGrid"
           version="0.0.1" control-type="standard">

    <data-set name="myDataSet" display-name-key="My Dataset">
      <!-- Fixed columns (optional) -->
      <!-- <property-set name="Name" display-name-key="Name" of-type="SingleLine.Text" /> -->
    </data-set>

    <resources>
      <code path="index.ts" order="1" />
      <css path="css/MyGrid.css" order="1" />
    </resources>
  </control>
</manifest>
```

---

## index.ts structure for a dataset

```typescript
import { IInputs, IOutputs } from "./generated/ManifestTypes";

export class MyGrid
  implements ComponentFramework.StandardControl<IInputs, IOutputs>
{
  private _container: HTMLDivElement;
  private _context: ComponentFramework.Context<IInputs>;

  constructor() {}

  public init(
    context: ComponentFramework.Context<IInputs>,
    notifyOutputChanged: () => void,
    state: ComponentFramework.Dictionary,
    container: HTMLDivElement
  ): void {
    this._container = container;
    this._context = context;

    // ⚠️ Data is NOT available in init.
    // Configure page size here:
    context.parameters.myDataSet.paging.setPageSize(25);
  }

  public updateView(context: ComponentFramework.Context<IInputs>): void {
    this._context = context;
    const ds = context.parameters.myDataSet;

    if (ds.loading) { this._showLoading(); return; }
    if (ds.error)   { this._showError(ds.errorMessage); return; }

    this._renderGrid(ds);
  }

  private _renderGrid(ds: ComponentFramework.PropertyTypes.DataSet): void {
    // See sections below for ds usage
  }

  public getOutputs(): IOutputs { return {}; }
  public destroy(): void {}
}
```

---

## Columns

```typescript
const ds = context.parameters.myDataSet;

ds.columns.forEach((col: ComponentFramework.PropertyHelper.DataSetApi.Column) => {
  col.name             // Dataverse logical name ("createdon", "name", "new_amount")
  col.displayName      // label shown to the user
  col.dataType         // data type: "SingleLine.Text", "Whole.None", "DateAndTime.DateOnly"...
  col.alias            // column alias in the view
  col.order            // display order (int)
  col.visualSizeFactor // suggested relative width (float, e.g. 0.3)
  col.isHidden         // boolean – column hidden in the view
  col.isPrimary        // boolean – primary column (record name)
  col.disableSorting   // boolean – sorting disabled for this column
});

// Build a table header
const thead = document.createElement("tr");
ds.columns
  .filter(col => !col.isHidden)
  .sort((a, b) => a.order - b.order)
  .forEach(col => {
    const th = document.createElement("th");
    th.innerText = col.displayName;
    thead.appendChild(th);
  });
```

---

## Records

```typescript
const ds = context.parameters.myDataSet;

// sortedRecordIds = array of GUIDs in view order
ds.sortedRecordIds.forEach(id => {
  const record = ds.records[id];

  // Read a value (returns string | number | boolean | Date | LookupValue[] | number[])
  const name:   string  = record.getValue("name") as string;
  const amount: number  = record.getValue("new_amount") as number;
  const date:   Date    = record.getValue("createdon") as Date;
  const active: boolean = record.getValue("statecode") as boolean;

  // Formatted value according to locale (always string)
  const formattedAmount: string = record.getFormattedValue("new_amount");
  const formattedDate:   string = record.getFormattedValue("createdon");

  // Record information
  const recordId:   string = record.getRecordId();    // GUID
  const tableName:  string = record.getEntityName();  // "account", "contact"...
  const entityRef          = record.getNamedReference(); // EntityReference

  // Open the record form
  ds.openDatasetItem(entityRef);
});
```

---

## Pagination

```typescript
const paging = ds.paging;

// Pagination info
paging.pageSize              // current page size
paging.totalResultCount      // total count (model-driven only, not canvas)
paging.hasNextPage           // boolean
paging.hasPreviousPage       // boolean
paging.firstPageNumber       // current first page number (1-based)
paging.lastPageNumber        // current last page number

// Change page size (call in init or updateView)
paging.setPageSize(50);

// Navigation
paging.loadNextPage();         // next page
paging.loadPreviousPage();     // previous page
paging.loadExactPage(3);       // jump directly to page 3
paging.reset();                // back to page 1

// Example: pagination buttons
const prevBtn = document.createElement("button");
prevBtn.disabled = !paging.hasPreviousPage;
prevBtn.onclick = () => paging.loadPreviousPage();

const nextBtn = document.createElement("button");
nextBtn.disabled = !paging.hasNextPage;
nextBtn.onclick = () => paging.loadNextPage();
```

---

## Filtering

```typescript
// Apply a filter and refresh
ds.filtering.setFilter({
  filterOperator: 0,    // 0 = AND, 1 = OR
  conditions: [
    {
      attributeName: "statecode",
      conditionOperator: 0,   // 0 = Equal
      value: 0,               // 0 = Active
    },
    {
      attributeName: "name",
      conditionOperator: 6,   // 6 = Like
      value: "Contoso%",
    },
  ],
});
ds.refresh(); // reload with the active filter

// Clear filters
ds.filtering.clearFilter();
ds.refresh();

// Available operators (conditionOperator)
// -1 = None         0 = Equal        1 = NotEqual
//  2 = GreaterThan  3 = LessThan     4 = GreaterEqual   5 = LessEqual
//  6 = Like         8 = In          12 = Null           13 = NotNull
// 14 = Yesterday   15 = Today       16 = Tomorrow
// 33 = LastXDays   34 = NextXDays
// 49 = Contains    75 = Above       76 = Under
// 87 = ContainValues (multi-select)

// ⚠️ Filtering is only available for Dataverse data sources.
```

---

## Sorting

```typescript
// Configure sorting (multiple columns supported)
ds.sorting = [
  { name: "createdon",   sortDirection: 1 },  // 0=ASC, 1=DESC
  { name: "name",        sortDirection: 0 },
];
ds.refresh(); // apply

// Read current sorting
ds.sorting.forEach(s => {
  console.log(s.name, s.sortDirection);
});

// ⚠️ Changing sorting resets existing filters.
// ⚠️ Sorting is only available for Dataverse data sources.

// Example: clickable header to sort
th.onclick = () => {
  const current = ds.sorting.find(s => s.name === col.name);
  ds.sorting = [{
    name: col.name,
    sortDirection: current?.sortDirection === 0 ? 1 : 0,
  }];
  ds.refresh();
};
```

---

## Record selection

```typescript
// Select programmatically
ds.setSelectedRecordIds(["id1", "id2", "id3"]);

// Read current selection
const selectedIds: string[] = ds.getSelectedRecordIds();

// Clear selection
ds.clearSelectedRecordIds();

// Example: checkbox per row
checkbox.onchange = (evt) => {
  const checked = (evt.target as HTMLInputElement).checked;
  const current = ds.getSelectedRecordIds();
  if (checked) {
    ds.setSelectedRecordIds([...current, recordId]);
  } else {
    ds.setSelectedRecordIds(current.filter(id => id !== recordId));
  }
};
```

---

## Dataset metadata

```typescript
ds.getTargetEntityType()  // "account" | "contact" | ...  (table logical name)
ds.getTitle()             // Active view name (e.g. "Active Accounts")
ds.getViewId()            // Active view GUID

// Add a column dynamically (model-driven only)
ds.addColumn("new_mycolumn", "myDataSet");
```

---

## Manual refresh

```typescript
// Force a reload from Dataverse
// (after setFilter, sorting, setPageSize, or from a button)
ds.refresh();
```

---

## Full example: rendering an HTML table

```typescript
private _renderGrid(ds: ComponentFramework.PropertyTypes.DataSet): void {
  this._container.innerHTML = "";

  const table = document.createElement("table");
  table.className = "MyNS_MyGrid-table";

  // Header
  const thead = document.createElement("thead");
  const headerRow = document.createElement("tr");
  const visibleCols = ds.columns
    .filter(c => !c.isHidden)
    .sort((a, b) => a.order - b.order);

  visibleCols.forEach(col => {
    const th = document.createElement("th");
    th.innerText = col.displayName;
    th.onclick = () => this._sortBy(col.name);
    headerRow.appendChild(th);
  });
  thead.appendChild(headerRow);
  table.appendChild(thead);

  // Body
  const tbody = document.createElement("tbody");
  ds.sortedRecordIds.forEach(id => {
    const record = ds.records[id];
    const tr = document.createElement("tr");
    tr.onclick = () => ds.openDatasetItem(record.getNamedReference());

    visibleCols.forEach(col => {
      const td = document.createElement("td");
      td.innerText = record.getFormattedValue(col.name) ?? "";
      tr.appendChild(td);
    });
    tbody.appendChild(tr);
  });
  table.appendChild(tbody);
  this._container.appendChild(table);

  // Pagination
  this._renderPagination(ds.paging);
}

private _sortBy(columnName: string): void {
  const ds = this._context.parameters.myDataSet;
  const current = ds.sorting.find(s => s.name === columnName);
  ds.sorting = [{
    name: columnName,
    sortDirection: current?.sortDirection === 0 ? 1 : 0,
  }];
  ds.refresh();
}
```
