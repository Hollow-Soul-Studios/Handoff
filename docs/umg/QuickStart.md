# Quick start

## 1. Export from Figma

1. Load the exporter once: in the Figma desktop app, **Plugins > Development > Import plugin from manifest...**
   and pick `Extras/FigmaExporter/manifest.json` in the installed `HandoffForUMG` folder
   (`Extras/FigmaExporter/README.md`). No build step.
2. Select the top-level frames (or nothing, to export every top-level frame on the page).
3. Run **Plugins > Development > Handoff exporter for Figma design > Export for Unreal Engine...**, keep
   *Image scale* at `1x and 2x`, press **Export .handoff.zip**.
4. Optional tags go in layer names or the plugin's *Layer tags* tab: `[button]`, `[focus]`, `[9slice=16]`,
   `[action=IA_Confirm]`, `[safezone]`, `[inline]`, `[ignore]`, `[rasterize]`, `[widget=Border]`.
   See [FeatureMapping.md](FeatureMapping.md).

You can try the importer without Figma using `Samples/MainMenu_v1.zip` (see `Samples/SCENARIO.md`). Any `.zip`
package works; the exporter names its files `*.handoff.zip` by convention.

## 2. First import

1. **Tools > Handoff for UMG > Import Design Package...**
2. Choose the `.handoff.zip` (a package folder or a `layout.json` also works).
3. Set **Destination** (default `/Game/UI/Handoff`, changeable in Project Settings > Plugins > Handoff for UMG).
4. Options: **CommonUI** (needs the Common UI plugin), **Safe zone** (wrap every screen), **Components as
   widgets**, **Textures 1x/2x** (2x default).
5. Press **Plan**. Nothing is written yet. Tabs:
   * **Changes** - one tree per asset. Untick *Apply* on a row to skip it.
   * **Re-link** - see [ReimportRules.md](ReimportRules.md). Empty on a first import.
   * **Fonts** - status per family/style. Unmapped fonts are flagged here: use *Map file...* or
     *Download Google Font (OFL)*. See [FontsAndStyles.md](FontsAndStyles.md).
   * **Messages** - warnings and info (rasterised layers, approximations, missing input actions).
6. Press **Apply**. Before writing, the affected `.uasset` files and the manifest are backed up to
   `Saved/HandoffBackups/<yyyyMMdd-HHmmss>/` (setting *Backup before apply*, on by default).

Result under the destination:

```
WBP_<Frame>                     one Widget Blueprint per top-level frame
Components/WBP_<Component|Set>  components and button sets
Textures/T_<Name>_<hash6>       rasterised layers and image backgrounds
Fonts/FF_<Family>_<Style>, F_<Family>, <Family>_OFL.txt
Styles/TS_*, BS_*               CommonUI mode only
```

The manifest is written to `<Project>/Handoff/Manifests/`. **Commit it with your assets**: it is what makes
re-import safe.

## 3. Preview

**Preview** (or console `Handoff.Preview /Game/UI/Handoff/WBP_MainMenu`) opens a window with a widget picker
and the resolutions 1920x1080, 2560x1440, 1280x800 and 2532x1170 (configurable in settings), rendered with
the project DPI curve.

## 4. Build your UI logic

Open the Widget Blueprints and add graphs, bindings, animations, extra widgets as usual. Layer names appear as
the widgets' display labels, so the hierarchy reads like Figma. Widget names are frozen (`fig_<hash>`) so
renames in Figma never break your references; you may rename a widget in Unreal and the importer still follows
it.

## 5. Re-import after design changes

1. Export again from Figma (same file; the exporter keeps `source.fileKey` stable).
2. Import over the **same destination**, press **Plan**.
3. In **Changes**: clean updates are pre-ticked; conflicts default to *Keep Unreal*. Toggle a row to
   *Take Figma*, or use **Keep all Unreal edits** / **Take all Figma**. Use **Conflicts only** and
   **Hide unchanged** to focus. Locked rows cannot be applied.
4. Check **Re-link** if a frame was copy-pasted in Figma (new ids).
5. **Apply**.

## 6. Undo

Use **Tools > Handoff for UMG > Restore Last Import** (or the footer button) to return to the pre-import files.
Ctrl+Z does **not** reliably undo an import: see the caveat in [ReimportRules.md](ReimportRules.md#backup-and-undo-caveat).

## 7. Write status back to Figma (optional)

After Apply the importer writes `<package>.receipt.json`. In the exporter run **Load import receipt...** to store
per-layer status in Figma plugin data.
