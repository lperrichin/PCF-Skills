# 07 – Packaging & Deployment

## Overview

```
npm run build          →  out/controls/MyComponent/bundle.js
pac solution init      →  Dataverse solution project
pac solution add-reference →  link component to the solution
msbuild                →  Solutions/bin/debug/solutions.zip
import into Dataverse  →  component available in Power Apps
```

---

## 1. Production build

```bash
# Always build in production mode before packaging
npm run build -- --buildMode production
```

---

## 2. Create the solution project

```bash
# From the PCF project root
mkdir Solutions && cd Solutions

pac solution init \
  --publisher-name MyPublisher \
  --publisher-prefix mypfx

# publisher-name : publisher display name (e.g. "Contoso", "MyCompany")
# publisher-prefix : short prefix (2-8 chars, e.g. "cnt", "myc")
#   → must match an existing publisher in the target environment,
#     or will be created on import
```

### Generated structure

```
Solutions/
├── Other/
│   └── Solution.xml       ← solution metadata
└── solutions.cdsproj      ← MSBuild project file
```

### Verify Solution.xml

```xml
<ImportExportXml>
  <SolutionManifest>
    <UniqueName>solutions</UniqueName>
    <LocalizedNames>
      <LocalizedName description="solutions" languagecode="1033" />
    </LocalizedNames>
    <Version>1.0.0.0</Version>
    <Managed>0</Managed>  <!-- 0=unmanaged, 1=managed -->
    <Publisher>
      <UniqueName>MyPublisher</UniqueName>
      <CustomizationPrefix>mypfx</CustomizationPrefix>
    </Publisher>
  </SolutionManifest>
</ImportExportXml>
```

---

## 3. Add the component to the solution

```bash
# From the Solutions/ folder
pac solution add-reference --path ..\..
# or absolute path:
# pac solution add-reference --path C:\repos\MyComponent

# Expected output:
# Project reference successfully added to Dataverse solution project.
```

---

## 4. Build the solution

```bash
# Restore NuGet packages (first time and after adding components)
msbuild /t:restore
# or with .NET SDK:
dotnet build

# Build the solution (debug mode by default)
msbuild

# Build in release mode (recommended for production)
msbuild /property:configuration=Release
```

### Location of the generated solution

```
Solutions/
└── bin/
    ├── debug/
    │   └── solutions.zip      ← unmanaged solution (debug)
    └── Release/
        └── solutions.zip      ← unmanaged solution (release)
```

### If msbuild is not found

```bash
# Option 1: use the Developer Command Prompt for Visual Studio
#   (search "Developer Command Prompt" in the Start menu)

# Option 2: use dotnet build if .NET 6 SDK is installed
dotnet build

# Option 3: add msbuild to the Windows PATH
# C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin
```

---

## 5. Import into Dataverse

### Via Power Apps UI (manual)

1. Go to **https://make.powerapps.com**
2. Select the target environment
3. Menu **Solutions** → **Import a solution**
4. Select the ZIP file
5. Follow the import wizard
6. ⚠️ For **unmanaged** solutions: publish customisations manually after import

### Via Power Platform CLI (automated)

```bash
# Authenticate against the environment
pac auth create --url https://myorg.crm.dynamics.com

# Import the solution
pac solution import --path Solutions\bin\debug\solutions.zip

# Import with automatic publish
pac solution import --path Solutions\bin\debug\solutions.zip --publish-changes

# Check import status
pac solution list
```

### Via CLI with a connection profile

```bash
# List connection profiles
pac auth list

# Create a profile
pac auth create \
  --name MyEnv \
  --url https://myorg.crm.dynamics.com \
  --username user@domain.com

# Select a profile
pac auth select --index 1

# Import
pac solution import --path Solutions\bin\debug\solutions.zip
```

---

## 6. Publish customisations

Required after importing an **unmanaged** solution:

```bash
pac org publish
```

Or via the UI:
- **https://make.powerapps.com** → Solutions → select the solution → **Publish all customisations**

---

## 7. Add the component to a form

### In a Model-Driven App form

1. Open **https://make.powerapps.com** → Tables → choose the table
2. Forms → open the form to edit
3. Select the field the component should replace
4. Properties panel → **Components** → **Add a component**
5. Select the PCF component from the list
6. Configure `input` properties if needed
7. Save and publish the form

### In a Canvas App

1. Open Power Apps Studio
2. Insert → Custom components → choose the PCF component
3. Bind properties to data sources

---

## 8. Updating an existing component

```bash
# 1. Edit the code
# 2. Increment the version in ControlManifest.Input.xml
#    version="0.0.1" → version="0.0.2"
# 3. Rebuild
npm run build -- --buildMode production

# 4. Rebuild the solution
cd Solutions
msbuild /property:configuration=Release

# 5. Reimport
pac solution import --path Solutions\bin\Release\solutions.zip
pac org publish
```

> ⚠️ Without incrementing the version, Power Apps may serve the component from cache.

---

## 9. CI/CD automation (Power Platform Build Tools)

For Azure DevOps or GitHub Actions pipelines:

```yaml
# Azure DevOps – simplified example
steps:
  - task: microsoft-IsvExpTools.PowerPlatform-BuildTools.build-solution-task@2
    displayName: "Build PCF Solution"
    inputs:
      SolutionSourceFolder: "$(Build.SourcesDirectory)/Solutions"

  - task: microsoft-IsvExpTools.PowerPlatform-BuildTools.import-solution@2
    displayName: "Import Solution"
    inputs:
      authenticationType: "PowerPlatformSPN"
      PowerPlatformSPN: "MyServiceConnection"
      SolutionInputFile: "$(Build.ArtifactStagingDirectory)/solutions.zip"
      PublishCustomizationChanges: true
```

---

## 10. Managed vs unmanaged solution

| Type | Advantage | Disadvantage |
|---|---|---|
| **Unmanaged** | Editable in the target environment | Hard to cleanly uninstall |
| **Managed** | Clean uninstall, protected | Cannot be modified in the target environment |

```bash
# Force a managed solution build
# Edit Solutions/Other/Solution.xml:
# <Managed>1</Managed>
# then rebuild with msbuild
```

---

## Deployment checklist

- [ ] `npm run build -- --buildMode production` completed
- [ ] Version incremented in `ControlManifest.Input.xml`
- [ ] `msbuild /property:configuration=Release` succeeded without critical errors
- [ ] Solution ZIP tested in a staging environment before production
- [ ] Solution imported successfully
- [ ] Customisations published
- [ ] Component tested in the target form / app
