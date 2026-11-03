# Automation

Everything the window does can be scripted. All entry points are editor-only.

## Python

```python
import unreal

lib = unreal.HandoffLibrary

# Import with project settings (destination "" = default). Conflicts keep the Unreal side.
result = lib.import_package("D:/Exports/MainMenu.handoff.zip", destination="/Game/UI/Handoff")

# Explicit options; start from the project defaults.
opts = lib.get_default_import_options()
opts.use_common_ui = True            # property names are the snake_case form of FHandoffImportOptions
r = lib.import_package_with_options("D:/Exports/MainMenu.handoff.zip", opts)

# Dry run: nothing is written, result.report holds the diff.
plan = lib.plan_package("D:/Exports/MainMenu.handoff.zip", opts)
print(plan.report)

# Undo the latest import of a destination ("" = latest of any destination).
lib.restore_last_import("/Game/UI/Handoff")

# User-triggered, OFL-only font download.
lib.download_google_font("Inter", ["Regular", "Bold"])

# REST fetch (see below).
lib.fetch_and_import_from_figma("https://www.figma.com/design/<key>/...", opts)

# Support: setup checks and a support bundle (see Docs/Support.md).
print(lib.health_check_report())     # Markdown table; lib.run_health_check() returns the list of checks
zip_path = lib.create_support_bundle(False)   # True also includes full manifests
```

The same functions are Blueprint-callable (category *Handoff*, development only). The result struct
(`FHandoffImportResult`) carries success, counts (created, updated, changes applied, conflicts, relink
suggestions, missing fonts), the report text, errors, backup id and receipt path.

`FHandoffImportOptions` fields: `destination`, `use_common_ui`, `wrap_screens_in_safe_zone`, `components_as_widgets`,
`texture_scale`, `conflict_policy` (`KEEP_UNREAL` / `TAKE_FIGMA`), `create_backup`, `frame_filter` (only import frames with
these names or ids; empty = all) and `receipt_path`.

## Commandlet

```
UnrealEditor-Cmd.exe <Project>.uproject -run=HandoffImport -Package=D:/Exports/MainMenu.handoff.zip ^
    -Destination=/Game/UI/Handoff -Plan -FailOnConflicts -Report=D:/out/plan.txt
```

| Argument | Meaning |
|---|---|
| `-Package=<path>` | `.handoff.zip`, package folder or `layout.json` (required unless `-Restore`) |
| `-Destination=/Game/...` | Destination folder (default: project setting) |
| `-CommonUI` | Enable CommonUI mode (also on if the project setting is on) |
| `-SafeZone` | Wrap every screen in a Safe Zone |
| `-InlineComponents` | Expand components in place instead of child widgets |
| `-TakeFigma` | Conflict policy Take Figma (default Keep Unreal) |
| `-Plan` | Dry run: print the plan, write nothing |
| `-Explain` | Print every managed value that differs between the design and the existing asset |
| `-NoBackup` | Skip the pre-apply backup |
| `-Relink=new=old,...` | Pre-link node ids, e.g. `-Relink=9:1=1:10,9:2=1:11` (`new>old` also accepted) |
| `-Report=<file>` | Write the report (UTF-8) to a file |
| `-Render=<dir>` | After a real import, render every screen at each preview resolution to `<dir>/<WBP>_<W>x<H>.png`. Needs `-AllowCommandletRendering` and a GPU (not `-nullrhi`) |
| `-FailOnConflicts` | Exit code 2 when conflicts were found |
| `-Restore [-Destination=...]` | Restore the latest backup instead of importing |
| `-Health [-Report=<file.md>]` | Run the setup checks HC01..HC16 instead of importing; exit code 1 if any check is Fail |
| `-SupportBundle [-IncludeManifests] [-Out=<dir>]` | Write a support bundle zip (default `Saved/HandoffSupport`) |

Exit codes: **0** success, **1** errors (package unreadable, import failed, no backup to restore, restore
failed), **2** conflicts found with `-FailOnConflicts`. The last log line is a summary:
`Handoff plan|import: success=.. created=.. updated=.. changes=.. conflicts=.. relinkSuggestions=.. missingFonts=.. backup=.. receipt=..`.

Typical CI gate for a design change: run with `-Plan -FailOnConflicts -Report=...` and upload the report; a
non-zero exit tells the reviewer that hand edits and design changes overlap.
With `-Plan`, relink suggestions are only counted; they are never applied unless you pass `-Relink`.

### Tests

```
UnrealEditor-Cmd.exe <Project>.uproject -ExecCmds="Automation RunTests Handoff; Quit" -unattended -nullrhi
```

Suites: `Handoff.Core.*` (parser, zip, schema versions, matcher, planner, manifest, REST converter) and
`Handoff.Importer.ReimportScenario` (the v1 -> v2 scenario in `Samples/SCENARIO.md`: 3 property conflicts, 1
container conflict, a pasted frame resolved through Re-link, `Ammo` kept while referenced, BindWidget / graph /
rename / hand-added widget intact, Restore byte-identical), `Handoff.Importer.ContainerSwapKeepsSlotEdits`
(an accepted Horizontal -> Vertical Box swap keeps a Padding edited in Unreal and does not re-flag it later) and
`Handoff.Importer.SupportBundle` (no token or project path in a bundle, manifests only on request, issue body
in the bug report form's order).

## Console commands

| Command | Effect |
|---|---|
| `Handoff.Import [path]` | Open the import window; with a path, plan it immediately |
| `Handoff.Preview <WBP path> [<WBP path> ...]` | Open the preview window for those Widget Blueprints |
| `Handoff.SaveImportWindowShot <file.png> [tab]` | Save a PNG of the import window; tab 0 Changes, 1 Re-link, 2 Fonts, 3 Messages |
| `Handoff.SaveWindowShot <file.png> <title text>` | Save a PNG of any window whose title contains the text |
| `Handoff.Health` | Run the setup checks, print the Markdown table and open the **Handoff Health Check** window |
| `Handoff.SupportBundle [manifests]` | Write a support bundle zip to `Saved/HandoffSupport` |

## Fetch through the REST API (optional)

For Figma Dev or Full seats. `fetch_and_import_from_figma(url_or_key, options)`:

1. Set your personal access token in **Editor Preferences > Plugins > Handoff for UMG (User)**. It is stored per
   user, never in the project config.
2. The call reads `GET /v1/files`, `GET /v1/images` (1x and 2x renders) and image fills, converts the result to
   the package format in memory, and imports it like a package.
3. The log reports how many API calls were made. Rate limits: View and Collab seats get 20 GET-file calls per
   month; Dev and Full seats 10-20 per minute. Every fetch counts against your seat.
4. Variables are Enterprise-only and not available; layer-name tags (`[button]`, ...) work.
5. Rotated container children are approximate; Display P3 colours are not converted.
6. The receipt is written to `Saved/Handoff/<key>.receipt.json` (or the `ReceiptPath` option).

The Figma exporter plugin does not need a token or REST access and is the recommended path.

## Receipts

After a real import a receipt is written to `<package>.receipt.json` (or the `ReceiptPath` option) with per-node
status `imported | updated | conflict | kept | removed`. Format: `Docs/LayoutSchema.md` section 9. Load it with the
exporter's **Load import receipt...**.
