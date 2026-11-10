# Figma exporter

The companion exporter is a free Figma plugin that works with Figma design. It exports the frames you select to a
tool-neutral `.handoff.zip` (`layout.json` plus PNG and optional SVG assets) that the Unreal importer turns into Widget
Blueprints (**Handoff for UMG**) or Slate C++ code (**Handoff for Slate**). The same prebuilt exporter ships with each
plugin download:

| Product | Exporter folder |
|---|---|
| Handoff for UMG | `HandoffForUMG/Extras/FigmaExporter/` |
| Handoff for Slate | `HandoffForSlate/Extras/FigmaExporter/` |

See the README in that folder.

* Works offline: its manifest declares no network access. Nothing is uploaded.
* No REST API, no access token, no Enterprise plan: it reads the file through the Figma Plugin API.
* Exports are deterministic; exporting the same design twice gives the same package (apart from the export timestamp).

*Figma is a trademark of Figma, Inc. The exporter is an independent tool and is not affiliated with or endorsed by
Figma.*

## Install

No build step and no Node.js: the plugin download already contains the built exporter.

1. Find the installed plugin folder: for a Fab install to the engine,
   `<UE install>/Engine/Plugins/Marketplace/HandoffForUMG/` or `.../HandoffForSlate/`; for a project install,
   `<Project>/Plugins/HandoffForUMG/` or `<Project>/Plugins/HandoffForSlate/`.
2. In the Figma desktop app choose **Plugins > Development > Import plugin from manifest...** and pick
   `Extras/FigmaExporter/manifest.json` in that folder.
3. Keep the folder in place: Figma loads `dist/code.js` and `dist/ui.html` next to the manifest.

You only need one copy; a package exported with either copy imports into either product.

## Export

1. Select the top-level frames, components or component sets to export, or select nothing to export every top-level
   frame on the current page.
2. Run **Plugins > Development > Handoff exporter for Figma design > Export for Unreal Engine...**
3. Choose the options and press **Export .handoff.zip**.

| Option | Meaning |
|---|---|
| Image scale | `1x and 2x` (default), `1x only` or `2x only` PNGs for rasterised layers |
| Include SVG | Also write an informational `.svg` for vector layers |
| Only selection | On: the selection (or all frames if nothing is selected). Off: all frames on the page |

The **Warnings** list after an export names layers the importer will approximate or cannot reproduce (mixed text
styles, missing fonts, unsupported effects and so on).

## Tag layers

Tags go in the **Layer tags** tab or in the layer name:

```
Play Button [button] [focus] [action=IA_Confirm]
Panel [9slice=16]            Panel [9slice=12,12,16,16]
HUD [safezone]               Logo [rasterize]              Debug [ignore]
Icon [widget=Image]          Header [inline]
```

What each tag does: Handoff for UMG: [Feature mapping](umg/FeatureMapping.md); Handoff for Slate:
[Code generation](slate/CodeGeneration.md).

## Import receipt (status write-back)

After an import, Unreal writes `<package>.receipt.json`. In the exporter run **Load import receipt...** and choose it.
Each layer then shows its import status and what it maps to in Unreal. This stores a few plugin-data keys on your
nodes; these are the only changes the exporter makes to your Figma file:

| Key | Handoff for UMG | Handoff for Slate |
|---|---|---|
| `handoff.status` | Import status of the layer | Import status of the layer |
| `handoff.ueAsset` | The Unreal asset (Widget Blueprint) the layer maps to | Project-relative path of the generated class header |
| `handoff.ueWidget` | The widget in that asset | The frozen member name in that class |
| `handoff.exportedAt` | Set on exported frames | Set on exported frames |

## Next steps

Import the zip in Unreal: [UMG quick start](umg/QuickStart.md) or [Slate quick start](slate/QuickStart.md).
Re-importing after design changes: [UMG re-import rules](umg/ReimportRules.md) or
[Slate re-import rules](slate/ReimportRules.md).
