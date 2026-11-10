# Re-import rules

Importing a package over an existing destination never starts from scratch. The importer compares three
things per property: what the last import wrote, what the asset has now, and what the new package says.
This page describes exactly what is changed, what is kept and what is never touched.

## The manifest

`<Project>/Handoff/Manifests/<destination with / replaced by _>.json` records, per asset and per widget:

* the node id (or `nodeId#role` for wrapper widgets), layer name and path (stored only here);
* the frozen widget name `fig_<8 hex FNV-1a of node id>[_role]` and a deterministic widget GUID registered in
  the Blueprint's `WidgetVariableNameToGuidMap` (so renaming a widget in Unreal is followed);
* parent key, sibling index, bounds;
* per-property hashes: `figma` (last design value), `ue` (value the importer wrote), `acceptedLocal`
  (a local edit you chose to keep).

The file is sorted, deterministic JSON (LF, no BOM). Commit it with your content.
Asset paths are frozen: renaming a frame in Figma never renames the Widget Blueprint. If a frame's id changes,
it is re-linked to its asset by name.

## Identity and re-link

1. **Node id** first.
2. Fallback score for nodes whose id vanished: 0.35 hierarchy path + 0.35 layer name + 0.15 type + 0.15 geometry.
3. A fallback match is linked automatically only when the score is >= 0.85, it is unambiguous (best candidate
   beats the next by a 2% margin) and the layer name still matches (similarity >= 0.9).
4. Scores >= 0.45 below that are **suggestions** only and are never applied.

The **Re-link** tab lists *Linked* (auto or user-made, each with **Unlink**), *Suggestions* (with **Link**) and
the unmatched old and new nodes. Every automatic link is listed for review. Typical case: a frame copy-pasted in
Figma gets new ids; linking it migrates the manifest ids, no widget is recreated.

## 3-way merge, per property

| Unreal vs last import | Figma vs last import | Result |
|---|---|---|
| unchanged | unchanged | nothing |
| unchanged | changed | **update** (applied) |
| changed | changed | **conflict**: default *Keep Unreal*, detail "Changed in Figma and in Unreal" |
| changed | unchanged | **conflict**, default *Keep Unreal*, detail "Edited in Unreal" (shown once; after you keep it, it is remembered and not reported again) |
| equal to the design value | any | converged: nothing to change |
| property not recorded before (newly managed) | | Figma value is applied |

A kept local edit is remembered (`acceptedLocal`) and not reported again until Figma changes that property.
The conflict policy option is `KeepUnreal` (default) or `TakeFigma` (`-TakeFigma` on the commandlet); in the window
use per-row toggles, **Keep all Unreal edits** or **Take all Figma**.

## Moves and order

* Moved to another parent in Figma only: **moved** (slot reset).
* Moved in Figma and in Unreal: conflict, kept.
* Layer order changes in Figma reorder managed siblings only; widgets you added by hand keep their place.

## Type and container changes

If a layer's widget kind changes (for example auto layout toggled off, so a Horizontal Box becomes a Canvas
Panel), the result is a conflict that keeps the old widget. Accepting swaps the widget under the same name and
moves the children into the new container:

* Slot fields you edited in Unreal that both slot types share (Padding, alignment, Size, ...) keep your value.
  The plan lists them ("Unreal slot edits carried to the new container"), and the manifest remembers them as
  kept edits, so later imports do not overwrite or re-flag them unless Figma changes that field.
* Slot fields you did not edit take the design value for the new container.
* Slot edits the new slot type cannot hold (for example canvas anchors when a Canvas Panel becomes a box) are
  listed as lost before you apply.

## Deletions

| Event | Result |
|---|---|
| Widget deleted in Unreal | conflict "Deleted in Unreal"; kept deleted by default |
| Node deleted in Figma | marked for **removal** |
| ... but the widget is referenced by a graph, property binding, animation or a `BindWidget` member | **locked**: never removed ("Deleted in Figma but kept: referenced by ...") |
| ... but it contains widgets you added by hand | **locked** ("Deleted in Figma but kept: contains widgets added in Unreal") |

Locked rows cannot be applied and stay in the manifest. **Take all Figma** never removes referenced widgets.

## Never touched

* Event graphs, functions, macros, variables, property bindings, animations
  (exceptions: the Handoff-owned `HandoffApplyText` / `HandoffApplyTextStyle` functions and their variables in
  component widgets, plus a single call node the importer adds to Event PreConstruct / OnCurrentTextStyleChanged);
* widgets you added by hand;
* the parent class: if CommonUI mode changed since the first import, a warning says the existing parent class
  is kept.

## Safety net

1. Before **Apply**, the affected `.uasset` files and the manifest are copied to
   `Saved/HandoffBackups/<yyyyMMdd-HHmmss>/` with a `backup.json`. With source control active and
   *Check out with source control* on, the files are also checked out.
2. **Restore last import** (menu, window footer, Python `restore_last_import`,
   or commandlet `-Restore`) copies the files back byte for byte, reloads the packages, deletes assets the import
   created and restores (or deletes) the manifest. It aborts without changes if a backed up file is missing.
3. Re-importing the same package plans zero changes (idempotent); `-Explain` lists every managed value that
   differs between design and asset.

### Backup and undo caveat

Ctrl+Z is **not** a reliable undo for a Handoff import. Tree and property edits are wrapped in an editor
transaction, but new assets, texture and font imports, new Blueprint classes and Blueprint compiles are not
reliably undone by it. Your real undo is **Restore last import** and your source control. Keep *Backup before
apply* on, commit before importing, and do not disable backups (`-NoBackup`) outside throwaway projects.
Only the latest backup of a destination is restored by the Restore command; older ones remain in
`Saved/HandoffBackups/` for manual recovery.
