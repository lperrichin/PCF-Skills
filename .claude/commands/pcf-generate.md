Generate the files for a PCF component: ControlManifest.Input.xml, index.ts, and CSS.

Arguments: $ARGUMENTS

## What you must do

1. Read `.claude/skills/pcf/references/01-init.md` and `.claude/skills/pcf/references/02-coding.md`.

2. If arguments are incomplete, ask for:
   - **Namespace** (e.g. `MyCompany`)
   - **Component name** (e.g. `ColorPicker`)
   - **Main property name** (e.g. `value`)
   - **Dataverse type**:
     `SingleLine.Text` | `Whole.None` | `Decimal` | `Currency` | `FP` |
     `TwoOptions` | `DateAndTime.DateOnly` | `DateAndTime.DateAndTime` |
     `OptionSet` | `MultiSelectOptionSet` | `Lookup.Simple` | `Multiple`
   - **Template**: `field` or `dataset`
   - **React?** yes/no

3. Generate the 3 complete files:
   - `ControlManifest.Input.xml` — with correct `of-type`, `usage="bound"`, resources
   - `index.ts` — with all 4 lifecycle methods, correctly typed for the Dataverse type
   - `css/<Name>.css` — with namespace scoping `.Namespace\.<Name> { ... }`

4. Offer to create these files in the `<Name>/` folder of the project.
