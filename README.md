# PCF Plugin for Claude Code

> Build Power Apps Component Framework (PCF) components faster with AI assistance directly in your terminal.

This plugin gives [Claude Code](https://docs.anthropic.com/en/docs/claude-code) deep knowledge of the PCF development workflow — from scaffolding to Dataverse deployment. No server. No compilation. Just copy one folder and start coding.

---

## What it does

When you work on a PCF component, Claude Code automatically knows:

- How to scaffold a `field` or `dataset` control with `pac pcf init`
- The full `ControlManifest.Input.xml` syntax and all Dataverse types
- Every lifecycle method (`init`, `updateView`, `getOutputs`, `destroy`) and the complete `context` API
- How to work with dataset grids: columns, records, pagination, filtering, sorting
- How to call Dataverse CRUD operations via `context.webAPI`
- How to build React / Virtual controls with Fluent UI 8 or 9
- How to build, debug with the PCF test harness, package a solution, and deploy

---

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- A PCF development environment:
  - [Node.js LTS](https://nodejs.org/)
  - [Power Platform CLI](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/powerapps-cli) (`pac`)
  - [.NET Build Tools](https://visualstudio.microsoft.com/downloads/) (Visual Studio or .NET 6 SDK)

---

## Installation

```bash
# Clone the repo
git clone https://github.com/lperrichin/PCF-Skills

# Copy the .claude folder into your PCF project
cp -r pcf-claude-plugin/.claude /path/to/your/pcf-project/

# Or install globally (works across all your projects)
cp -r pcf-claude-plugin/.claude ~/
```

That's it — no `npm install`, no build step, no server to run.

---

## Usage

### Slash commands

Type these directly in Claude Code:

| Command | What it does |
|---|---|
| `/pcf-init` | Scaffold a new PCF component (asks for namespace, name, template, React?) |
| `/pcf-docs [topic]` | Display reference documentation for a specific topic |
| `/pcf-generate` | Generate `ControlManifest.Input.xml`, `index.ts`, and CSS for a component |
| `/pcf-build [phase]` | Get the exact commands for a build phase |

**`/pcf-docs` topics:** `init` · `coding` · `dataset` · `webapi` · `react` · `build` · `packaging`

**`/pcf-build` phases:** `dev` · `debug` · `production` · `package` · `deploy`

### Examples

```
/pcf-init MyCompany RatingControl field
/pcf-docs webapi
/pcf-generate
/pcf-build debug
/pcf-build deploy
```

### Automatic mode

You don't have to use slash commands. The skill triggers automatically whenever
Claude Code detects PCF keywords in your request:

> *"Create a PCF dataset control that displays accounts grouped by city"*
> *"How do I call context.webAPI to update a record?"*
> *"Package my component and deploy it to the dev environment"*

Claude Code will read the relevant reference file and respond with accurate,
context-aware code and commands.

---

## What's inside

```
.claude/
├── commands/                        ← Slash commands
│   ├── pcf-init.md                  → /pcf-init
│   ├── pcf-docs.md                  → /pcf-docs [topic]
│   ├── pcf-generate.md              → /pcf-generate
│   └── pcf-build.md                 → /pcf-build [phase]
└── skills/
    └── pcf/
        ├── SKILL.md                 ← Auto-loaded router (reads matching reference)
        └── references/
            ├── 01-init.md           Scaffold, ControlManifest, Dataverse types
            ├── 02-coding.md         Lifecycle methods, full context API
            ├── 03-dataset.md        Grids, pagination, filtering, sorting
            ├── 04-webapi.md         Dataverse CRUD (create/read/update/delete)
            ├── 05-react.md          React / Virtual controls with Fluent UI
            ├── 06-build-debug.md    Build, PCF test harness, common errors
            └── 07-packaging-deploy.md  Solution packaging, deployment, CI/CD
```

The `SKILL.md` acts as a router: it only loads the reference file(s) relevant to
the current task, keeping the context window lean.

---

## Extending the plugin

Add your own skills alongside the PCF ones:

```
.claude/skills/
├── pcf/                    ← this plugin
└── my-skill/               ← your new skill
    ├── SKILL.md            ← required entry point
    └── references/         ← optional sub-files
        └── my-topic.md
```

Every `SKILL.md` needs a frontmatter header that tells Claude Code when to load it:

```markdown
---
name: my-skill
description: >
  Use this skill when the user asks about X or Y.
  Trigger on: keyword-1, keyword-2, keyword-3.
---

# My skill content
...
```

---

## Reference

| Topic | Microsoft Docs |
|---|---|
| PCF overview | [learn.microsoft.com](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview) |
| Create your first component | [learn.microsoft.com](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript) |
| PCF API reference | [learn.microsoft.com](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/) |
| pac CLI reference | [learn.microsoft.com](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/pcf) |
| PCF samples (GitHub) | [github.com/microsoft/PowerApps-Samples](https://github.com/microsoft/PowerApps-Samples/tree/master/component-framework) |

---

## License

MIT
