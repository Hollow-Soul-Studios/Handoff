# Support: health checks and support bundles

Two built-in tools help you fix setup problems yourself and, if that fails, file an issue we can act on.
Nothing is uploaded by either tool.

## Check Setup (health checks)

**Tools > Handoff for Slate > Check Setup...** opens the **Handoff Health Check** window. It runs 16 checks and
shows each one as **Pass**, **Info** (worth knowing, nothing to fix), **Warning** or **Fail**, with the fix next
to it. **Re-run checks** after you change something; **Copy summary** copies the table as Markdown for the
"Health check summary" field of an issue.

| Id | Check (title) | Fail / Warning when |
|---|---|---|
| HC01 | Plugin and engine version | Fail: `SlateHandoffCore` or `SlateHandoffEditor` is not loaded, or the engine is below 5.7; Warning: engine newer than 5.8 |
| HC02 | Destination folder | Fail: a destination is outside the project or not inside a module (nearest `*.Build.cs`) |
| HC03 | Module dependencies | Fail: the module's `Build.cs` lacks `Slate` / `SlateCore` (and `Projects` for plugin modules); the fix shows the line to add |
| HC04 | Resources folder | Fail: not inside the project (or the owning plugin) or not writable; Info: not created yet |
| HC05 | Resources staged for packaging | Fail / Warning: packaged builds would not contain the resources folder (project module: not staged under *Additional Non-Asset Directories to Package*, or outside `Content`; plugin module: not declared as `RuntimeDependencies`, or outside the plugin) |
| HC06 | Import manifests | Fail: a manifest cannot be read or was written by a newer plugin; Info: no import yet |
| HC07 | Baselines | Warning: a recorded generated file has no baseline (re-imports cannot separate your edits from generated lines) |
| HC08 | Edited generated files | Info only: generated files edited since the last import (merged on the next import) or missing |
| HC09 | User stubs | Warning: a recorded user-owned file is missing (classes that include it do not compile) |
| HC10 | Font mappings | Warning: a mapping row points at a missing font file |
| HC11 | Font search folders | Info only (missing folders are listed) |
| HC12 | Backups | Fail: backups are on but `Saved/SlateHandoffBackups` is not writable; Warning: backups off, or backups use more than 1 GB |
| HC13 | Source control | Info only |
| HC14 | Python Editor Script Plugin | Info only (needed only for the Python API) |
| HC15 | Figma REST token | Info only; the token value is never shown |
| HC16 | IDE project files | Info only: whether IDE project files exist and a refresh is needed for new classes |

Other ways to run the checks:

| Where | How |
|---|---|
| Console | `SlateHandoff.Health` (prints the table to the log and opens the window) |
| Python | `unreal.SlateHandoffLibrary.run_health_check()` (list of checks), `unreal.SlateHandoffLibrary.health_check_report()` (Markdown) |
| Commandlet / CI | `UnrealEditor-Cmd.exe Project.uproject -run=SlateHandoffImport -Health [-Report=<file.md>]` (exit code 1 if any check is Fail) |

## Create Support Bundle

**Tools > Handoff for Slate > Create Support Bundle...** (also a button in the health window) asks whether to
include the full import manifests, writes a zip to `<Project>/Saved/SlateHandoffSupport/`, opens that folder and
copies the health check summary to the clipboard.

| File in the zip | Contents |
|---|---|
| `ISSUE_BODY.md` | Issue text in the same order as the GitHub bug report form; fill in the open sections |
| `report.md` | Overview: versions, health checks, list of files |
| `health.json` | The checks, machine-readable |
| `system.json` | Plugin and engine version, OS, CPU, GPU, memory; the project name only as a hash |
| `settings.json` | Handoff project and user settings; the Figma token only as `<set, redacted>` / `<not set>` |
| `plugins.json` | Enabled plugins and versions |
| `log_excerpt.txt` | Handoff lines, errors and warnings of the current session; your home and project folders replaced by `<home>` / `<project>` |
| `last_import_report.txt` | The last import report (`Saved/SlateHandoffSupport/last_import_report.txt`), if any |
| `manifests_summary.json` | Counts per import destination (files by kind, missing files and baselines); design file keys only as hashes |
| `manifests/`, `baselines/` | Full manifests and the baselines (the text Handoff generated), only if you said Yes (they contain layer names, file paths and generated code) |

Never included: your token, design packages, images, fonts, your own source files or the current content of any
generated file. Redaction is automatic but not a guarantee: open the zip and read it before you attach it.

Other ways to create a bundle:

| Where | How |
|---|---|
| Console | `SlateHandoff.SupportBundle` or `SlateHandoff.SupportBundle manifests` |
| Python | `unreal.SlateHandoffLibrary.create_support_bundle(False)` returns the zip path; `import slate_handoff_support; slate_handoff_support.run()` prints the checks and creates a bundle |
| Commandlet | `UnrealEditor-Cmd.exe Project.uproject -run=SlateHandoffImport -SupportBundle [-IncludeManifests] [-Out=<dir>]` |

## Filing an issue

1. Run **Check Setup...** and try the suggested fixes; see also [Troubleshooting](Troubleshooting.md).
2. Create a support bundle and review it.
3. Open the issue tracker (the **Open issue tracker** button in the health window, or the Support link on the
   plugin's Fab page) and pick the form that matches: bug report, import result, re-import, setup health check,
   or feature request.
4. Paste the sections of `ISSUE_BODY.md` into the form and attach the zip.
