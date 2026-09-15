Provide the commands to build, debug, package, or deploy a PCF component.

Arguments: $ARGUMENTS
Possible values: `dev` | `debug` | `production` | `package` | `deploy`

## What you must do

1. Read `.claude/skills/pcf/references/06-build-debug.md` for dev/debug/production.
   Read `.claude/skills/pcf/references/07-packaging-deploy.md` for package/deploy.

2. If no argument is provided, ask for the phase:

   | Phase | Description |
   |---|---|
   | `dev` | Fast development build |
   | `debug` | Launch the PCF test harness → localhost:8181 |
   | `production` | Optimised build before deployment |
   | `package` | Create the Dataverse solution (.zip) |
   | `deploy` | Import into a Dataverse environment |

3. For `package`: if no `Solutions/` folder exists in the project, ask for
   the `publisher-name` and `publisher-prefix`.

4. For `deploy`: ask for the Dataverse organisation URL
   (e.g. `https://myorg.crm.dynamics.com`).

5. Display the exact commands and offer to run them.
