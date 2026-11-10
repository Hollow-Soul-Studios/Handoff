# Support bundle

The support bundle is a zip you create on demand and attach to an issue. **Nothing is uploaded automatically.**

## Creating it

* **Tools > Handoff for UMG > Create Support Bundle...** asks whether to include full manifests, writes the zip, opens
  its folder and copies the health check summary to the clipboard. The **Create Support Bundle...** button in the
  **Handoff Health Check** window does the same.
* Console: `Handoff.SupportBundle` (add `manifests` to include full manifests)
* Python: `unreal.HandoffLibrary.create_support_bundle(include_manifests=False)` returns the zip path;
  `import handoff_support; handoff_support.run()` prints the health checks and creates a bundle.
* Commandlet: `UnrealEditor-Cmd.exe <Project>.uproject -run=HandoffImport -SupportBundle [-IncludeManifests] [-Out=<dir>]`

The zip is written to `<Project>/Saved/HandoffSupport/HandoffSupport_<yyyyMMdd-HHmmss>.zip` (or `-Out`).

## Files

| File | Content |
|---|---|
| `report.md` | Human-readable report: the [health checks](health-checks.md) plus the system summary. |
| `ISSUE_BODY.md` | Pre-filled issue text in the same order as the bug report form: Plugin version, Engine version, Operating system, Area, Health check summary, What happened?, What did you expect?, Steps to reproduce, Attachments. Fill in the open sections and paste. |
| `health.json` | Machine-readable health check results (id, title, status, detail, fix). |
| `system.json` | Plugin version, engine version, build configuration, operating system, CPU and GPU names, core count, memory, and the project name as a short hash only. |
| `settings.json` | Handoff project and user settings. The Figma token is **never included**; only `<set, redacted>` or `<not set>`. |
| `plugins.json` | All enabled plugins with version and whether they come from the engine or the project. |
| `log_excerpt.txt` | Handoff log lines plus errors and warnings from the current session log (last 2000 matching lines). Your user home path is replaced by `<home>` and the project path by `<project>`. |
| `last_import_report.txt` | The last import report from `Saved/Handoff/LastImportReport.txt`, if there is one. It lists widget and asset names. |
| `manifests_summary.json` | Per manifest: destination, source tool, schema version, asset counts per kind, widget count, missing assets. The design file key is reduced to a short hash. **No layer names.** |
| `manifests/` | Full manifests, **only if you opted in**. They contain layer names and node paths from your design. |

## What is never included

* Your Figma token (the settings file only shows whether one is set).
* Design packages (`.handoff.zip`), textures, fonts and your `.uasset` files.
* Your project source code.

## Redaction

Absolute paths in the log excerpt are shortened: the user home folder becomes `<home>` and the project folder becomes
`<project>`. Redaction is automatic but not a guarantee. Layer, asset and frame names appear in the import report and
in the manifest summary. If your design names are confidential, read the files before attaching.

## Inspecting before you attach

1. Open the zip with any archive tool.
2. Read `ISSUE_BODY.md`, `report.md` and `log_excerpt.txt` in a text editor.
3. Search the files for anything you do not want to share (client names, paths, e-mail addresses).
4. Delete those lines or the files, and say in the issue that you removed them. The report still helps even when
   some files are missing.

If you cannot attach the zip at all, paste the "Health check summary" section of `ISSUE_BODY.md` (or the output of
`Handoff.Health`) into the issue form and explain why.
