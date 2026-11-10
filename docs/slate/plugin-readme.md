# Handoff for Slate

An Unreal Engine editor plugin that turns a design package into Slate C++ source code and lets you re-import
after the design changes without losing your own code or your hand edits.

* Works with Figma design through the free companion exporter plugin, shipped ready to load in
  `Extras/FigmaExporter/` of the plugin download. The package format is tool-neutral
  (`Docs/LayoutSchema.md`), so other exporters can target it.
* Output is ordinary Slate code: one `SCompoundWidget` class per screen and component, built with
  `SNew` / `SAssignNew`, using `SConstraintCanvas` anchors, `SHorizontalBox` / `SVerticalBox` / `SWrapBox`,
  `SBorder`, `SImage`, `STextBlock`, `SBox`, `SSpacer` and `SButton`. A generated style class holds fonts, text
  styles and image brushes.
* Generation gap: every class is split into `Generated/S<Name>Base.h/.cpp` (owned by Handoff, regenerated on
  every import) and `S<Name>.h/.cpp` (yours: created once, never touched again). Put your logic in your class.
* Re-import is a 3-way merge. Hand edits inside generated files are merged line by line against a baseline of
  what Handoff generated last time; conflicts default to **Keep code**. Members your code uses are never removed
  and never silently change widget class. Every change is shown in a diff window before anything is written.
* A preview window builds the screens from the plan with live widgets, so you can look at a design before it is
  compiled.
* Editor only. There is no runtime module and nothing from the plugin ships in your packaged game. The
  generated code is yours and depends only on `Slate` and `SlateCore`.

*Figma is a trademark of Figma, Inc. Handoff for Slate is an independent product and is not affiliated with or
endorsed by Figma.*

## Requirements

* Unreal Engine 5.7 or 5.8 (built and tested on 5.8). UE 5.6 is not supported.
* Win64 (Mac and Linux are allow-listed in the `.uplugin` modules).
* A C++ project: the generated classes go into one of your game modules (or a plugin module) and are compiled
  by your normal build. A Visual Studio / Rider / Xcode toolchain is needed to build the generated code.
* The module that receives the code depends on `Slate` and `SlateCore` (plus `Projects` for plugin modules).
  **Check Setup** shows the exact line to add if one is missing.
* To export: the Figma desktop app (to load the companion plugin; no build step).
* To fetch through the REST API instead (optional): a Figma personal access token from a Dev or Full seat.

## Install

1. Copy the `HandoffForSlate` folder to `<YourProject>/Plugins/`.
2. Regenerate project files and build the project. Enable **Handoff for Slate** under Edit > Plugins if it is not
   already enabled, then restart.
3. Load the Figma exporter once: in the Figma desktop app use Plugins > Development > Import plugin from
   manifest... and pick `Extras/FigmaExporter/manifest.json` inside the installed `HandoffForSlate` folder.
   See `Extras/FigmaExporter/README.md`.

## 5-minute quick start

1. In Figma, select your top-level frames and run **Export for Unreal Engine...** to get a `.handoff.zip`.
2. In Unreal: **Tools > Handoff for Slate > Import Design Package...**, pick the zip.
3. Check the **Source folder** (default `Source/<YourGameModule>/UI/Handoff`), press **Plan**, review the
   **Changes** tab and **Preview** the screens.
4. Press **Apply**. Handoff writes the classes (`Generated/SMainMenuBase.h/.cpp` plus your `SMainMenu.h/.cpp`),
   the style class and the resources (`Content/Slate/Handoff`).
5. Refresh the IDE project files (the window offers a button), build the project (not Live Coding: it cannot add
   classes), and use the class: `SNew(SMainMenu)`.
6. Edit the design, export again, import the new package over the same source folder. Review conflicts and
   apply. **Restore last import** undoes the whole import from a backup.
7. Packaging a game: stage the resources folder (*Project Settings > Packaging > Additional Non-Asset Directories
   to Package*, `Slate/Handoff`). **Check Setup** verifies it.

Full walkthrough: [Docs/QuickStart.md](QuickStart.md).

## Documentation

| Document | Contents |
|---|---|
| [Docs/QuickStart.md](QuickStart.md) | Step by step: export, import, review, build, use, re-import, restore, package |
| [Docs/CodeGeneration.md](CodeGeneration.md) | Design element -> Slate code, files and names, tags, the style class, widget tree specs |
| [Docs/ReimportRules.md](ReimportRules.md) | Identity, re-link, line-level merge with baselines, locks, type changes, safety net, **backup and undo caveat** |
| [Docs/FontsAndStyles.md](FontsAndStyles.md) | Font mapping, Google Fonts (OFL), the style class, text styles, image brushes |
| [Docs/Automation.md](Automation.md) | Python, commandlet, CI recipe and exit codes, `-Explain`, `-Render`, console commands, REST fetch |
| [Docs/Troubleshooting.md](Troubleshooting.md) | Symptom -> cause -> fix |
| [Docs/Support.md](Support.md) | **Check Setup** health checks HC01..HC16, **support bundles** and filing an issue |
| [Docs/LayoutSchema.md](LayoutSchema.md) | Package format (`layout.json` schema v1) |
| [Docs/DesignLog.md](DesignLog.md) | Dated clean-room design log |
| [Samples/SCENARIO.md](SCENARIO.md) | Sample packages and the v1 -> v2 re-import scenario |
| [ThirdPartyNotices.txt](ThirdPartyNotices.txt) | Third-party notices |

## Known limits

* Per-state text colours of buttons are not generated: the text of a button class uses the default variant's
  colour. Input actions (`[action=...]`) are reported but not generated; bind input in your own code.
* Mixed text styles import only the first run. Justified text imports left-aligned. Title case, underline and
  strikethrough have no Slate equivalent in the generated code. Text shadow blur is ignored.
* Gradients, blurs, boolean operations and vectors are rasterised by the exporter and imported as images.
* Grid auto layout imports as an `SWrapBox`. Non-circular ellipses without an asset import as pills.
* Fonts that are not mapped fall back to the engine default font at runtime.
* Rotated container children are approximate when fetched through REST. Display P3 colours are not converted.
* The generated code needs a normal build: Live Coding cannot add the new classes, and UnrealBuildTool may need
  refreshed project files to see them.
* No prototype flows, no UEFN, no runtime module, no UMG assets (use the separate Handoff for UMG product for
  Widget Blueprints).
