# Automation

Everything the window does can be scripted. All entry points are editor-only. In scripts and CI the source
folder is **project-relative** (for example `Source/MyGame/UI/Handoff`), inside a module.

## Python

```python
import unreal

lib = unreal.SlateHandoffLibrary

# Import with project settings (destination "" = default). Conflicts keep your code.
result = lib.import_package("D:/Exports/MainMenu.handoff.zip", destination="Source/MyGame/UI/Handoff")
print(result.new_source_files)       # True: regenerate project files and build before using the classes

# Explicit options; start from the project defaults.
opts = lib.get_default_import_options()
opts.class_prefix = "Ui"             # property names are the snake_case form of FSlateHandoffImportOptions
opts.wrap_screens_in_safe_zone = True
r = lib.import_package_with_options("D:/Exports/MainMenu.handoff.zip", opts)

# Dry run: nothing is written, result.report holds the diff.
plan = lib.plan_package("D:/Exports/MainMenu.handoff.zip", opts)
print(plan.report)

# Undo the latest import of a source folder ("" = latest of any folder).
lib.restore_last_import("Source/MyGame/UI/Handoff")

# Render every imported screen with the preview interpreter (needs a renderer; no compile needed).
pngs = lib.render_preview("Source/MyGame/UI/Handoff", "D:/Renders")   # <Class>_<W>x<H>.png per resolution

# User-triggered, OFL-only font download.
lib.download_google_font("Inter", ["Regular", "Bold"])

# REST fetch (see below).
lib.fetch_and_import_from_figma("https://www.figma.com/design/<key>/...", opts)

# Support: setup checks and a support bundle (see Docs/Support.md).
print(lib.health_check_report())     # Markdown table; lib.run_health_check() returns the list of checks
zip_path = lib.create_support_bundle(False)   # True also includes full manifests
```

The same functions are Blueprint-callable (category *Handoff*, development only). The result struct
(`FSlateHandoffImportResult`) carries success, counts (files created and updated, design changes applied,
conflicts, relink suggestions, missing fonts, merged hand edits), the written file paths (`assets`), warnings,
errors, the report text, backup id, manifest path, receipt path and `new_source_files`.

`FSlateHandoffImportOptions` fields: `destination`, `resource_directory`, `style_class`, `class_prefix`,
`wrap_screens_in_safe_zone`, `components_as_widgets`, `texture_scale` (`X1` / `X2`), `conflict_policy`
(`KEEP_CODE` / `TAKE_FIGMA`), `create_backup`, `frame_filter` (only import frames with these names or ids; empty
= all) and `receipt_path`.

## Commandlet

```
UnrealEditor-Cmd.exe <Project>.uproject -run=SlateHandoffImport -Package=D:/Exports/MainMenu.handoff.zip ^
    -Destination=Source/MyGame/UI/Handoff -Plan -FailOnConflicts -Report=D:/out/plan.txt
```

| Argument | Meaning |
|---|---|
| `-Package=<path>` | `.handoff.zip`, package folder or `layout.json` (required unless `-Fetch`, `-Restore`, `-Health` or `-SupportBundle`) |
| `-Fetch=<Figma file url or key>` | Fetch through the REST API with the access token of the user settings instead of `-Package` (see below) |
| `-Destination=<folder>` | Project-relative source folder inside a module (default: project setting) |
| `-Resources=<folder>` | Project-relative resources folder (default: project setting, or the folder recorded for the destination) |
| `-StyleClass=<FName>` | Generated style class (F prefix) |
| `-ClassPrefix=<prefix>` | Prefix inserted after `S` in new class names |
| `-Frames=<name or id>[,...]` | Only import these frames |
| `-SafeZone` | Wrap every screen in an `SSafeZone` |
| `-InlineComponents` | Expand components in place instead of generating component classes |
| `-Textures=1x\|2x` | Copy the 1x or 2x PNGs |
| `-TakeFigma` | Conflict policy Take Figma (default keeps your code) |
| `-Plan` | Dry run: print the plan, write nothing |
| `-Explain` | Print every planned change and every file action, including merge conflicts |
| `-NoBackup` | Skip the pre-apply backup |
| `-Relink=new=old,...` | Pre-link node ids, e.g. `-Relink=9:1=1:10,9:2=1:11` (`new>old` also accepted) |
| `-Report=<file>` | Write the report (UTF-8) to a file |
| `-Render=<dir>` | After the import, render every screen of the package at each preview resolution into `<dir>`, `<Class>_<W>x<H>.png`. Needs `-AllowCommandletRendering` and a GPU (not `-nullrhi`) |
| `-FailOnConflicts` | Exit code 2 when conflicts remain |
| `-Restore [-Destination=...]` | Restore the latest backup instead of importing |
| `-Health [-Report=<file.md>]` | Run the setup checks HC01..HC16 instead of importing; exit code 1 if any check is Fail |
| `-SupportBundle[=manifests] [-IncludeManifests] [-Out=<dir>]` | Write a support bundle zip (default `Saved/SlateHandoffSupport`) |

Exit codes: **0** success, **1** errors (package unreadable, import failed, no backup to restore, restore
failed, invalid argument), **2** conflicts found with `-FailOnConflicts`. The last log line is a summary:
`Handoff plan|import: success=.. created=.. updated=.. changes=.. conflicts=.. relinkSuggestions=.. missingFonts=.. mergedEdits=.. newSourceFiles=.. backup=.. receipt=..`.

`conflicts` counts design-side conflicts (type changes of members your code uses) plus merge conflicts kept on
the code side. With `-Plan`, relink suggestions are only counted; they are never applied unless you pass
`-Relink`.

### CI recipe

A design-change pipeline that imports, builds and publishes the preview:

```
:: 1. Gate: do hand edits and design changes overlap?
UnrealEditor-Cmd.exe Game.uproject -run=SlateHandoffImport -Package=Design.handoff.zip ^
    -Destination=Source/MyGame/UI/Handoff -Plan -FailOnConflicts -Report=out/plan.txt
:: exit 2 -> upload out/plan.txt and stop for a human; 1 -> the import itself is broken

:: 2. Import (writes the classes, the baselines and the manifest) and render the screens
UnrealEditor-Cmd.exe Game.uproject -run=SlateHandoffImport -Package=Design.handoff.zip ^
    -Destination=Source/MyGame/UI/Handoff -Render=out/renders -AllowCommandletRendering -Report=out/import.txt

:: 3. Regenerate project files and BUILD the project: the import only writes source files
GenerateProjectFiles ...   (or -NoUBTMakefiles on the build)
Build.bat GameEditor Win64 Development -Project=Game.uproject
```

Commit the changed sources, `Handoff/Slate/Manifests` and `Handoff/Slate/Baselines` together. When
`newSourceFiles=1` is in the summary, refresh the project files before building. Compile errors in the generated
code or in your own classes appear in step 3, not in the import: the import does not compile anything.

### Tests

```
UnrealEditor-Cmd.exe <Project>.uproject -ExecCmds="Automation RunTests SlateHandoff, Quit" -unattended -nullrhi
```

(`-ExecCmds` separates commands with commas; the list is `Automation RunTests <filter>` then `Quit`.) Suites:
`SlateHandoff.Core.*` (parser, zip and Zip64, schema versions, matcher and relink, planner merge and structure,
resolve and state, manifest round trip, text merge, REST converter), `SlateHandoff.Editor.SupportBundle` (no token
or project path in a bundle, manifests only on request, issue body in the bug report form's order) and the
re-import scenario tests of the Editor module (the v1 -> v2 scenario in `Samples/SCENARIO.md`).

## Console commands

Path arguments of console commands must not contain spaces; use the commandlet for such paths. With `-ExecCmds`
separate commands with commas.

| Command | Effect |
|---|---|
| `SlateHandoff.Import [path]` | Open the import window; with a path, plan it immediately |
| `SlateHandoff.Preview [source folder]` | Open the preview window for an imported source folder (default: last destination or project default); built from the manifest and the resource files |
| `SlateHandoff.SaveImportWindowShot <file.png> [tab]` | Save a PNG of the import window; tab 0 Changes, 1 Re-link, 2 Fonts, 3 Messages |
| `SlateHandoff.SaveWindowShot <file.png> <title text>` | Save a PNG of any window whose title contains the text |
| `SlateHandoff.Health` | Run the setup checks, print the Markdown table and open the **Handoff Health Check** window |
| `SlateHandoff.SupportBundle [manifests]` | Write a support bundle zip to `Saved/SlateHandoffSupport` |

Menu entries (**Tools > Handoff for Slate**): Import Design Package..., Restore Last Import, Preview Imported
Screens..., Check Setup..., Create Support Bundle..., Handoff Settings....

## Fetch through the REST API (optional)

For Figma Dev or Full seats. `fetch_and_import_from_figma(url_or_key, options)` or `-Fetch=` on the commandlet:

1. Set your personal access token in **Editor Preferences > Plugins > Handoff for Slate (User)**. It is stored per
   user, never in the project config.
2. The call reads `GET /v1/files`, `GET /v1/images` (1x and 2x renders) and image fills, converts the result to
   the package format in memory, and imports it like a package.
3. The log reports how many API calls were made. Rate limits: View and Collab seats get 20 GET-file calls per
   month; Dev and Full seats 10-20 per minute. Every fetch counts against your seat.
4. Variables are Enterprise-only and not available; layer-name tags (`[button]`, ...) work.
5. Rotated container children are approximate; Display P3 colours are not converted.
6. The receipt is written to `Saved/SlateHandoff/<key>.receipt.json` (or the `ReceiptPath` option).

The Figma exporter plugin does not need a token or REST access and is the recommended path.

## Receipts

After a real import a receipt is written to `<package>.receipt.json` (or the `ReceiptPath` option) with per-node
status `imported | updated | conflict | kept | removed`; `asset` is the project-relative path of your class
header. Format: `Docs/LayoutSchema.md` section 9. Load it with the exporter's **Load import receipt...**.
