# Handoff for UMG

An Unreal Engine editor plugin that imports a design package into Widget Blueprints and lets you re-import
after the design changes without losing your graphs, bindings or hand edits.

* Works with Figma design through the free companion exporter plugin, shipped ready to load in
  `Extras/FigmaExporter/` of the plugin download. The package format
  is tool-neutral (`Docs/LayoutSchema.md`), so other exporters can target it.
* Output is ordinary UMG: Canvas Panel anchors, Horizontal/Vertical/Wrap boxes, Borders, Images, Text Blocks,
  Size Boxes, Spacers, child Widget Blueprints for components. Optional CommonUI mode generates text styles,
  button styles, `CommonButtonBase` buttons and `CommonActivatableWidget` screens.
* Re-import is a 3-way merge: a property you changed in Unreal is never overwritten unless you choose
  "Take Figma". Every change is shown in a diff window before anything is written.
* Editor only. There is no runtime module and nothing from the plugin ships in your packaged game.

*Figma is a trademark of Figma, Inc. Handoff for UMG is an independent product and is not affiliated with or
endorsed by Figma.*

## Requirements

* Unreal Engine 5.7 or 5.8 (built and tested on 5.8). UE 5.6 is not supported.
* Win64 (Mac and Linux are allow-listed in the `.uplugin` modules).
* Optional: the **Common UI** plugin for CommonUI mode (and Enhanced Input support in Common Input settings for
  `[action=...]` tags on 5.8).
* To export: the Figma desktop app (to load the companion plugin; no build step).
* To fetch through the REST API instead (optional): a Figma personal access token from a Dev or Full seat.

## Install

1. Copy the `HandoffForUMG` folder to `<YourProject>/Plugins/`.
2. Open the project (or regenerate project files and build if it is a C++ project). Enable **Handoff for UMG**
   under Edit > Plugins if it is not already enabled, then restart.
3. Load the Figma exporter once: in the Figma desktop app use Plugins > Development > Import plugin from
   manifest... and pick `Extras/FigmaExporter/manifest.json` inside the installed `HandoffForUMG` folder.
   See `Extras/FigmaExporter/README.md`.

## 5-minute quick start

1. In Figma, select your top-level frames and run **Export for Unreal Engine...** to get a `.handoff.zip`.
2. In Unreal: **Tools > Handoff for UMG > Import Design Package...**, pick the zip.
3. Check the destination (default `/Game/UI/Handoff`), press **Plan**, review the **Changes** tab.
4. Press **Apply**. Widget Blueprints appear as `WBP_<Frame>`; press **Preview** to see them at 1920x1080 etc.
5. Edit the design, export again, import the new package over the same destination. Review conflicts and apply.
   **Restore last import** undoes the whole import from a backup.

Full walkthrough: [Docs/QuickStart.md](QuickStart.md).

## Documentation

| Document | Contents |
|---|---|
| [Docs/QuickStart.md](QuickStart.md) | Step by step: export, import, review, re-import, restore |
| [Docs/FeatureMapping.md](FeatureMapping.md) | Design element -> UMG output, plain and CommonUI mode, tags |
| [Docs/ReimportRules.md](ReimportRules.md) | Identity, re-link, 3-way merge, removals, safety net, **backup and undo caveat** |
| [Docs/FontsAndStyles.md](FontsAndStyles.md) | Font mapping, Google Fonts (OFL), text styles, CommonUI styles |
| [Docs/Automation.md](Automation.md) | Python, commandlet, CI exit codes, `-Explain`, `-Render`, console commands, REST fetch |
| [Docs/Troubleshooting.md](Troubleshooting.md) | Symptom -> cause -> fix |
| [Docs/Support.md](Support.md) | **Check Setup** health checks, **support bundles** and filing an issue |
| [Docs/LayoutSchema.md](LayoutSchema.md) | Package format (`layout.json` schema v1) |
| [Docs/DesignLog.md](DesignLog.md) | Dated clean-room design log |
| [Samples/SCENARIO.md](SCENARIO.md) | Sample packages and the v1 -> v2 re-import scenario |
| [ThirdPartyNotices.txt](ThirdPartyNotices.txt) | Third-party notices |

## Known limits

* Mixed text styles import only the first run. Justified text imports left-aligned. Title case, underline and
  strikethrough have no UMG equivalent. Text shadow blur is ignored.
* Gradients, blurs, boolean operations and vectors are rasterised by the exporter and imported as textures.
* Grid auto layout imports as a WrapBox. Non-circular ellipses without an asset import as pills.
* Rotated container children are approximate when fetched through REST. Display P3 colours are not converted.
* No prototype flows, no UEFN, no runtime module.
