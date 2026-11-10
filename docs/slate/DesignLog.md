# Design log (clean-room record)

Purpose: show how Handoff for Slate, and the core it shares with Handoff for UMG, were designed and implemented
independently. This log is kept with the source because the plugin ships as source.

## Policy

* The importer, the package format, the planner, the manifest, the line merge, the code generator and the exporter
  were written from scratch for this project, from the product plans (`docs/designer-to-umg.md`,
  `docs/designer-to-slate.md`) and from Unreal Engine's public API and headers.
* **No code was copied or adapted from Figma2UMG, UnityFigmaBridge, FigForge or any other project.** Where the
  behaviour of other tools was studied, only observable behaviour and published documentation were used.
* If a snippet is ever adapted from a permissively licensed project, its notice must be added to
  `ThirdPartyNotices.txt` in the same change and a dated entry added here.
* Product naming: "Figma" is never part of a product name; the docs say "works with Figma" or "for Figma design".

## Prior art and what it informed

| Project | License | Studied | Used |
|---|---|---|---|
| Figma2UMG | MIT | Behaviour: how components, variants and instances can map to Widget Blueprints; the idea of a patch mode | Ideas only. No code |
| UnityFigmaBridge | MIT | Components as prefabs; Google Fonts download; re-adding scripts and button bindings after a sync | Ideas only: informed the safe re-import goal and the user-triggered font download |
| FigForge | MIT | Plugin + bridge architecture, per-axis anchors, deduplicated rasterisation | Ideas only |
| Figma Converter for Unity (marketplace listing) | commercial | Public feature list | Inspiration for scope only; no product code examined |
| benui / Unreal Garden articles | public articles | Technique: constructing a `UWidgetTree` in code, widget variables and `WidgetVariableNameToGuidMap` | Technique credited for the UMG product; not used by Handoff for Slate, which generates C++ source |

The Slate product adds no new prior-art sources: its line-level merge is a standard three-way text merge (diff3)
written from the textbook description, and the generation-gap split (generated base class, hand-written
subclass) is a long-standing code-generation pattern.

## Entries

### 2026-10-24 (shared core, Handoff for UMG)

* **Package as the contract.** Decided the importer reads a tool-neutral package (`layout.json` + PNG assets),
  not Figma directly. Separate engine-neutral module `HandoffCore` (parser, zip reader, planner, manifest, REST
  converter) so the logic is unit-testable without the editor. Schema documented in `Docs/LayoutSchema.md`.
  (Handoff for Slate carries the same core as `SlateHandoffCore`.)
* **Rasterise by contract.** Exporters rasterise anything that is not zero or one solid fill (`asset` /
  `fillAsset`); the importer never tries to reproduce gradients, blurs, booleans or vectors.
* **Determinism.** Frozen widget names (`fig_<FNV-1a of node id>`), deterministic widget GUIDs, stable text keys,
  sorted manifest JSON, content-hashed asset ids. Goal: re-importing an unchanged package plans zero changes.
* **Identity and merge.** Node id first; fallback match score = 0.35 hierarchy path + 0.35 name + 0.15 type +
  0.15 geometry, auto-link at >= 0.85 with a 2% margin and name check, suggestions from 0.45. Per-property
  hashes {figma, ue, acceptedLocal} drive a 3-way merge: Unreal edits win by default.
* **Never-touched list.** Graphs, functions, variables, bindings and hand-added widgets are outside the managed
  set. Removal of a referenced widget is locked rather than offered.
* **Mapping choices.** Auto layout -> Horizontal/Vertical/Wrap boxes with wrapper roles (`size`, `bg`, `stack`,
  `abs`, `valign`, `gapN`) so every design feature has a plain expression; constraints -> canvas anchors; px ->
  Slate points at 0.75.
* **Tree construction (UMG).** Built Widget Blueprints by creating widgets through `UWidgetTree::ConstructWidget`
  and registering variable GUIDs, following the technique described in the benui / Unreal Garden material.

### 2026-10-25 (shared core, Handoff for UMG)

* **Safety net.** Backup of affected `.uasset` files plus manifest to `Saved/HandoffBackups/<timestamp>/`
  before Apply; restore copies bytes back and deletes created assets. Documented plainly that Ctrl+Z does not
  cover asset creation, texture/font import, Blueprint class creation or compiles.
* **CommonUI isolation (UMG).** CommonUI support lives in a separate `HandoffCommonUI` module that uses reflection
  only and is active only when the CommonUI plugin is enabled, so the core plugin has no hard dependency.
* **Fonts.** Nothing bundled. Mapping table, search folders, and a user-triggered Google Fonts download that
  checks for an OFL license first and keeps `OFL.txt`.
* **REST path.** Optional, for Dev/Full seats; converts the REST response to the same package format so the
  importer has a single code path. Not required for normal use.
* **Verification.** Automation tests `Handoff.Core.*` and the `Handoff.Importer.ReimportScenario` M4 scenario
  were written against the plan's exit criteria; sample packages are generated by a deterministic script
  (`Tools/Samples/make_samples.py`, internal, not shipped) from the scenario table in `Samples/SCENARIO.md`.
* **Docs.** User documentation written against the code (mapping rules in `HandoffWidgetMapper.cpp`, planner
  in `HandoffPlan.cpp`), not against other tools' docs.

### 2026-11-03 (Handoff for Slate)

* **Code generation instead of assets.** The Slate product writes C++ source (`SCompoundWidget` classes built
  with `SNew` / `SAssignNew`) instead of creating assets. The mapper produces engine-neutral widget tree specs
  whose property values are the C++ argument expressions; the code generator (`SlateHandoffCodeGen.cpp`) turns a
  spec into source text and the preview interpreter (`SlateHandoffPreview.cpp`) reads the same strings. One
  mapping rule set (`SlateHandoffMapper.cpp`) therefore serves generation, planning and preview. The contract is
  `Docs/CodeGeneration.md`.
* **Generation gap.** Each class is split into `Generated/S<Name>Base.h/.cpp` (owned by Handoff, regenerated on
  every import) and `S<Name>.h/.cpp` (a stub written once when missing and never touched again). Logic belongs in
  the user class, so the common case needs no merge at all. Class and member names are frozen in the manifest so
  a renamed layer never renames code.
* **Line-level merge with baselines.** Hand edits inside generated files are still possible, so a re-import
  merges them with `FSlateHandoffTextMerge::Merge3` (diff3, in the engine-neutral core): base = baseline text of
  the last generated version (`Handoff/Slate/Baselines/<manifest name>/...`), local = file on disk, remote = new
  generated text. Generated code puts one argument per line, which makes the line merge a property merge.
  Conflicts default to keeping the code; the baseline written after Apply is the new generated text, not the
  merged file.
* **Record-derived state.** The planner no longer reads asset state. It rebuilds what the generated code contains
  from the manifest record (`StateFromRecord`, with the generated property values stored in the record) and
  diffs the desired tree against it. Members named in user code are found by scanning the module's non-generated
  sources (`FindMemberReferences`); removal of such a member is locked (kept as a detached collapsed
  placeholder) and a class change is a `TypeChange` conflict whose Keep leaves the class files unchanged.
  Unreferenced class changes are `ReplaceWidget` and apply.
* **Image brushes as style entries.** Slate loads image files only for brushes of a registered style set, so an
  image is never an inline brush in a widget: the style class registers `Image.<stem>.<W>x<H>` / `Box...`
  entries (`FSlateImageBrush` / `FSlateBoxBrush`) with the resources folder as the content root and widgets
  reference them by name. Fonts load lazily from the same folder. Packaged builds must stage that folder, so a
  health check (HC05) verifies it.
* **Canvas Alignment default 0.5.** `SConstraintCanvas::FSlot` defaults `Alignment` to (0.5, 0.5). The generator
  always emits `.Alignment(...)` so anchored layout matches the design; auto-sized text uses its alignment
  fraction. `SImage` gets `DesiredSizeOverride` because colour brushes carry no size.
* **Preview interpreter with dynamic image brushes.** The preview window builds live widgets from the planned (or
  manifest-recorded) trees without compiling, evaluating the same value expressions. Image brushes are loaded
  from files as dynamic brushes there, because the style set of the generated code does not exist before the
  user builds. `render_preview` / `-Render` render the same widgets to PNG.
* **Verification.** Automated tests keep the `SlateHandoff.*` prefix; the v1 -> v2 scenario in
  `Samples/SCENARIO.md` is the regression script for the merge, locks, type change and restore.
