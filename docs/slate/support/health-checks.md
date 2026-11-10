# Health checks

Run them from **Tools > Handoff for Slate > Check Setup...** (window "Handoff Health Check"), the console command
`SlateHandoff.Health`, Python `unreal.SlateHandoffLibrary.run_health_check()` (list of structs with id, title, status,
detail, fix) or `unreal.SlateHandoffLibrary.health_check_report()` (Markdown),
`import slate_handoff_support; slate_handoff_support.run()`, or the commandlet:

```
UnrealEditor-Cmd.exe <Project>.uproject -run=SlateHandoffImport -Health [-Report=<file.md>]
```

The commandlet exits with code **1** if any check is **Fail**, otherwise 0.

## Statuses

| Status | Meaning |
|---|---|
| **Pass** | Fine. |
| **Info** | Worth knowing, nothing to fix. Often an optional feature that is not in use. |
| **Warning** | Something may not work as you expect or a feature you might use is unavailable. |
| **Fail** | Something that you rely on is broken. Fix it before importing. |

Every check has a stable id (HC01 to HC16) that is also used in the support bundle's `health.json` and in the
[setup health issue form](../../../.github/ISSUE_TEMPLATE/setup_health.yml). The window also has **Re-run checks**, **Copy
summary** (Markdown for the issue form) and **Open issue tracker**.

## The checks

| Id | Title (as shown) | What it checks | Typical fix |
|---|---|---|---|
| HC01 | Plugin and engine version | `SlateHandoffCore` and `SlateHandoffEditor` are loaded and the engine is 5.7 or 5.8. **Fail** if a module is not loaded or the engine is below 5.7, **Warning** above 5.8 (not yet tested). | Reinstall the plugin for this engine version (one package per engine version on Fab), enable **Handoff for Slate** in Edit > Plugins and restart. |
| HC02 | Destination folder | Every destination (source folder) is inside the project and inside a module (nearest `*.Build.cs`). **Fail** otherwise. | Choose a source folder under one of your game or plugin modules, for example `Source/<YourGameModule>/UI/Handoff`. |
| HC03 | Module dependencies | The module's `Build.cs` lists `Slate` and `SlateCore` (and `Projects` for plugin modules). **Fail** if one is missing; the fix shows the line to add. | Add the shown line to `PublicDependencyModuleNames` (or private) in the module's `Build.cs` and rebuild. |
| HC04 | Resources folder | The resources folder (default `Content/Slate/Handoff`) is inside the project (or the owning plugin) and writable. **Fail** otherwise, Info while it has not been created yet. | Point *Resources folder* at a writable folder inside the project, or make the folder writable or check it out from source control. |
| HC05 | Resources staged for packaging | Packaged builds contain the resources folder. **Fail** or **Warning** when they would not: for a project module the folder is not under *Project Settings > Packaging > Additional Non-Asset Directories to Package* or is outside `Content`; for a plugin module it is not declared as `RuntimeDependencies` or is outside the plugin. | Add the folder (for example `Slate/Handoff`) to *Additional Non-Asset Directories to Package*, or declare it in the plugin module's `Build.cs`. See [Fonts and styles](../FontsAndStyles.md). |
| HC06 | Import manifests | Each file in `<Project>/Handoff/Slate/Manifests/` parses and was not written by a newer plugin (**Fail** otherwise). Info when there has been no import yet. | Restore the manifest from source control or **Restore Last Import**; update the plugin if the manifest is from a newer version. See [Re-import rules](../ReimportRules.md). |
| HC07 | Baselines | Every generated file recorded in a manifest has a baseline under `<Project>/Handoff/Slate/Baselines/`. **Warning** if one is missing: a re-import cannot separate your edits from generated lines. | Restore the baselines from source control (commit them with your code) or **Restore Last Import**. |
| HC08 | Edited generated files | Generated files that were edited since the last import (merged on the next import) or are missing. Info only. | Nothing to fix; review the merge in the Changes tab on the next import. |
| HC09 | User stubs | Every user-owned `S<Name>.h/.cpp` recorded in a manifest exists. **Warning** if one is missing: classes that include it do not compile. | Restore the file from source control; the next import writes a fresh stub only for a missing one. |
| HC10 | Font mappings | Every mapping row points at an existing font file. **Warning** lists the unresolved rows. | Fix the path of those rows in the **Fonts** group of the settings, or use *Map file...* in the import window. See [Fonts and styles](../FontsAndStyles.md). |
| HC11 | Font search folders | Every folder in *Font search directories* exists (default `<Project>/Fonts`). Info only; missing folders are listed. | Create the folder or remove it from the list; not needed if you map fonts explicitly. |
| HC12 | Backups | Backups are on and `Saved/SlateHandoffBackups` is writable (**Fail** if not). **Warning** when backups are off or use more than 1 GB. | Turn backups on, make the folder writable, or delete old backup folders. **Restore Last Import** uses only the latest backup of a destination. |
| HC13 | Source control | Provider name and availability, and whether check-out before Apply is on. Info only. | Configure a provider if you want automatic check-out. Commit manifests, baselines and generated code with your project. |
| HC14 | Python Editor Script Plugin | Whether it is enabled. **Needed only for the Python API**; the window, console commands and commandlet work without it. Info only. | Enable it in Edit > Plugins and restart if you want `unreal.SlateHandoffLibrary` / `slate_handoff_support`. |
| HC15 | Figma REST token | Whether a personal access token is set for the optional REST fetch. The value is never shown, logged or bundled. Info only. | Only needed for REST fetch: set it in Editor Preferences > Plugins > Handoff for Slate (User). The exporter plugin needs no token. |
| HC16 | IDE project files | Whether IDE project files exist and a refresh is needed so the build sees new classes. Info only. | Refresh the project files (the import window offers a button) before building after an import that adds classes. |

An unused optional feature is normally reported as **Info**, not a problem. When you file an issue, a Warning or Fail
that matches your problem helps us a lot; use the [setup health form](../../../.github/ISSUE_TEMPLATE/setup_health.yml)
if one of the checks itself looks wrong.
