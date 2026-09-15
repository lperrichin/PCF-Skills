Create a new PCF component from scratch.

Arguments: $ARGUMENTS

## What you must do

1. If arguments are incomplete, ask for:
   - **Namespace** TypeScript (e.g. `MyCompany`)
   - **Component name** (e.g. `RatingControl`)
   - **Template**: `field` (field control) or `dataset` (grid/view)
   - **React/Fluent UI?** yes (`-fw react`) or no

2. Read `.claude/skills/pcf/references/01-init.md` for the exact commands.

3. Generate and display:
   - The full `pac pcf init` command
   - A matching `ControlManifest.Input.xml`
   - An `index.ts` with all 4 lifecycle methods
   - An empty CSS file with the correct namespace scoping

4. Create these files in the project if the user confirms.

5. Remind the user to run `npm run refreshTypes` after any manifest change.
