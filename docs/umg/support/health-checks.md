# Health checks

Run them from **Tools > Handoff for UMG > Check Setup...** (window "Handoff Health Check"), the console command
`Handoff.Health`, Python `unreal.HandoffLibrary.run_health_check()` (list of structs with id, title, status, detail,
fix) or `unreal.HandoffLibrary.health_check_report()` (Markdown), `import handoff_support; handoff_support.run()`, or
the commandlet:

```
UnrealEditor-Cmd.exe <Project>.uproject -run=HandoffImport -Health
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
[setup health issue form](../../.github/ISSUE_TEMPLATE/setup_health.yml).

## The checks

| Id | Title (as shown) | What it checks | Typical fix |
|---|---|---|---|
| HC01 | Plugin modules | `HandoffCore` and `HandoffImporter` are loaded (`HandoffCommonUI` is reported but optional). **Fail** if a required module is missing. | Reinstall the plugin for this engine version (Fab Library > Install to Engine), enable **Handoff for UMG** in Edit > Plugins and restart. |
| HC02 | Engine version | The running engine is 5.7 or 5.8. **Fail** below 5.7, **Warning** above 5.8 (not yet tested). | Use the plugin package built for your engine version (one package per engine version on Fab). |
| HC03 | CommonUI | Whether the Common UI plugin is enabled and the `HandoffCommonUI` bridge is active. **Fail only if CommonUI mode is on and the bridge is missing**; Info when CommonUI is unused. | Enable **Common UI** in Edit > Plugins and restart, or turn CommonUI mode off in Project Settings > Plugins > Handoff for UMG. |
| HC04 | Enhanced Input support (CommonUI) | *Enable Enhanced Input Support* in Project Settings > Common Input. **Warning** only when CommonUI mode is on and it is off (buttons with `[action=...]` tags would not react to the action). | Turn on Project Settings > Common Input > Enable Enhanced Input Support. |
| HC05 | Input action folder | Info when not set (`[action=...]` searches all of `/Game`); **Warning** when set but empty or missing. | Point *Input Action Folder* at the folder holding your `UInputAction` assets. |
| HC06 | Default destination | The default destination is a valid content path (default `/Game/UI/Handoff`). **Fail** otherwise. | Set a valid `/Game/...` folder in Project Settings > Plugins > Handoff for UMG. |
| HC07 | Font mappings | Every mapping row points at an existing font file or font asset. **Warning** lists the unresolved rows; Info when there are no rows. | Fix the path or font asset of those rows in the **Fonts** group of the settings, or use *Map file...* in the import window. See [Fonts and styles](../FontsAndStyles.md). |
| HC08 | Font search folders | Every folder in *Font search directories* exists (default `<Project>/Fonts`). Info when one is missing. | Create the folder or remove it from the list; not needed if you map fonts explicitly. |
| HC09 | Import manifests | Each file in `<Project>/Handoff/Manifests/` parses (**Fail** otherwise) and the assets it records exist (**Warning** otherwise). | Restore the manifest from source control or **Restore Last Import**. A missing asset is recreated by the next import; see [Re-import rules](../ReimportRules.md). |
| HC10 | Write access | The plugin can write to `<Project>/Handoff/Manifests` and `<Project>/Saved/HandoffBackups`. **Fail** otherwise. | Make the folders writable or check the files out from source control. |
| HC11 | Backups | Number and total size of backups in `Saved/HandoffBackups`, and the latest one. **Warning above 1 GB.** | Delete old backup folders you no longer need. **Restore Last Import** uses only the latest backup of a destination. |
| HC12 | Source control | Provider name and availability, and whether check-out before Apply is on. Info only. | Configure a provider if you want automatic check-out. Commit manifests with your assets. |
| HC13 | Python Editor Script Plugin | Whether it is enabled. **Needed only for the Python API**; the window, console commands and commandlet work without it. Info only. | Enable it in Edit > Plugins and restart if you want `unreal.HandoffLibrary` / `handoff_support`. |
| HC14 | Figma REST token | Whether a personal access token is set for the optional REST fetch. The value is never shown, logged or bundled. Info only. | Only needed for REST fetch: set it in Editor Preferences > Plugins > Handoff for UMG (User). The exporter plugin needs no token. |
| HC15 | Rendering (preview / -Render) | Whether this process can render (preview window, commandlet `-Render`). Info when it cannot. | Run the editor with a GPU; for the commandlet drop `-nullrhi` and pass `-AllowCommandletRendering`. |
| HC16 | Handoff errors this session | Counts `LogHandoff: Error` lines in the current session log. **Warning** if there are any. | Filter the Output Log by `LogHandoff`, read the first error, check [Troubleshooting](../Troubleshooting.md); attach a support bundle if it persists. |

An unused optional feature is normally reported as **Info**, not a problem. When you file an issue, a Warning or Fail
that matches your problem helps us a lot; use the [setup health form](../../.github/ISSUE_TEMPLATE/setup_health.yml)
if one of the checks itself looks wrong.
