# 02 – Coding a PCF Component

## The 4 lifecycle methods

```typescript
import { IInputs, IOutputs } from "./generated/ManifestTypes";

export class MyComponent
  implements ComponentFramework.StandardControl<IInputs, IOutputs>
{
  // ── Private properties ────────────────────────────────────────────────
  private _container: HTMLDivElement;
  private _context: ComponentFramework.Context<IInputs>;
  private _notifyOutputChanged: () => void;
  private _currentValue: number;
  private _inputElement: HTMLInputElement;
  private _labelElement: HTMLLabelElement;

  constructor() {}

  // ── 1. init ──────────────────────────────────────────────────────────
  // Called ONCE on load.
  // Goal: build the DOM, attach listeners, read the initial value.
  // ⚠️ Dataset data is NOT available here — use updateView instead.
  public init(
    context: ComponentFramework.Context<IInputs>,
    notifyOutputChanged: () => void,
    state: ComponentFramework.Dictionary,  // persistent session state
    container: HTMLDivElement
  ): void {
    this._context = context;
    this._notifyOutputChanged = notifyOutputChanged;
    this._container = container;

    // Read the initial value from Dataverse
    this._currentValue = context.parameters.controlValue.raw ?? 0;

    // Build DOM elements
    this._inputElement = document.createElement("input");
    this._inputElement.setAttribute("type", "range");
    this._inputElement.setAttribute("min", "0");
    this._inputElement.setAttribute("max", "100");
    this._inputElement.setAttribute("class", "MyNS-slider");
    this._inputElement.value = String(this._currentValue);
    this._inputElement.addEventListener("input", this._onChange);

    this._labelElement = document.createElement("label");
    this._labelElement.innerText = String(this._currentValue);

    container.appendChild(this._inputElement);
    container.appendChild(this._labelElement);

    // Enable container resize tracking
    context.mode.trackContainerResize(true);
  }

  // ── 2. updateView ────────────────────────────────────────────────────
  // Called on EVERY context change:
  //   - field value changed externally
  //   - container resized
  //   - mode changed (disabled, visible)
  //   - focus returned, tab changed, etc.
  // ⚠️ Never recreate the entire DOM here — only update it.
  public updateView(context: ComponentFramework.Context<IInputs>): void {
    this._context = context;

    // Optimisation: only process what changed
    const changed = context.updatedProperties;

    if (changed.includes("controlValue")) {
      this._currentValue = context.parameters.controlValue.raw ?? 0;
      this._inputElement.value = String(this._currentValue);
      this._labelElement.innerText = context.parameters.controlValue.formatted ?? String(this._currentValue);
    }

    if (changed.includes("layout")) {
      // Adapt to the new container size
      const width = context.mode.allocatedWidth;
      this._inputElement.style.width = `${width - 60}px`;
    }

    // Handle read-only mode
    this._inputElement.disabled = context.mode.isControlDisabled;
  }

  // ── 3. getOutputs ────────────────────────────────────────────────────
  // Called by the framework AFTER notifyOutputChanged().
  // Return the values to write back to Dataverse.
  // ⚠️ Only properties declared as usage="bound" or usage="output".
  public getOutputs(): IOutputs {
    return {
      controlValue: this._currentValue,
    };
  }

  // ── 4. destroy ───────────────────────────────────────────────────────
  // Called when the component is removed from the DOM.
  // ⚠️ Mandatory: remove ALL listeners to avoid memory leaks.
  public destroy(): void {
    this._inputElement.removeEventListener("input", this._onChange);
  }

  // ── Private handler ──────────────────────────────────────────────────
  private _onChange = (evt: Event): void => {
    const newVal = Number((evt.target as HTMLInputElement).value);
    if (newVal !== this._currentValue) {
      this._currentValue = newVal;
      this._labelElement.innerText = String(newVal);
      this._notifyOutputChanged(); // triggers getOutputs()
    }
  };
}
```

---

## context.parameters – Reading values and metadata

```typescript
// Raw value (type depends on the of-type declared in the manifest)
const raw: number  = context.parameters.controlValue.raw!;
const raw: string  = context.parameters.myText.raw!;
const raw: boolean = context.parameters.myBoolean.raw!;
const raw: Date    = context.parameters.myDate.raw!;

// Formatted value according to the user's locale (always a string)
const formatted: string = context.parameters.controlValue.formatted!;

// Dataverse column metadata
const attrs = context.parameters.controlValue.attributes;
attrs?.IsDisabled         // boolean – field disabled at Dataverse level
attrs?.DisplayName        // string  – field label
attrs?.LogicalName        // string  – logical name ("new_amount")
attrs?.RequiredLevel      // 0=None 1=SystemRequired 2=ApplicationRequired 3=Recommended
attrs?.MaxLength          // number  – for text types
attrs?.MinValue           // number  – for numeric types
attrs?.MaxValue           // number  – for numeric types
attrs?.Precision          // number  – decimal places for Currency/Decimal/FP
attrs?.ImeMode            // for text fields
// For OptionSet / MultiSelectOptionSet:
attrs?.Options            // Array<{ Value: number, Label: string, Color?: string }>
attrs?.DefaultValue       // number – default option set value

// Property type
context.parameters.controlValue.type
// "SingleLine.Text" | "Whole.None" | "Decimal" | "FP" | "Currency" |
// "TwoOptions" | "DateAndTime.DateOnly" | "DateAndTime.DateAndTime" |
// "OptionSet" | "MultiSelectOptionSet" | "Lookup.Simple" | "Multiple"
```

---

## context.mode – Component state

```typescript
context.mode.isControlDisabled    // boolean – field is read-only
context.mode.isVisible            // boolean – component is visible
context.mode.label                // string  – field label in the form
context.mode.allocatedHeight      // number  – allocated height (px), -1 if auto
context.mode.allocatedWidth       // number  – allocated width (px)

// Enable resize notification
// → triggers updateView with "layout" in updatedProperties
context.mode.trackContainerResize(true);

// Persist state across updateView calls (survives refresh)
context.mode.setControlState({ myValue: this._value, page: 2 });
// Retrieve in init via the state parameter:
// const saved = state?.["myValue"];

// Full screen (canvas apps only)
context.mode.setFullScreen(true);
```

---

## context.updatedProperties – Optimising re-renders

```typescript
public updateView(context: ComponentFramework.Context<IInputs>): void {
  const changed = context.updatedProperties;
  // Possible values:
  // "controlValue"    → field value changed
  // "maxValue"        → input property changed
  // "layout"          → container resized
  // "fullscreen_open" / "fullscreen_close"
  // "OfflineStatus"   → went online/offline

  if (changed.includes("controlValue")) {
    // only recalculate if the value actually changed
  }
  if (changed.includes("layout")) {
    const { allocatedWidth, allocatedHeight } = context.mode;
    // adapt the layout
  }
}
```

---

## context.formatting – Formatting values

```typescript
// Numbers
context.formatting.formatCurrency(1234.56, 2, "$")     // "$1,234.56"
context.formatting.formatDecimal(1234.56, 2)            // "1,234.56"
context.formatting.formatInteger(1234)                  // "1,234"

// Dates (according to the organisation locale)
context.formatting.formatDateShort(new Date())          // "12/31/2024"
context.formatting.formatDateLong(new Date())           // "Tuesday, December 31, 2024"
context.formatting.formatDateLongAbbreviated(new Date())// "Tue, Dec 31, 2024"
context.formatting.formatDateYearMonth(new Date())      // "December 2024"
context.formatting.formatTime(new Date(), 0)            // "2:30 PM"
// UTC ↔ local conversions
context.formatting.formatUserDateTimeToUTC(new Date())  // ISO UTC string
context.formatting.formatUTCDateTimeToUserDate("2024-12-31T14:30:00Z") // local Date
// Parse
context.formatting.parseDateFromInput("12/31/2024")    // Date object | null
context.formatting.getWeekOfYear(new Date())           // 1-53
```

---

## context.navigation – Navigating from the component

```typescript
// Open an existing record
await context.navigation.openForm({
  entityName: "account",
  entityId: "00000000-0000-0000-0000-000000000001",
});

// Open a blank create form
await context.navigation.openForm({ entityName: "contact" });

// Confirmation dialog
const result = await context.navigation.openConfirmDialog({
  text: "Delete this item?",
  title: "Confirm",
  confirmButtonLabel: "Delete",
  cancelButtonLabel: "Cancel",
});
if (result.confirmed) { /* proceed */ }

// Simple alert
await context.navigation.openAlertDialog({
  text: "Operation completed.",
  title: "Success",
}, { height: 180, width: 350 });

// Open a URL
await context.navigation.openUrl("https://example.com");

// Download / open a file
await context.navigation.openFile({
  fileContent: base64String,
  fileName: "report.pdf",
  fileSize: 54321,
  mimeType: "application/pdf",
});
```

---

## context.device – Native capabilities (mobile)

```typescript
// Camera
const img = await context.device.captureImage({
  allowEdit: true,
  quality: 90,      // 0-100
  height: 600,
  width: 800,
});
// img.fileContent: base64 string, img.mimeType, img.fileName

// Geolocation
const pos = await context.device.getCurrentPosition();
const { latitude, longitude, accuracy } = pos.coords;

// Barcode scanner (mobile only)
const scan = await context.device.getBarcodeValue();
const code: string = scan.value;

// Audio
const audio = await context.device.captureAudio();
```

---

## context.resources – Manifest resources

```typescript
// Localised string from a .resx file
const label: string = context.resources.getString("MyKey");

// Image or binary file (returns base64 via callback)
context.resources.getResource(
  "img/icon.png",
  (data: string) => {
    const img = document.createElement("img");
    img.src = `data:image/png;base64,${data}`;
    this._container.appendChild(img);
  },
  () => console.error("Resource not found")
);
```

---

## context.client – Execution context

```typescript
context.client.getClient()       // "Web" | "UCI" | "Mobile" | "Outlook"
context.client.getFormFactor()   // 0=Unknown 1=Desktop 2=Tablet 3=Phone
context.client.isOffline()       // boolean

// Adapt behaviour to the client
if (context.client.getFormFactor() === 3) {
  // mobile: simplified UI
}
```

---

## context.userSettings – User information

```typescript
context.userSettings.userId                // logged-in user GUID
context.userSettings.userName              // "First Last"
context.userSettings.languageId            // 1033=en, 1036=fr, 3082=es...
context.userSettings.isRTL                 // true for Arabic/Hebrew
context.userSettings.timeZoneOffsetMinutes // UTC offset in minutes
context.userSettings.dateFormattingInfo    // org date format object
context.userSettings.numberFormattingInfo  // org number format object
```

---

## CSS – Mandatory scoping rules

```css
/* ALWAYS prefix with MyNS\.MyComponent
   to avoid polluting the host form */

.MyNS\.MyComponent {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
}

.MyNS\.MyComponent .MyNS-slider {
  flex: 1;
  margin: 4px 0;
  -webkit-appearance: none;
  height: 4px;
  background: #0078d4;
  border-radius: 2px;
}

.MyNS\.MyComponent .MyNS-label {
  min-width: 40px;
  text-align: right;
  font-size: 14px;
  color: #323130;
}

.MyNS\.MyComponent .MyNS-button {
  background: #0078d4;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.MyNS\.MyComponent .MyNS-button:hover {
  background: #106ebe;
}

.MyNS\.MyComponent .MyNS-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

---

## Accessing the current record ID and table name

```typescript
// Not officially exposed, but accessible via cast:
const pageContext = (context as any).page;
const entityId:   string = pageContext?.entityId;       // record GUID
const entityName: string = pageContext?.entityTypeName; // "account", "contact"...
```
