# Support

Support is free for buyers of **Handoff for UMG** and **Handoff for Slate**. **Replies come within 2 business days.**

## Before you file an issue

1. Read Troubleshooting and the FAQ of your product ([UMG](docs/umg/Troubleshooting.md),
   [Slate](docs/slate/Troubleshooting.md); FAQ: [UMG](docs/umg/support/faq.md), [Slate](docs/slate/support/faq.md)), and
   search the [existing issues](../../issues?q=is%3Aissue).
2. Update to the latest plugin version from Fab if you are behind. Check the [changelog](CHANGELOG.md).
3. Run the health check (below). Many problems are a missing plugin or module dependency, a missing font mapping, an
   unstaged resources folder (Slate) or a read-only folder, and the check names the fix.

## Filing an issue

1. In Unreal, open **Tools > Handoff for UMG > Check Setup...** or **Tools > Handoff for Slate > Check Setup...**
   (window "Handoff Health Check"). Fix anything marked Warning or Fail if it is relevant. Each check is explained in
   the health checks reference: [UMG](docs/umg/support/health-checks.md), [Slate](docs/slate/support/health-checks.md).
2. Open **Tools > Handoff for UMG > Create Support Bundle...** (or the same entry under **Handoff for Slate**). It
   writes a zip under `<Project>/Saved/`:

   | Product | Bundle path |
   |---|---|
   | Handoff for UMG | `<Project>/Saved/HandoffSupport/HandoffSupport_<yyyyMMdd-HHmmss>.zip` |
   | Handoff for Slate | `<Project>/Saved/SlateHandoffSupport/SlateHandoffSupport_<yyyyMMdd-HHmmss>.zip` |

   **Nothing is uploaded automatically**; you decide whether to attach it.
3. Choose **Issues > New issue**, pick the form that fits (bug, import result, re-import, setup health, feature
   request) and choose the **Product** (Handoff for UMG, Handoff for Slate or Figma exporter). The bundle contains
   `ISSUE_BODY.md`, pre-filled in the same order as the bug form, so you can copy the sections across.
4. Attach the zip to the issue. If you cannot (confidential project), say so in the form and paste the
   "Health check summary" section from `ISSUE_BODY.md`.

## What is in the bundle, and what is not

The bundle holds the health report, plugin, engine, OS and plugin-list information, the Handoff settings, an excerpt of
the editor log and, if present, the last import report. Full contents and how to inspect them:
[UMG](docs/umg/support/support-bundle.md), [Slate](docs/slate/support/support-bundle.md).

Privacy:

* Your Figma token is **never** included. The settings file shows the token only as set or not set.
* Layer names are stored in manifests (Handoff for Slate also keeps file paths and generated code in baselines).
  **Manifests, and baselines for Slate, are left out by default.** Only a manifest *summary* is included (counts and
  asset names or counts per destination, no layer names). Full manifests (and baselines) are added only if you opt in:
  **Include manifests** in the dialog (UMG) or answering Yes in the dialog (Slate), the console `manifests` argument,
  `include_manifests=True` in Python (UMG) or `-IncludeManifests` in the commandlet.
* Handoff for Slate never includes your own source files or the current content of generated files.
* Your home folder path in the log excerpt is replaced by `<home>` and the project path by `<project>`.
* You can open the zip and read every file before you attach it.

## Without the menu

The same actions are available from the console, Python and the command line.

**Handoff for UMG**

| Where | Health check | Support bundle |
|---|---|---|
| Editor console | `Handoff.Health` | `Handoff.SupportBundle` (add `manifests` to include full manifests) |
| Python | `unreal.HandoffLibrary.run_health_check()` or `.health_check_report()` | `unreal.HandoffLibrary.create_support_bundle(include_manifests=False)` |
| Python script | `import handoff_support; handoff_support.run()` | same script |
| Commandlet | `UnrealEditor-Cmd.exe <Project>.uproject -run=HandoffImport -Health` (exit code 1 if any check fails) | `UnrealEditor-Cmd.exe <Project>.uproject -run=HandoffImport -SupportBundle [-IncludeManifests] [-Out=<dir>]` |

**Handoff for Slate**

| Where | Health check | Support bundle |
|---|---|---|
| Editor console | `SlateHandoff.Health` | `SlateHandoff.SupportBundle` (add `manifests` to include full manifests and baselines) |
| Python | `unreal.SlateHandoffLibrary.run_health_check()` or `.health_check_report()` | `unreal.SlateHandoffLibrary.create_support_bundle(False)` |
| Python script | `import slate_handoff_support; slate_handoff_support.run()` | same script |
| Commandlet | `UnrealEditor-Cmd.exe <Project>.uproject -run=SlateHandoffImport -Health` (exit code 1 if any check fails) | `UnrealEditor-Cmd.exe <Project>.uproject -run=SlateHandoffImport -SupportBundle [-IncludeManifests] [-Out=<dir>]` |

## Other channels

* **License, purchase or refund questions:** use the publisher contact on the Fab product page. Do not post purchase
  details in a public issue.
* **Bugs and feature requests:** this issue tracker.

## Support scope

Supported: the plugins as downloaded from Fab, on supported engine versions (see the [README](README.md#support-policy)),
and the companion Figma exporter. Not covered: custom engine builds, other plugins' conflicts we cannot reproduce,
design-tool bugs, general UMG, CommonUI, Slate or C++ questions, and errors in code you wrote in your own
(non-generated) classes.
