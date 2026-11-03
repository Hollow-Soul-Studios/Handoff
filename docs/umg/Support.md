# Support: health checks and support bundles

Two built-in tools help you fix setup problems yourself and, if that fails, file an issue we can act on.
Nothing is uploaded by either tool.

## Check Setup (health checks)

**Tools > Handoff for UMG > Check Setup...** opens the **Handoff Health Check** window. It runs 16 checks and
shows each one as **Pass**, **Info** (worth knowing, nothing to fix), **Warning** or **Fail**, with the fix next
to it. **Re-run checks** after you change something; **Copy summary** copies the table as Markdown for the
"Health check summary" field of an issue.

| Id | Check | Fail / Warning when |
|---|---|---|
| HC01 | Plugin modules | Fail: `HandoffCore` or `HandoffImporter` is not loaded |
| HC02 | Engine version | Fail below 5.7; Warning above 5.8 |
| HC03 | CommonUI | Fail: CommonUI mode is on but the Common UI plugin / bridge is not active |
| HC04 | Enhanced Input support (CommonUI) | Warning: CommonUI mode is on and Common Input's *Enable Enhanced Input Support* is off |
| HC05 | Input action folder | Warning: the folder is set but empty or missing |
| HC06 | Default destination | Fail: not a valid content path |
| HC07 | Font mappings | Warning: a row points at a missing file or font asset |
| HC08 | Font search folders | Info only (missing folders are listed) |
| HC09 | Import manifests | Fail: a manifest cannot be read; Warning: assets it records are missing |
| HC10 | Write access | Fail: `Handoff/Manifests` or `Saved/HandoffBackups` is not writable |
| HC11 | Backups | Warning: backups use more than 1 GB |
| HC12 | Source control | Info only |
| HC13 | Python Editor Script Plugin | Info only (needed only for the Python API) |
| HC14 | Figma REST token | Info only; the token value is never shown |
| HC15 | Rendering (preview / -Render) | Info only |
| HC16 | Handoff errors this session | Warning: `LogHandoff` logged errors since the editor started |

Other ways to run the checks:

| Where | How |
|---|---|
| Console | `Handoff.Health` (prints the table to the log and opens the window) |
| Python | `unreal.HandoffLibrary.run_health_check()` (list of checks), `unreal.HandoffLibrary.health_check_report()` (Markdown) |
| Commandlet / CI | `UnrealEditor-Cmd.exe Project.uproject -run=HandoffImport -Health [-Report=<file.md>]` (exit code 1 if any check is Fail) |

## Create Support Bundle

**Tools > Handoff for UMG > Create Support Bundle...** (also a button in the health window) asks whether to
include the full import manifests, writes a zip to `<Project>/Saved/HandoffSupport/`, opens that folder and copies
the health check summary to the clipboard.

| File in the zip | Contents |
|---|---|
| `ISSUE_BODY.md` | Issue text in the same order as the GitHub bug report form; fill in the open sections |
| `report.md` | Overview: versions, health checks, list of files |
| `health.json` | The checks, machine-readable |
| `system.json` | Plugin and engine version, OS, CPU, GPU, memory; the project name only as a hash |
| `settings.json` | Handoff project and user settings; the Figma token only as `<set, redacted>` / `<not set>` |
| `plugins.json` | Enabled plugins and versions |
| `log_excerpt.txt` | Handoff lines, errors and warnings of the current session; your home and project folders replaced by `<home>` / `<project>` |
| `last_import_report.txt` | The last import report (`Saved/Handoff/LastImportReport.txt`), if any |
| `manifests_summary.json` | Counts per import destination; design file keys only as hashes |
| `manifests/` | Full manifests, only if you said Yes (they contain layer names and asset paths) |

Never included: your token, design packages, textures, fonts, `.uasset` files or source code. Redaction is
automatic but not a guarantee: open the zip and read it before you attach it.

Other ways to create a bundle:

| Where | How |
|---|---|
| Console | `Handoff.SupportBundle` or `Handoff.SupportBundle manifests` |
| Python | `unreal.HandoffLibrary.create_support_bundle(False)` returns the zip path; `import handoff_support; handoff_support.run()` prints the checks and creates a bundle |
| Commandlet | `UnrealEditor-Cmd.exe Project.uproject -run=HandoffImport -SupportBundle [-IncludeManifests] [-Out=<dir>]` |

## Filing an issue

1. Run **Check Setup...** and try the suggested fixes; see also [Troubleshooting](Troubleshooting.md).
2. Create a support bundle and review it.
3. Open the issue tracker (the **Open issue tracker** button in the health window, or the Support link on the
   plugin's Fab page) and pick the form that matches: bug report, import result, re-import, setup health check,
   or feature request.
4. Paste the sections of `ISSUE_BODY.md` into the form and attach the zip.
