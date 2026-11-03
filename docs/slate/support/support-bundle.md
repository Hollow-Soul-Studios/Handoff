# Support bundle

The support bundle is a zip you create on demand and attach to an issue. **Nothing is uploaded automatically.**

## Creating it

* **Tools > Handoff for Slate > Create Support Bundle...** asks whether to include the full manifests and baselines,
  writes the zip, opens its folder and copies the health check summary to the clipboard. The **Create Support
  Bundle...** button in the **Handoff Health Check** window does the same.
* Console: `SlateHandoff.SupportBundle` (add `manifests` to include full manifests and baselines)
* Python: `unreal.SlateHandoffLibrary.create_support_bundle(False)` returns the zip path (pass `True` to include
  manifests and baselines); `import slate_handoff_support; slate_handoff_support.run()` prints the health checks and
  creates a bundle.
* Commandlet: `UnrealEditor-Cmd.exe <Project>.uproject -run=SlateHandoffImport -SupportBundle [-IncludeManifests] [-Out=<dir>]`

The zip is written to `<Project>/Saved/SlateHandoffSupport/SlateHandoffSupport_<yyyyMMdd-HHmmss>.zip` (or `-Out`).

## Files

| File | Content |
|---|---|
| `report.md` | Human-readable report: versions, the [health checks](health-checks.md) and the list of files. |
| `ISSUE_BODY.md` | Pre-filled issue text in the same order as the bug report form: Plugin version, Engine version, Operating system, Area, Health check summary, What happened?, What did you expect?, Steps to reproduce, Attachments. Fill in the open sections and paste. |
| `health.json` | Machine-readable health check results (id, title, status, detail, fix). |
| `system.json` | Plugin version, engine version, operating system, CPU and GPU names, memory, and the project name as a short hash only. |
| `settings.json` | Handoff project and user settings. The Figma token is **never included**; only `<set, redacted>` or `<not set>`. |
| `plugins.json` | All enabled plugins with their versions. |
| `log_excerpt.txt` | Handoff log lines plus errors and warnings from the current session. Your user home path is replaced by `<home>` and the project path by `<project>`. |
| `last_import_report.txt` | The last import report (`Saved/SlateHandoffSupport/last_import_report.txt`), if there is one. It lists class, member and file names. |
| `manifests_summary.json` | Per import destination: counts of files by kind, missing files and missing baselines. The design file key is reduced to a short hash. **No layer names.** |
| `manifests/`, `baselines/` | Full manifests and the baselines (the text Handoff generated), **only if you opted in**. They contain layer names, node paths, file paths and generated code. |

## What is never included

* Your Figma token (the settings file only shows whether one is set).
* Design packages (`.handoff.zip`), images and fonts.
* Your own source files and the current content of any generated file.

## Redaction

Absolute paths in the log excerpt are shortened: the user home folder becomes `<home>` and the project folder becomes
`<project>`. Redaction is automatic but not a guarantee. Layer, class and frame names appear in the import report and
in the manifests. If your design names are confidential, read the files before attaching.

## Inspecting before you attach

1. Open the zip with any archive tool.
2. Read `ISSUE_BODY.md`, `report.md` and `log_excerpt.txt` in a text editor.
3. Search the files for anything you do not want to share (client names, paths, e-mail addresses).
4. Delete those lines or the files, and say in the issue that you removed them. The report still helps even when
   some files are missing.

If you cannot attach the zip at all, paste the "Health check summary" section of `ISSUE_BODY.md` (or the output of
`SlateHandoff.Health`) into the issue form and explain why.
