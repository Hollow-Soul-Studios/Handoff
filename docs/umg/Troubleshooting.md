# Troubleshooting

Start with the **Messages** tab (or the commandlet report / `Result.Report`). `-Explain` on the commandlet lists
every managed value where the asset differs from the design.

| Symptom | Cause | Fix |
|---|---|---|
| "Font 'Inter Bold' is not mapped. Text using it keeps the engine default font..." | No mapping row, no file in a search folder, nothing imported earlier | Fonts tab > *Map file...*, put `Inter-Bold.ttf` in `<Project>/Fonts`, or *Download Google Font (OFL)*. Re-import: the text picks the font up. See [FontsAndStyles.md](FontsAndStyles.md) |
| Text spacing / line height slightly off | Font not mapped, so its metrics are unknown | Map the font |
| "Parent class is X but the import wants Y (CommonUI mode changed?). The existing parent class is kept." | CommonUI mode was switched after the first import; parent classes are never changed on re-import | Keep the mode used originally, or restore the last import and re-import everything with the new mode, or reparent the Blueprint yourself |
| "CommonUI mode needs the CommonUI plugin. Enable it (Edit > Plugins > Common UI) and restart, or turn CommonUI mode off." Import does not open | Common UI plugin disabled | Enable it and restart, or untick CommonUI |
| CommonUI option has no effect / no styles generated | `HandoffCommonUI` bridge only loads when Common UI is enabled | Enable Common UI and restart |
| "Input action 'IA_X' needs CommonUI mode; ignored." | `[action=...]` without CommonUI mode | Import in CommonUI mode |
| "No UInputAction asset named 'IA_X' under /Game; input action ignored." (5.8) | The asset is not in *Input action folder* (default `/Game`), or the name differs | Create the `UInputAction`, set the folder in settings (CommonUI group), re-import. Also enable Enhanced Input support in Common Input settings |
| "Input action row not found in the legacy CommonUI input action table" (5.7) | Row missing, or table not set | Set *Legacy input action table* and add a row named like the tag |
| "Enhanced Input is not available; input action ignored." | Enhanced Input plugin disabled | Enable it |
| A removed layer is still in the Widget Blueprint; row says "Locked" ("Deleted in Figma but kept: referenced by ...") | The widget is used by a graph, property binding, animation or `BindWidget` member | Remove the reference, then re-import; or delete the widget by hand |
| "Deleted in Figma but kept: contains widgets added in Unreal" | You added widgets inside it | Move your widgets out, then re-import |
| My edit shows as a conflict | Property changed in Unreal (and perhaps in Figma) | Keep Unreal is the default and is remembered; choose *Take Figma* only if you want the design value |
| A widget I deleted came back as conflict "Deleted in Unreal" | Design still has the layer | Default keeps it deleted; no action needed |
| A pasted/duplicated frame in Figma creates new widgets and "removes" the old ones | Node ids changed and the fallback match was below 0.85 | Open **Re-link**, press *Link* on the suggestion (or `-Relink=new=old` in the commandlet) |
| A layer renamed in Figma was not matched | Id changed and layer name differs, so auto-link refused (layer name similarity must be >= 0.9) | Link it manually in Re-link |
| Container conflict after toggling auto layout in Figma | Widget kind changed (for example Horizontal Box -> Canvas Panel) | Keep the old widget, or accept the swap; lost slot edits are listed |
| Texture looks blurry | Textures imported at 1x, or exporter produced 1x only | Set *Texture scale* to 2x in settings or the import window, export with `1x and 2x`, re-import |
| Layer imported as an image instead of widgets | Exporter rasterised it (vector, boolean, blur, mask, gradient) | Info message names the reason; restructure the layer or accept it |
| "Package was written by a newer exporter (schema N, importer supports M)..." | Exporter newer than the plugin | Update the Handoff plugin; unknown fields are ignored and import still proceeds. Packages two schema versions behind are rejected |
| "The package could not be opened." / "Choose a .handoff.zip, a package folder or a layout.json first." | Wrong path or damaged zip | Re-export; point at the zip, folder or `layout.json` |
| Grid auto layout looks different | Imported as a Wrap Box | Switch the frame to a regular auto layout, or accept |
| "An asset that is not a Widget Blueprint exists at this path; skipped." | Another asset sits at the frozen asset path | Rename or move the other asset |
| "Some packages could not be saved (read-only or checked out by someone else?)." | Source control / file attributes | Check out or make writable, re-run |
| Figma fetch: "No personal access token set (Editor Preferences > Plugins > Handoff for UMG (User))." | No token | Add it in the user settings |
| Figma fetch: "Not a Figma file URL or key." | Bad argument | Pass a design URL or the file key |
| Figma fetch reports a rate limit | View/Collab seats: 20 GET-file calls per month; Dev/Full: 10-20 per minute. Each fetch counts | Wait, fetch fewer times, or use the exporter plugin, which does not use the REST API |
| REST import is missing variables | Variables are Enterprise-only and not available through this path | Layer-name tags such as `[button]` work; or use the exporter plugin |
| "No renderer available: run the editor with rendering, or start the commandlet with -AllowCommandletRendering." | `-Render` under `-nullrhi` or without the flag | Run with a GPU and `-AllowCommandletRendering` |
| Ctrl+Z did not undo the import | Assets, textures, fonts, new Blueprint classes and compiles are outside the transaction | Use **Restore last import**, or revert in source control. See [ReimportRules.md](ReimportRules.md#backup-and-undo-caveat) |
| "No backup found for ..." (restore) | Backups disabled, or none for that destination | Pass the right `-Destination`; backups live in `Saved/HandoffBackups/` |
| "Manifest version N was written by a newer plugin." | Manifest written by a newer plugin version | Update the plugin |
| Re-import of the unchanged package shows changes | The asset or manifest was edited outside Handoff (for example the manifest was not committed) | Run `-Plan -Explain` to see which values differ; restore the manifest from source control |
