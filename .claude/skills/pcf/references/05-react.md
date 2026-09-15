# 05 – React / Virtual Components with Fluent UI

React PCF components (called "Virtual Controls") share React and Fluent UI libraries
with the Power Apps platform: smaller bundle, better performance, consistent design.

> Available for **Model-Driven Apps** and **Canvas Apps**.
> Not available for **Power Pages**.

---

## Scaffold

```bash
pac pcf init -n MyReactComponent -ns MyNS -t field -fw react -npm
# -fw react  →  generates a Virtual (ReactControl) project
```

---

## Differences from a standard component

| Aspect | Standard (control-type="standard") | React (control-type="virtual") |
|---|---|---|
| Implemented class | `StandardControl<I,O>` | `ReactControl<I,O>` |
| `container` parameter in init | ✅ Yes | ❌ No |
| `updateView` return type | `void` | `React.ReactElement` |
| React/Fluent bundled in bundle.js | ✅ Yes (increases size) | ❌ No (provided by platform) |

---

## ControlManifest.Input.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<manifest>
  <control namespace="MyNS"
           constructor="MyReactComponent"
           version="0.0.1"
           display-name-key="MyReactComponent"
           description-key="MyReactComponent_Desc"
           control-type="virtual">
    <!-- ↑ "virtual" is mandatory for React components -->

    <property name="value"
              of-type="Whole.None"
              usage="bound"
              required="true"
              display-name-key="Value" />

    <resources>
      <code path="index.ts" order="1" />
      <!-- Platform libraries: do NOT install these via npm -->
      <platform-library name="React" version="16.14.0" />
      <platform-library name="Fluent" version="9.46.2" />
      <!-- Use Fluent 8 OR Fluent 9, not both at the same time -->
    </resources>

  </control>
</manifest>
```

### Available versions

| npm package | Supported versions | Runtime version loaded |
|---|---|---|
| `react` | 16.14.0 | 17.0.2 (model-driven) / 16.14.0 (canvas) |
| `@fluentui/react` (Fluent 8) | 8.29.0 or 8.121.1 | 8.29.0 / 8.121.1 |
| `@fluentui/react-components` (Fluent 9) | >=9.4.0 <=9.46.2 | 9.68.0 |

> The platform may load a higher compatible version at runtime.

---

## index.ts – ReactControl structure

```typescript
import * as React from "react";
import { IInputs, IOutputs } from "./generated/ManifestTypes";
import { SliderComponent } from "./components/SliderComponent";

export class MyReactComponent
  implements ComponentFramework.ReactControl<IInputs, IOutputs>
{
  private _notifyOutputChanged: () => void;
  private _context: ComponentFramework.Context<IInputs>;
  private _value: number;

  constructor() {}

  // init: NO container parameter
  public init(
    context: ComponentFramework.Context<IInputs>,
    notifyOutputChanged: () => void,
    state: ComponentFramework.Dictionary
  ): void {
    this._context = context;
    this._notifyOutputChanged = notifyOutputChanged;
    this._value = context.parameters.value.raw ?? 0;
  }

  // updateView: RETURNS a ReactElement (not void)
  public updateView(
    context: ComponentFramework.Context<IInputs>
  ): React.ReactElement {
    this._context = context;
    this._value = context.parameters.value.raw ?? 0;

    return React.createElement(SliderComponent, {
      value: this._value,
      min: 0,
      max: 100,
      isDisabled: context.mode.isControlDisabled,
      label: context.mode.label,
      onChange: this._handleChange,
    });
  }

  private _handleChange = (newValue: number): void => {
    if (newValue !== this._value) {
      this._value = newValue;
      this._notifyOutputChanged();
    }
  };

  public getOutputs(): IOutputs {
    return { value: this._value };
  }

  public destroy(): void {}
}
```

---

## React component with Fluent UI 9

```typescript
// components/SliderComponent.tsx
import * as React from "react";
import { Slider, Label, FluentProvider, webLightTheme } from "@fluentui/react-components";

interface SliderProps {
  value: number;
  min: number;
  max: number;
  isDisabled: boolean;
  label: string;
  onChange: (value: number) => void;
}

export const SliderComponent: React.FC<SliderProps> = ({
  value, min, max, isDisabled, label, onChange
}) => {
  return (
    <FluentProvider theme={webLightTheme}>
      <div style={{ display: "flex", flexDirection: "column", gap: "4px", width: "100%" }}>
        <Label>{label}</Label>
        <Slider
          min={min}
          max={max}
          value={value}
          disabled={isDisabled}
          onChange={(_, data) => onChange(data.value)}
          style={{ width: "100%" }}
        />
        <span style={{ fontSize: "12px", color: "#605e5c" }}>{value}</span>
      </div>
    </FluentProvider>
  );
};
```

---

## React component with Fluent UI 8

```typescript
// components/ButtonComponent.tsx
import * as React from "react";
import { PrimaryButton, DefaultButton, Stack } from "@fluentui/react";

interface ButtonProps {
  primaryLabel: string;
  secondaryLabel: string;
  onPrimary: () => void;
  onSecondary: () => void;
  isDisabled: boolean;
}

export const ButtonComponent: React.FC<ButtonProps> = ({
  primaryLabel, secondaryLabel, onPrimary, onSecondary, isDisabled
}) => {
  return (
    <Stack horizontal tokens={{ childrenGap: 8 }}>
      <PrimaryButton
        text={primaryLabel}
        onClick={onPrimary}
        disabled={isDisabled}
      />
      <DefaultButton
        text={secondaryLabel}
        onClick={onSecondary}
        disabled={isDisabled}
      />
    </Stack>
  );
};
```

---

## React component with hooks (useState, useEffect)

```typescript
// components/SearchComponent.tsx
import * as React from "react";
import { useState, useEffect, useCallback } from "react";

interface SearchProps {
  initialValue: string;
  isDisabled: boolean;
  onSearch: (query: string) => void;
}

export const SearchComponent: React.FC<SearchProps> = ({
  initialValue, isDisabled, onSearch
}) => {
  const [query, setQuery] = useState(initialValue);
  const [isLoading, setIsLoading] = useState(false);

  // Sync if the external value changes
  useEffect(() => {
    setQuery(initialValue);
  }, [initialValue]);

  const handleSearch = useCallback(async () => {
    setIsLoading(true);
    await onSearch(query);
    setIsLoading(false);
  }, [query, onSearch]);

  return (
    <div style={{ display: "flex", gap: "8px", alignItems: "center" }}>
      <input
        type="text"
        value={query}
        disabled={isDisabled}
        onChange={(e) => setQuery(e.target.value)}
        onKeyDown={(e) => e.key === "Enter" && handleSearch()}
        style={{ flex: 1, padding: "6px 8px", border: "1px solid #8a8886" }}
      />
      <button onClick={handleSearch} disabled={isDisabled || isLoading}>
        {isLoading ? "..." : "Search"}
      </button>
    </div>
  );
};
```

---

## Passing context to React components

```typescript
// In updateView, pass only the data needed
// (do not pass the context object directly to a React component)

public updateView(context: ComponentFramework.Context<IInputs>): React.ReactElement {
  return React.createElement(MyComponent, {
    // Data
    value: context.parameters.value.raw ?? 0,
    // Options from Dataverse
    options: context.parameters.optionSetProp.attributes?.Options ?? [],
    // State
    isDisabled: context.mode.isControlDisabled,
    isVisible: context.mode.isVisible,
    // Dimensions
    width: context.mode.allocatedWidth,
    height: context.mode.allocatedHeight,
    // Locale
    languageId: context.userSettings.languageId,
    // Callbacks
    onChange: this._handleChange,
    onNavigate: (id: string) => context.navigation.openForm({ entityName: "account", entityId: id }),
  });
}
```

---

## CSS for React components

For Virtual/React components, prefer CSS-in-JS or inline styles
rather than external CSS files, as scoping is handled by React:

```typescript
// Inline styles (simple)
const containerStyle: React.CSSProperties = {
  display: "flex",
  gap: "8px",
  width: "100%",
  padding: "4px",
};

// makeStyles from Fluent UI 9 (recommended)
import { makeStyles } from "@fluentui/react-components";
const useStyles = makeStyles({
  container: { display: "flex", gap: "8px" },
  label: { fontWeight: "600", color: "#323130" },
});

const MyComp = () => {
  const styles = useStyles();
  return <div className={styles.container}><span className={styles.label}>...</span></div>;
};
```

---

## package.json – React/Fluent dependencies

```json
{
  "dependencies": {
    "@types/react": "^16.14.0",
    "@fluentui/react-components": "^9.46.2"
  },
  "devDependencies": {
    "react": "^16.14.0",
    "@types/react": "^16.14.0"
  }
}
```

> React and Fluent are NOT bundled in bundle.js thanks to `platform-library`.
> They remain in `devDependencies` for TypeScript compilation only.
