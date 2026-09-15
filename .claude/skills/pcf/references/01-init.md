# 01 – Project Initialisation

## Prerequisites

| Tool | Link / command |
|---|---|
| Visual Studio Code | https://code.visualstudio.com/Download — check **Add to PATH** |
| Node.js LTS | https://nodejs.org/en/download/ |
| Power Platform CLI | VSCode extension **Power Platform Tools** or `winget install Microsoft.PowerAppsCLI` |
| .NET Build Tools | Visual Studio 2019/2022 with `.NET build tools` workload **or** .NET 6 SDK |

Verify installation:
```bash
pac --version
node --version
npm --version
```

---

## Scaffolding the project

```bash
# 1. Create and navigate into the component folder
mkdir MyComponent && cd MyComponent

# 2. Initialise the PCF project
pac pcf init \
  --namespace MyNS \
  --name MyComponent \
  --template field \
  --run-npm-install

# --template options:
#   field   → control bound to a field/column (most common)
#   dataset → control bound to a grid or view (list of records)

# For a React/Fluent UI component, add:
#   --framework react   (or -fw react)
```

### All `pac pcf init` parameters

| Parameter | Shorthand | Description |
|---|---|---|
| `--namespace` | `-ns` | TypeScript namespace (e.g. `SampleNamespace`) |
| `--name` | `-n` | Component class name (e.g. `LinearInputControl`) |
| `--template` | `-t` | `field` or `dataset` |
| `--framework` | `-fw` | Omit = standard JS/TS; `react` = React/Virtual component |
| `--run-npm-install` | `-npm` | Runs `npm install` automatically |

---

## Generated structure

```
MyComponent/                         ← project root
├── MyComponent/                     ← component folder
│   ├── ControlManifest.Input.xml    ← manifest (properties, resources)
│   ├── index.ts                     ← component logic
│   ├── css/                         ← create manually for styles
│   └── generated/
│       └── ManifestTypes.d.ts       ← auto-generated types (DO NOT EDIT)
├── package.json
├── tsconfig.json
└── .eslintrc.json
```

---

## ControlManifest.Input.xml – Full reference

```xml
<?xml version="1.0" encoding="utf-8" ?>
<manifest>
  <control
    namespace="MyNS"
    constructor="MyComponent"
    version="0.0.1"
    display-name-key="MyComponent"
    description-key="MyComponent_Desc"
    control-type="standard">
    <!-- control-type="virtual" for React components -->

    <!-- ── Type group (optional) ──────────────────────────────── -->
    <!-- Allows one property to accept multiple Dataverse types -->
    <type-group name="numbers">
      <type>Whole.None</type>
      <type>Currency</type>
      <type>FP</type>
      <type>Decimal</type>
    </type-group>

    <!-- ── Bound property (read + write) ──────────────────────── -->
    <property
      name="controlValue"
      display-name-key="Control Value"
      description-key="Control value description"
      of-type-group="numbers"
      usage="bound"
      required="true" />

    <!-- ── Input property (read-only config) ──────────────────── -->
    <!-- Configured by the maker in the form, read by the component -->
    <property
      name="maxValue"
      display-name-key="Max Value"
      of-type="Whole.None"
      usage="input"
      required="false" />

    <!-- ── Dataset (dataset template only) ────────────────────── -->
    <data-set name="myDataSet" display-name-key="My Dataset" />

    <!-- ── Resources ──────────────────────────────────────────── -->
    <resources>
      <code path="index.ts" order="1" />
      <css path="css/MyComponent.css" order="1" />
      <!-- Localisation RESX (optional): -->
      <!-- <resx path="strings/MyComponent.1033.resx" version="1.0.0" /> -->
      <!-- React (virtual only): -->
      <!-- <platform-library name="React" version="16.14.0" /> -->
      <!-- <platform-library name="Fluent" version="9.46.2" /> -->
    </resources>

  </control>
</manifest>
```

### `<control>` attributes

| Attribute | Description |
|---|---|
| `namespace` | TypeScript namespace |
| `constructor` | Exact name of the exported class in index.ts |
| `version` | Semver — increment on every deployed update |
| `display-name-key` | i18n key for the name shown in Power Apps |
| `description-key` | i18n key for the description |
| `control-type` | `standard` (vanilla JS/TS) or `virtual` (React) |

### `<property>` attributes

| Attribute | Values / Description |
|---|---|
| `name` | Technical identifier (used in `context.parameters.xxx`) |
| `of-type` | Direct Dataverse type (see table below) |
| `of-type-group` | References a `<type-group>` defined in the manifest |
| `usage` | `bound` = read+write linked to the field; `input` = read-only config |
| `required` | `true` / `false` |

### All Dataverse types (`of-type`)

| Value | Dataverse type | TypeScript raw |
|---|---|---|
| `SingleLine.Text` | Single-line text | `string` |
| `SingleLine.Email` | Email | `string` |
| `SingleLine.URL` | URL | `string` |
| `SingleLine.Phone` | Phone number | `string` |
| `Multiple` | Multi-line text area | `string` |
| `Whole.None` | Integer | `number` |
| `Decimal` | Decimal | `number` |
| `FP` | Floating point | `number` |
| `Currency` | Currency | `number` |
| `TwoOptions` | Yes/No (boolean) | `boolean` |
| `DateAndTime.DateOnly` | Date only | `Date` |
| `DateAndTime.DateAndTime` | Date and time | `Date` |
| `OptionSet` | Choice (single select) | `number` |
| `MultiSelectOptionSet` | Choice (multi-select) | `number[]` |
| `Lookup.Simple` | Lookup field | `ComponentFramework.LookupValue[]` |

---

## Regenerate types after editing the manifest

```bash
npm run refreshTypes
```

This updates `generated/ManifestTypes.d.ts` with the `IInputs` and `IOutputs` interfaces
matching exactly the properties declared in the manifest.

---

## Adding a CSS file

```bash
mkdir MyComponent/css
touch MyComponent/css/MyComponent.css
```

CSS must be **scoped** to the namespace to avoid polluting the host form:

```css
/* Always prefix with MyNS\.MyComponent */
.MyNS\.MyComponent {
  display: flex;
  align-items: center;
}
```

For full CSS details and scoping rules, see `references/02-coding.md`.

---

## Initialise a Git repository (recommended)

```bash
git init
git add .
git commit -m "init: scaffold PCF MyComponent"
```
