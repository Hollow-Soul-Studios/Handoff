# Quick start

## 1. Export from Figma

1. Load the exporter once: in the Figma desktop app, **Plugins > Development > Import plugin from manifest...**
   and pick `Extras/FigmaExporter/manifest.json` in the installed `HandoffForSlate` folder
   (`Extras/FigmaExporter/README.md`). No build step.
2. Select the top-level frames (or nothing, to export every top-level frame on the page).
3. Run **Plugins > Development > Handoff exporter for Figma design > Export for Unreal Engine...**, keep
   *Image scale* at `1x and 2x`, press **Export .handoff.zip**.
4. Optional tags go in layer names or the plugin's *Layer tags* tab: `[button]`, `[focus]`, `[9slice=16]`,
   `[action=IA_Confirm]`, `[safezone]`, `[inline]`, `[ignore]`, `[rasterize]`, `[widget=Border]`.
   See [CodeGeneration.md](CodeGeneration.md).

You can try the importer without Figma using the package folder `Samples/MainMenu_v1` (see `Samples/SCENARIO.md`):
in the import window use **Folder...**, or pass the folder to `-Package=`. Any `.zip` package works too; the
exporter names its files `*.handoff.zip` by convention.

## 2. Prepare the project

* A C++ project with a game module. The generated code goes into a folder inside a module (the nearest
  `*.Build.cs` above the folder decides which module).
* The module's `Build.cs` must depend on `Slate` and `SlateCore` (plus `Projects` when the module belongs to a
  plugin). If one is missing, **Plan** warns and prints the line to add, for example:
  `PrivateDependencyModuleNames.AddRange(new string[] { "Slate", "SlateCore" });`

## 3. First import

1. **Tools > Handoff for Slate > Import Design Package...**
2. Choose the `.handoff.zip` (a package folder or a `layout.json` also works).
3. Set **Source folder** (project-relative, inside a module; default `Source/<first game module>/UI/Handoff`,
   changeable in Project Settings > Plugins > Handoff for Slate) and, optionally, **Resources folder** (default
   `Content/Slate/Handoff`), **Style class** (default `FHandoffUIStyle`) and **Class prefix** (inserted after
   the `S` of new class names, so `SMainMenu` becomes `SUiMainMenu` with `Ui`).
4. Options: **Safe zone** (wrap every screen in an `SSafeZone`), **Components as classes**, **Backup**,
   **Textures 1x/2x** (2x default).
5. Press **Plan**. Nothing is written yet. Tabs:
   * **Changes** - one tree per generated class, then the other files and the resources. Each file has a
     **View diff** button. Untick *Apply* on a design row to skip it.
   * **Re-link** - see [ReimportRules.md](ReimportRules.md). Empty on a first import.
   * **Fonts** - status per family/style. Unmapped fonts are flagged here: use *Map file...* or
     *Download Google Font (OFL)*. See [FontsAndStyles.md](FontsAndStyles.md).
   * **Messages** - warnings and info (rasterised layers, approximations, missing dependencies).
6. Press **Preview** to look at the screens at 1920x1080, 2560x1440, 1280x800 and 2532x1170 (configurable in
   settings) before anything is compiled.
7. Press **Apply**. Before writing, the files that exist and the manifest are backed up to
   `Saved/SlateHandoffBackups/<yyyyMMdd-HHmmss>/` (setting *Backup before apply*, on by default).

Result:

```
<Source folder>/
  Generated/<Style>.h/.cpp                  style class (fonts, text styles, image brushes)
  Generated/S<Screen>Base.h/.cpp            Handoff-owned, regenerated on every import
  Generated/Components/S<Component>Base.h/.cpp
  S<Screen>.h/.cpp                          yours: written once, never touched again
  Components/S<Component>.h/.cpp            yours
<Resources folder>/
  Images/T_<Name>_<hash6>.png               rasterised layers and image backgrounds
  Fonts/<Family>-<Style>.ttf (or .otf), <Family>-OFL.txt
<Project>/Handoff/Slate/Manifests/<source folder with / replaced by _>.json
<Project>/Handoff/Slate/Baselines/<manifest name>/<project-relative path of each generated file>
```

**Commit the manifest and the baselines with your source.** They are what makes re-import safe.

## 4. Build

New source files need a normal build:

1. Press **Refresh IDE project files** in the result panel (or run Tools > Refresh Visual Studio Project, or
   Generate Project Files from the `.uproject`). Without it the IDE and sometimes UnrealBuildTool do not see the
   new files; if the build ignores them, regenerate project files or build once with `-NoUBTMakefiles`.
2. Build the project from the IDE or the editor. **Live Coding cannot add new classes**: close the editor and
   do a full build when classes were added.

## 5. Use the generated classes

```cpp
#include "UI/Handoff/SMainMenu.h"

ChildSlot
[
    SNew(SMainMenu)
];
```

Edit `SMainMenu.cpp`, the class you own: it derives from `SMainMenuBase`, which declares every named design layer as a
`TSharedPtr<...> fig_<hash>` member. Bind events and data there (for example
`fig_2b3c4d5e->SetOnClicked(...)` on a button class). The `fig_<hash>` member names are frozen, so renaming a
layer in the design never breaks your code. The generated file lists each member with the layer name in a
comment. See [CodeGeneration.md](CodeGeneration.md).

Call `FHandoffUIStyle::Shutdown()` from your module's `ShutdownModule` when you want the style set unregistered
(the style is created and registered lazily on first use).

## 6. Re-import after design changes

1. Export again from Figma (same file; the exporter keeps `source.fileKey` stable).
2. Import over the **same source folder**, press **Plan**.
3. In **Changes**: clean updates are pre-applied; conflicts default to *Keep code*. Toggle a row or a merge
   conflict to *Take Figma*, or use **Keep all code edits** / **Take all Figma**. Use **Conflicts only** and
   **Hide unchanged** to focus. Locked rows cannot be applied.
4. Check **Re-link** if a frame was copy-pasted in Figma (new ids).
5. **Apply**, then build.

## 7. Undo

Use **Tools > Handoff for Slate > Restore Last Import** (or the footer button) to return to the pre-import files.
Your editor's undo does not cover files on disk: see the caveat in
[ReimportRules.md](ReimportRules.md#backup-and-undo-caveat).

## 8. Package a game

Generated images and fonts are plain files that the style class loads at runtime. Stage them: *Project Settings
> Packaging > Additional Non-Asset Directories to Package* gets the folder relative to `Content` (default
`Slate/Handoff`). For a plugin module, declare the resources folder as `RuntimeDependencies` in the module's
`Build.cs` (see [FontsAndStyles.md](FontsAndStyles.md#packaging)). **Check Setup** (HC05) verifies both.

## 9. Write status back to Figma (optional)

After Apply the importer writes `<package>.receipt.json`. In the exporter run **Load import receipt...** to store
per-layer status in Figma plugin data.
