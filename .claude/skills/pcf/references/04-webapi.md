# 04 – WebAPI – Dataverse CRUD from a PCF Component

`context.webAPI` allows direct Dataverse calls from within the component.

> ⚠️ **Available in Model-Driven Apps only.**
> Not available in Canvas Apps or Power Pages.
> For Canvas Apps, use Power Automate connectors instead.

---

## CREATE – Create a record

```typescript
const result = await context.webAPI.createRecord(
  entityLogicalName: string,
  data: ComponentFramework.WebApi.Entity
);
// result.id         → new record GUID (string)
// result.entityType → table name

// Example: create an account
try {
  const result = await context.webAPI.createRecord("account", {
    name: "Contoso Ltd",
    telephone1: "555-0100",
    emailaddress1: "contact@contoso.com",
    websiteurl: "https://contoso.com",
    revenue: 1000000,
    numberofemployees: 250,
    // Relationship — use the navigation property, not the raw key:
    "parentaccountid@odata.bind": `/accounts(${parentId})`,
  });
  const newId: string = result.id;
} catch (error) {
  console.error("Create error:", error.message);
}
```

---

## READ – Retrieve a single record

```typescript
const record = await context.webAPI.retrieveRecord(
  entityLogicalName: string,
  id: string,               // GUID (with or without braces)
  options?: string          // OData querystring: $select, $expand
);

// Example: read a contact with specific fields
const contact = await context.webAPI.retrieveRecord(
  "contact",
  "00000000-0000-0000-0000-000000000001",
  "?$select=firstname,lastname,emailaddress1,_parentcustomerid_value"
);
const firstName: string = contact["firstname"];
const lastName:  string = contact["lastname"];
const email:     string = contact["emailaddress1"];

// Access a lookup field (returns the GUID)
const accountId: string = contact["_parentcustomerid_value"];
// Access the formatted lookup name
const accountName: string = contact["_parentcustomerid_value@OData.Community.Display.V1.FormattedValue"];

// Expand: retrieve related data in a single request
const account = await context.webAPI.retrieveRecord(
  "account",
  accountId,
  "?$select=name&$expand=contact_customer_accounts($select=firstname,lastname)"
);
const contacts = account["contact_customer_accounts"];
```

---

## READ MULTIPLE – Retrieve a collection

```typescript
const response = await context.webAPI.retrieveMultipleRecords(
  entityLogicalName: string,
  options?: string,      // OData querystring
  maxPageSize?: number   // page size (default: 5000, max: 5000)
);
// response.entities   → EntityRecord[]
// response.nextLink   → next page URL (string | undefined)

// Example: active accounts, sorted, 10 per page
const response = await context.webAPI.retrieveMultipleRecords(
  "account",
  "?$select=name,telephone1,emailaddress1" +
  "&$filter=statecode eq 0" +
  "&$orderby=name asc" +
  "&$top=10"
);
const accounts = response.entities;
accounts.forEach(acc => {
  const name: string  = acc["name"];
  const phone: string = acc["telephone1"];
});

// Pagination: fetch all pages
async function fetchAll(entityName: string, query: string) {
  const all: ComponentFramework.WebApi.Entity[] = [];
  let result = await context.webAPI.retrieveMultipleRecords(entityName, query);
  all.push(...result.entities);
  while (result.nextLink) {
    result = await context.webAPI.retrieveMultipleRecords(entityName, result.nextLink);
    all.push(...result.entities);
  }
  return all;
}

// Common OData filters
// statecode eq 0                       → active records
// contains(name,'Contoso')             → name contains "Contoso"
// createdon ge 2024-01-01              → created after Jan 1 2024
// revenue gt 100000                    → revenue > 100,000
// _ownerid_value eq '<GUID>'           → owned by a specific user
```

---

## UPDATE – Update a record

```typescript
// Updates only the provided fields (PATCH)
await context.webAPI.updateRecord(
  entityLogicalName: string,
  id: string,
  data: ComponentFramework.WebApi.Entity
);

// Example: update a contact
await context.webAPI.updateRecord(
  "contact",
  "00000000-0000-0000-0000-000000000001",
  {
    firstname: "John",
    lastname: "Doe",
    emailaddress1: "john.doe@example.com",
    // Change a lookup:
    "parentcustomerid_account@odata.bind": `/accounts(${newAccountId})`,
    // Clear a lookup:
    // "parentcustomerid_account": null,
  }
);
```

---

## DELETE – Delete a record

```typescript
await context.webAPI.deleteRecord(
  entityLogicalName: string,
  id: string
);

// Example
try {
  await context.webAPI.deleteRecord("task", taskId);
  console.log("Task deleted");
} catch (error) {
  console.error("Delete error:", error.message);
}
```

---

## Error handling

```typescript
// All webAPI methods return Promises.
// Always use try/catch or .catch()

// With async/await
try {
  const result = await context.webAPI.createRecord("account", data);
} catch (error) {
  // error.message : Dataverse error message
  // error.code    : HTTP error code (400, 403, 404...)
  console.error(`Code: ${error.code}, Message: ${error.message}`);
  await context.navigation.openAlertDialog({
    title: "Error",
    text: `Could not create the record: ${error.message}`,
  });
}
```

---

## Useful OData query examples

```typescript
// Search by name (case-insensitive)
"?$select=name&$filter=contains(tolower(name),'contoso')"

// Recent records (last 7 days)
"?$select=name,createdon&$filter=createdon ge " +
  new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString() +
  "&$orderby=createdon desc"

// Records owned by the current user
`?$select=name&$filter=_ownerid_value eq ${context.userSettings.userId}`

// Count records ($count=true)
"?$select=name&$filter=statecode eq 0&$count=true"
// response["@odata.count"]

// FetchXML (for complex queries)
const fetchXml = `
  <fetch top="10">
    <entity name="account">
      <attribute name="name" />
      <attribute name="telephone1" />
      <filter>
        <condition attribute="statecode" operator="eq" value="0" />
      </filter>
    </entity>
  </fetch>`;
const query = `?fetchXml=${encodeURIComponent(fetchXml)}`;
const result = await context.webAPI.retrieveMultipleRecords("account", query);
```

---

## Common table logical names

| Display name | Logical name |
|---|---|
| Account | `account` |
| Contact | `contact` |
| Opportunity | `opportunity` |
| Task | `task` |
| Email | `email` |
| Phone Call | `phonecall` |
| Appointment | `appointment` |
| User | `systemuser` |
| Team | `team` |
| Queue | `queue` |
| Note (Annotation) | `annotation` |
