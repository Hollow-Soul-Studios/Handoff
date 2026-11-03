# Changelog

All notable changes to Handoff for UMG, Handoff for Slate and the shared Figma exporter. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/). Plugin versions use `MAJOR.MINOR.PATCH` and are numbered per product.

## Handoff for UMG

### 1.0.0

First public release. Supports Unreal Engine 5.8 (tested) and 5.7 (one plugin package per engine version).

#### Import
* Imports a `.handoff.zip` design package (also a package folder or a `layout.json`) into Widget Blueprints, one
  `WBP_<Frame>` per top-level frame.
* Constraints become Canvas Panel anchors; auto layout becomes Horizontal, Vertical or Wrap boxes; rectangles become
  Border, Image or Size Box; text becomes Text Block; spacers and components (child Widget Blueprints, or inline with
  `[inline]`) are supported.
* Texture import with UI settings, 1x or 2x; 9-slice margins (`[9slice=...]`); safe-zone wrapper (`[safezone]` or
  project option).
* Layer tags: `[button]`, `[focus]`, `[9slice=...]`, `[action=...]`, `[safezone]`, `[inline]`, `[ignore]`,
  `[rasterize]`, `[widget=...]`.
* Plan step: **Plan** shows every change in a Changes / Re-link / Fonts / Messages window before **Apply** writes
  anything.
* Preview window at 1920x1080, 2560x1440, 1280x800 and 2532x1170 (configurable), rendered with the project DPI curve.

#### Re-import
* Three-way merge per property (last import, current asset, new package); Unreal-side edits are kept by default.
* Identity by node id, with automatic re-link by hierarchy path, layer name, type and geometry above a strict
  threshold, and a **Re-link** tab for suggestions and manual links.
* Widgets referenced by graphs, bindings, animations or `BindWidget` members are locked and never removed; widgets you
  added by hand are never touched.
* Container swaps keep slot edits made in Unreal where both slot types share the field.
* Deterministic manifest in `<Project>/Handoff/Manifests/` (commit it with your assets).
* Safety net: backup of affected `.uasset` files and the manifest to `Saved/HandoffBackups/<timestamp>/` before apply,
  **Restore last import**, optional source control check-out.

#### Fonts and styles
* Font mapping table (composite font assets or TTF/OTF files), search folders, reuse of earlier imports; missing
  fonts are flagged, never silently replaced. No fonts are bundled.
* Optional, user-triggered download of Google Fonts that are published under the SIL OFL, saved with the OFL text.
* CommonUI mode: `UCommonTextBlock` with generated `UCommonTextStyle` subclasses, button sets as `UCommonButtonBase`
  subclasses with `UCommonButtonStyle`, `UCommonActivatableWidget` screens, default focus, `[action=...]` input actions
  (Enhanced Input path on 5.8; legacy data table path on 5.7).

#### Automation
* Python API `unreal.HandoffLibrary` (import, plan, restore, Google Font download, REST fetch).
* Commandlet `-run=HandoffImport` with `-Plan`, `-Explain`, `-Render`, `-Relink`, `-Report`, `-FailOnConflicts`,
  `-Restore` and CI exit codes (0, 1, 2).
* Console commands `Handoff.Import`, `Handoff.Preview`, `Handoff.SaveImportWindowShot`, `Handoff.SaveWindowShot`.
* Optional REST fetch from Figma for Dev and Full seats (token stored per user, never in the project).
* Import receipts (`<package>.receipt.json`) that the exporter can write back to Figma plugin data.

#### Support tools
* **Tools > Handoff for UMG > Check Setup...** (16 health checks, HC01 to HC16) and **Create Support Bundle...**.
* Console `Handoff.Health` and `Handoff.SupportBundle`, Python `run_health_check()`, `health_check_report()` and
  `create_support_bundle()`, `Content/Python/handoff_support.py`, commandlet `-Health` and `-SupportBundle`.

#### Samples and documentation
* `MainMenu_v1.zip` and `MainMenu_v2.zip` sample packages and the v1 to v2 re-import scenario.
* Quick start, feature mapping, re-import rules, fonts and styles, automation, troubleshooting, layout schema, design
  log.

#### Known limits
See the [plugin README](docs/umg/plugin-readme.md#known-limits).

## Handoff for Slate

### 1.0.0

First public release. Supports Unreal Engine 5.8 (tested) and 5.7 (one plugin package per engine version).

#### Import
* Imports a `.handoff.zip` design package (also a package folder or a `layout.json`) into Slate C++ source code: one
  `SCompoundWidget` class per top-level frame and per component, plus one style class per destination.
* Generation gap: `Generated/S<Name>Base.h/.cpp` is owned by Handoff and regenerated on every import;
  `S<Name>.h/.cpp` is yours, written only when missing and never touched again.
* Constraints become `SConstraintCanvas` anchors; auto layout becomes `SHorizontalBox`, `SVerticalBox` or `SWrapBox`;
  rectangles become `SBorder`, `SImage` or `SBox`; text becomes `STextBlock`; buttons become `SButton`; spacers and
  components are supported. Generated code depends only on `Slate` and `SlateCore`.
* Images and fonts are copied as plain files into a resources folder (default `Content/Slate/Handoff`) and loaded
  through the generated style class; 1x or 2x; 9-slice margins (`[9slice=...]`); safe-zone wrapper (`[safezone]` or
  project option).
* Layer tags: `[button]`, `[focus]`, `[9slice=...]`, `[action=...]`, `[safezone]`, `[inline]`, `[ignore]`,
  `[rasterize]`, `[widget=...]`. See [code generation](docs/slate/CodeGeneration.md).
* Plan step: **Plan** shows every change in a Changes / Re-link / Fonts / Messages window before **Apply** writes
  anything.
* Preview window builds the screens from the plan with live widgets at 1920x1080, 2560x1440, 1280x800 and 2532x1170
  (configurable).

#### Re-import
* Line-level 3-way merge of generated files against a baseline of what Handoff generated last time; hand edits are
  kept by default and conflicts default to **Keep code**.
* Identity by node id, with automatic re-link by hierarchy path, layer name, type and geometry above a strict
  threshold, and a **Re-link** tab for suggestions and manual links.
* Members referenced by your own code are locked: never removed (kept as a detached placeholder) and a class change
  becomes a conflict instead of being applied silently. Code you added by hand is never touched.
* Deterministic manifest in `<Project>/Handoff/Slate/Manifests/` and baselines in
  `<Project>/Handoff/Slate/Baselines/` (commit them with your code).
* Safety net: backup of affected files to `Saved/SlateHandoffBackups/<timestamp>/` before apply, **Restore last
  import**, optional source control check-out.

#### Fonts and styles
* Font mapping table (TTF/OTF files), search folders, reuse of earlier imports; missing fonts are flagged, never
  silently replaced. No fonts are bundled.
* Optional, user-triggered download of Google Fonts that are published under the SIL OFL, saved with the OFL text.
* The generated style class holds fonts, text styles and image brushes.

#### Automation
* Python API `unreal.SlateHandoffLibrary` (import, plan, restore, Google Font download, REST fetch).
* Commandlet `-run=SlateHandoffImport` with `-Plan`, `-Explain`, `-Render`, `-Relink`, `-Report`, `-FailOnConflicts`,
  `-Restore` and CI exit codes (0, 1, 2).
* Console commands `SlateHandoff.Import`, `SlateHandoff.Preview`, `SlateHandoff.Health`, `SlateHandoff.SupportBundle`,
  `SlateHandoff.SaveImportWindowShot`, `SlateHandoff.SaveWindowShot`.
* Optional REST fetch from Figma for Dev and Full seats (token stored per user, never in the project).
* Import receipts (`<package>.receipt.json`) that the exporter can write back to Figma plugin data.

#### Support tools
* **Tools > Handoff for Slate > Check Setup...** (16 health checks, HC01 to HC16) and **Create Support Bundle...**.
* Console `SlateHandoff.Health` and `SlateHandoff.SupportBundle`, Python `run_health_check()`,
  `health_check_report()` and `create_support_bundle()`, commandlet `-Health` and `-SupportBundle`.

#### Samples and documentation
* `MainMenu_v1.zip` and `MainMenu_v2.zip` sample packages and the v1 to v2 re-import scenario.
* Quick start, code generation, re-import rules, fonts and styles, automation, troubleshooting, layout schema, design
  log.

#### Known limits
See the [plugin README](docs/slate/plugin-readme.md#known-limits).

## Figma exporter

Shared by both products; each plugin download ships the same prebuilt exporter in `Extras/FigmaExporter`.

### 1.0.0

* Free Figma exporter plugin, shipped prebuilt in `Extras/FigmaExporter` (no build step): exports selected frames to
  `.handoff.zip` with PNG/SVG assets, deterministic output, layer tag editor, import receipt write-back. Works offline
  and uses no REST API.
