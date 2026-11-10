# Troubleshooting

Start with the **Messages** tab (or the commandlet report / `Result.Report`). `-Explain` on the commandlet lists
every planned change and file action, including merge conflicts. **Check Setup** (Tools > Handoff for Slate)
finds most setup problems; see [Support.md](Support.md).

## Build and project setup

| Symptom | Cause | Fix |
|---|---|---|
| The import succeeded but the new classes do not compile / "class not found" / the new files are not in the build | New source files need regenerated project files; Live Coding cannot add new classes | Press **Refresh IDE project files** (or Generate Project Files from the `.uproject`), then do a full build with the editor closed. If UnrealBuildTool still ignores the files, build once with `-NoUBTMakefiles` |
| Unresolved external symbol / "cannot open include file" for Slate headers (`Widgets/...`, `Styling/...`) | The module's `Build.cs` lacks `Slate` / `SlateCore` (and `Projects` for plugin modules) | Add the line **Plan** printed: "Module X does not depend on Slate, SlateCore. Add to ...Build.cs: PrivateDependencyModuleNames.AddRange(new string[] { ... });" HC03 shows it too |
| "No module (*.Build.cs) found at or above 'X'. Pick a folder inside a game or plugin module, e.g. Source/MyGame/UI/Handoff." | The source folder is not below a module | Choose a folder inside a module (the nearest `*.Build.cs` decides the module) |
| "Invalid destination folder: ..." / "Invalid resources folder: ..." | Path outside the project or malformed | Use a project-relative path |
| "The module X belongs to the plugin Y, so the resources folder must be inside the plugin (...)" | Plugin module with a resources folder outside the plugin | Use a folder below the plugin, e.g. `Plugins/Y/Resources/Handoff` |
| Include errors for `#include "SMainMenuBase.h"` or `"../Components/..."` | Generated includes are relative to the including file; a moved or copied generated file breaks them | Leave generated files where they are; put your code in your own class |
| Class name clash: `SButton`, `SBorder`... "redefinition" | A design frame/component named like an engine or project Slate widget | New classes get a free name automatically (`SButton` -> `SButtonWidget`, then `SButtonWidget2`...); to avoid it from the start set **Class prefix** (`Ui` -> `SUiButton`) in the window or `ClassPrefix` in settings. Existing classes keep their names |
| "The file exists and was not written by Handoff; it is not overwritten. Move or delete it, or choose another destination or class name." | A file at a generated path exists and the manifest has no record of it | Move or delete it, change the destination or class prefix. Restoring the manifest from source control also works when the file was Handoff's |
| "X and another class generate the same file." | Two classes map to one path | Rename one frame/component in the design or use a class prefix |

## Merge and re-import

| Symptom | Cause | Fix |
|---|---|---|
| My edit in a generated file shows as a merge conflict | You and the design changed the same (or an adjacent) line | **Keep code** is the default; choose *Take Figma* per conflict only if you want the generated line. Better: move your logic to your own class |
| "The baseline of the last import is missing and the file differs from what was written: the whole file is one conflict." / HC07 warning | `Handoff/Slate/Baselines` was not committed or was deleted | Restore the baselines from source control or from `Saved/SlateHandoffBackups`; otherwise decide the single conflict, after which the baseline is written again |
| "Generated file was deleted; recreated." | You deleted a generated file | Nothing to do; it is recreated. Hand edits in it are lost with it |
| "This destination was last imported from file X; the package comes from Y. Nodes are re-matched by name and hierarchy." | A different design file was imported into the same source folder | Check **Re-link**; use the same design file for a destination |
| A removed layer is still in the generated code; row says "Locked" ("Deleted in Figma but kept: referenced by <files>") | Your code uses the `fig_...` member | The member stays as a collapsed detached placeholder. Remove the references, then re-import; the placeholder goes away |
| TypeChange conflict on a member | The design changed its widget class and your code uses the member | *Keep code* (default) leaves the class's files unchanged ("The class keeps its current generated files: a design change to a member your code uses was kept"); *Take Figma* changes the member's class, then fix the code that no longer compiles |
| A pasted/duplicated frame in Figma creates new members and "removes" the old ones | Node ids changed and the fallback match was below 0.85 | Open **Re-link**, press *Link* on the suggestion (or `-Relink=new=old` in the commandlet) |
| A layer renamed in Figma was not matched | Id changed and layer name differs, so auto-link refused (layer name similarity must be >= 0.9) | Link it manually in Re-link |
| Re-import of the unchanged package shows changes | A generated file, the manifest or a baseline was edited or reverted outside Handoff (for example the manifest was not committed) | Run `-Plan -Explain` to see which files differ; restore manifest and baselines from source control |
| "Manifest version N was written by a newer plugin." | Manifest written by a newer plugin version | Update the plugin |
| "Could not write the file (read-only or checked out by someone else?)." | Source control / file attributes | Check out or make writable, re-run (use **Restore last import** for a partial import) |
| Ctrl+Z / the IDE did not undo the import | Imports write files; there is no editor transaction | Use **Restore last import**, or revert in source control. See [ReimportRules.md](ReimportRules.md#backup-and-undo-caveat) |
| "No backup found for ..." (restore) | Backups disabled, or none for that source folder | Pass the right `-Destination`; backups live in `Saved/SlateHandoffBackups/` |

## Fonts, images and packaged builds

| Symptom | Cause | Fix |
|---|---|---|
| "Font 'Inter Bold' is not mapped. Text using it uses the engine default font at runtime until you map a TTF/OTF file..." | No mapping row, no file in a search folder, nothing copied earlier | Fonts tab > *Map file...*, put `Inter-Bold.ttf` in `<Project>/Fonts`, or *Download Google Font (OFL)*, then re-import. Unmapped fonts fall back to the engine default font at runtime. See [FontsAndStyles.md](FontsAndStyles.md) |
| Text spacing / line height slightly off | Font not mapped, so its metrics are unknown | Map the font |
| Images are white, missing or the default font shows in a **packaged** build, but the editor is fine | The resources folder is not staged, so the images and fonts are not in the build | Add `Slate/Handoff` (path relative to `Content`) to *Project Settings > Packaging > Additional Non-Asset Directories to Package*; for a plugin module add `RuntimeDependencies`. **Check Setup** HC05 says which. See [FontsAndStyles.md](FontsAndStyles.md#packaging) |
| HC05 "is outside Content, so it cannot be staged" | Project resources folder outside `Content` | Use a folder below `Content`, e.g. `Content/Slate/Handoff`, and re-import (the old files stay) |
| Images look blurry | Textures copied at 1x, or the exporter produced 1x only | Set *Textures* to 2x in settings or the import window, export with `1x and 2x`, re-import |
| Layer imported as an image instead of widgets | Exporter rasterised it (vector, boolean, blur, mask, gradient) | The info message names the reason; restructure the layer or accept it |
| Preview shows an image but the compiled widget is empty | Resource file missing from the resources folder at runtime | The brush is a style entry loaded from `Resource("Images/...")`; check the file exists below the resources folder (or is staged) |
| Grid auto layout looks different | Imported as a Wrap Box | Switch the frame to a regular auto layout, or accept |
| "Input actions are not generated for Slate; bind input in your code ('IA_X' on 'Layer')." | `[action=...]` tags are reported, not generated | Bind input in your own class |

## Package and fetch

| Symptom | Cause | Fix |
|---|---|---|
| "The package was written by a newer exporter (schema N, importer supports M). Unknown fields are ignored; update the Handoff plugin." | Exporter newer than the plugin | Update the Handoff plugin; the import still proceeds. An old schema below the supported range is rejected ("Schema version N is no longer supported ...": re-export) |
| "The package could not be opened." / "Choose a .handoff.zip, a package folder or a layout.json first." | Wrong path or damaged zip | Re-export; point at the zip, folder or `layout.json` |
| "Missing schemaVersion; this is not a Handoff package." | Not a Handoff `layout.json` | Export through the Handoff exporter |
| Figma fetch: "No personal access token set (Editor Preferences > Plugins > Handoff for Slate (User))." | No token | Add it in the user settings |
| Figma fetch: "Not a Figma file URL or key." | Bad argument | Pass a design URL or the file key |
| Figma fetch reports a rate limit | View/Collab seats: 20 GET-file calls per month; Dev/Full: 10-20 per minute. Each fetch counts | Wait, fetch fewer times, or use the exporter plugin, which does not use the REST API |
| REST import is missing variables | Variables are Enterprise-only and not available through this path | Layer-name tags such as `[button]` work; or use the exporter plugin |
| "No renderer available: run the editor with rendering, or start the commandlet with -AllowCommandletRendering." | `-Render` / `render_preview` under `-nullrhi` or without the flag | Run with a GPU and `-AllowCommandletRendering` |
| `-ExecCmds` console command does nothing with a path | Path contains spaces, or commands were not comma separated | Use paths without spaces, separate commands with commas, or use the commandlet |
