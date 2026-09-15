# 06 – Build & Debug

## Build commands

```bash
# Development build (fast, source maps included)
npm run build

# Production build (minified, optimised, no eval)
npm run build -- --buildMode production

# Regenerate TypeScript types from the manifest
npm run refreshTypes
```

### What the build produces

```
out/
└── controls/
    └── MyComponent/
        ├── bundle.js           ← compiled and bundled code (upload to Dataverse)
        └── ControlManifest.xml ← final manifest (different from the .Input.xml source)
```

---

## Debug with the PCF Test Harness

```bash
npm start watch
```

- Starts a dev server with hot reload
- Automatically opens **http://localhost:8181**
- TypeScript/CSS changes are recompiled and reloaded without restarting

### Using the test harness

The PCF test harness simulates the Power Apps environment in the browser:

- **Component** tab: change property values in real time
- **Context** tab: simulate `mode.isControlDisabled`, `mode.allocatedWidth/Height`, etc.
- The component behaves exactly as it would in Power Apps

### Stop the server

```
Ctrl + C
```

---

## Debugging in the browser (DevTools)

The development build includes source maps. To debug:

1. Open http://localhost:8181
2. Open DevTools (F12)
3. Go to **Sources** → find `index.ts` in the file tree
4. Set breakpoints directly in the TypeScript source

In production (after deploying to Dataverse):
1. Open the model-driven app
2. F12 → Sources → find `bundle.js`
3. Use source maps if available, otherwise debug the compiled JS

---

## Common build errors

| Error | Cause | Fix |
|---|---|---|
| `'EventListenerOrEventListenerObject' is not defined no-undef` | ESLint rule too strict | In `.eslintrc.json`: `"no-undef": ["warn"]` |
| `npm not found` | Node.js not installed or not on PATH | Reinstall Node.js LTS and restart the terminal |
| `Cannot find module './generated/ManifestTypes'` | Types not generated | Run `npm run refreshTypes` |
| `Class 'X' incorrectly implements interface 'StandardControl'` | Incorrect method signature | Check the signatures of `init`, `updateView`, `getOutputs`, `destroy` |
| `Property 'xxx' does not exist on type 'IInputs'` | Property not declared in the manifest | Add the property to `ControlManifest.Input.xml` then `npm run refreshTypes` |
| `Module not found: Error: Can't resolve 'react'` | React not installed | `npm install react @types/react` |

### Fix the `no-undef` ESLint error

Open `.eslintrc.json` and change:

```json
{
  "rules": {
    "no-unused-vars": "off",
    "no-undef": ["warn"]
  }
}
```

---

## Linting and code quality

```bash
# Check code without building
npx eslint MyComponent/index.ts

# Auto-fix fixable issues
npx eslint MyComponent/index.ts --fix
```

---

## Check tool versions

```bash
pac --version          # Power Platform CLI
node --version         # Node.js
npm --version          # npm
npx pcf-scripts --help # PCF Scripts
```

---

## Testing multiple configurations

In the test harness (http://localhost:8181), the **Context** tab lets you simulate:

- `isControlDisabled: true/false` → test read-only mode
- `allocatedWidth` / `allocatedHeight` → test responsive behaviour
- Different languages → test localisation
- Offline mode → test offline behaviour

---

## Advanced debug: structured logging

```typescript
// Use prefixes to find logs easily in the console
// ⚠️ Use console.error to avoid polluting stdout (important in MCP contexts)
private _log(method: string, message: string, data?: unknown): void {
  console.error(`[MyNS.MyComponent.${method}]`, message, data ?? "");
}

public init(context, notifyOutputChanged, state, container): void {
  this._log("init", "Starting", { value: context.parameters.controlValue.raw });
}

public updateView(context): void {
  this._log("updateView", "Updating", {
    changed: context.updatedProperties,
    value: context.parameters.controlValue.raw,
    disabled: context.mode.isControlDisabled,
  });
}
```
