# Re-import rules

Importing a package over an existing source folder never starts from scratch. The importer compares three
things: what the last import generated, what is in your files now, and what the new package says. This page
describes exactly what is changed, what is kept and what is never touched.

## What is stored

| What | Where | Commit it? |
|---|---|---|
| Manifest | `<Project>/Handoff/Slate/Manifests/<source folder with / replaced by _>.json` | **yes** |
| Baselines | `<Project>/Handoff/Slate/Baselines/<manifest name>/<project-relative path of the generated file>` | **yes** |
| Your classes, generated files, resources | the source and resources folders | yes (they are your code) |
| Backups | `<Project>/Saved/SlateHandoffBackups/<yyyyMMdd-HHmmss>/` | no |

The **manifest** records, per generated class and per widget:

* the node id (or `nodeId#role` for wrapper widgets), layer name and path (stored only here);
* the frozen member name `fig_<8 hex FNV-1a of node id>[_role]` and a deterministic widget GUID;
* the generated widget class, parent key, sibling index, bounds, and `detached` for placeholders;
* per property: hashes `figma` (last design value), `ue` (value the importer generated), `acceptedLocal`
  (a kept local value) and the generated `value` itself. The state of the generated tree is rebuilt from these
  values, so no code is parsed to find out what Handoff generated;
* every generated and user file: path, kind, owning class, content hash and whether it is user-owned;
* the resources folder, the style class and the resources (images, fonts) it copied.

A **baseline** is the exact text Handoff generated for a file the last time (not the merged result written to
disk). It is the common ancestor of the 3-way merge. The manifest file is sorted, deterministic JSON (LF, no
BOM). Without the manifest, the importer cannot tell which files are Handoff's; without a baseline it cannot
separate your edits from generated lines.

Class names and file paths are frozen: renaming a frame in Figma never renames a class or its files. If a frame's
id changes, it is re-linked to its class by name.

## Identity and re-link

1. **Node id** first.
2. Fallback score for nodes whose id vanished: 0.35 hierarchy path + 0.35 layer name + 0.15 type + 0.15 geometry.
3. A fallback match is linked automatically only when the score is >= 0.85, it is unambiguous (best candidate
   beats the next by a 2% margin) and the layer name still matches (similarity >= 0.9).
4. Scores from 0.45 up to that are **suggestions** only and are never applied.

The **Re-link** tab lists *Linked* (auto or user-made, each with **Unlink**), *Suggestions* (with **Link**) and
the unmatched old and new nodes (a design-only layer is *new*, a generated-code-only layer is *old*). Every
automatic link is listed for review. Typical case: a frame copy-pasted in Figma gets new ids; linking it migrates
the manifest ids, so the members keep their names and your code keeps compiling. On the commandlet use
`-Relink=<newId>=<oldId>,...`.

## Generated files: line-level 3-way merge

The planner decides per file:

| File | Situation | Action |
|---|---|---|
| your class (`S<Name>.h/.cpp`) | missing | **Create** once (a stub deriving from the generated base) |
| your class | exists | **Yours**: never read for merging, never written |
| generated file | missing | **Create** (recreated if you deleted it) |
| generated file | identical to the new text | Unchanged |
| generated file | file equals the baseline (no hand edits) | **Update** to the new text |
| generated file | new text equals the baseline (no design change) | Unchanged: your edits stay |
| generated file | both differ | **Merge**: base = baseline, local = file on disk, remote = new generated text |
| generated file | exists, but the manifest has no record of it | **Skipped** with an error: "The file exists and was not written by Handoff" |
| generated file | record exists but the baseline is missing | unchanged-since-written: Update; otherwise the whole file is one conflict |

Generated code puts **one argument per line**, so a line-level merge is a property-level merge:

| Your file vs baseline | Design vs baseline | Result |
|---|---|---|
| unchanged | unchanged | nothing |
| unchanged | changed | the design lines are taken |
| changed | unchanged | **your lines are kept** |
| changed, identically | changed | same change on both sides: taken once |
| changed | changed (the same or an adjacent line) | **conflict**: default **Keep code**, option **Take Figma** |

Conflicts are shown as *Conflict at line N* with both sides in the diff window (**View diff** on the file row).
Choose *Keep code* (your lines stay) or *Take Figma* (the generated lines; your edit of those lines is
overwritten) per conflict. The conflict policy option is `KeepCode` (default) or `TakeFigma` (`-TakeFigma` on the
commandlet); in the window use per-conflict choices, **Keep all code edits** or **Take all Figma**. Your decision
for a conflict that did not change is remembered across **Plan** presses. Line endings are normalised to LF for
comparing.

After Apply the baseline is the **new generated text**, not the merged file. A kept hand edit therefore keeps
showing up as a local change next time, and stays merged while the design does not touch those lines.

Edit your own class instead of the generated files whenever you can: nothing in your class is ever merged or
overwritten.

## Design changes to widgets

The planner diffs the new design against the state rebuilt from the manifest record.

| Event | Result |
|---|---|
| Property value changed in the design | the generated line changes (merged with your edits as above) |
| Layer renamed | manifest only; the member keeps its frozen name ("follows rename") |
| Layer added | new member and widget code |
| Moved to another parent / order changed | follows the design |
| Layer deleted in the design, member not used by your code | marked for **removal** (applied) |
| ... member **is used by your code** | **locked**: kept as a detached placeholder ("Deleted in Figma but kept: referenced by <files>") |

A member is *used by your code* when its name (`fig_...`) appears in a `.h`, `.cpp` or `.inl` file below the
module directory that Handoff does not own (the generated files are excluded): your classes and any other file of
yours in the module count.

A **detached placeholder** is created after `ChildSlot` in the generated base class as
`SAssignNew(member, Class)` with `.Visibility(EVisibility::Collapsed)` and a comment, so your code keeps
compiling; the widget is not part of the layout. It stays in the manifest (`detached`). Locked rows cannot be
applied, and **Take all Figma** never removes members your code uses. Once you remove the references, the next
import removes the placeholder.

## Type changes

If a layer's Slate widget class changes (for example auto layout toggled off, so an `SHorizontalBox` becomes an
`SConstraintCanvas`):

* **Member not used by your code:** a **ReplaceWidget** change, applied: the member is generated as the new class
  and the children move into it. Slot values the design changes are regenerated.
* **Member used by your code (TypeChange):** a **conflict**. *Keep code* (default) leaves the class's generated
  files unchanged ("Skipped"; the design change to that class is not generated), so your code keeps compiling.
  *Take Figma* gives the member the design's widget class. Code that calls class-specific methods on the member
  may no longer compile; the conflict names your files. Slot values the new container cannot hold are listed as
  lost.

Slot values that both slot types share (padding, alignment, size) are carried over: a value that differs from the
generated one in the manifest and is shared by the new slot type is kept and remembered (`acceptedLocal`), so later
imports neither overwrite nor re-flag it unless the design changes that value again.

## Never touched

* Your classes `S<Name>.h/.cpp` and everything else you wrote in the module.
* Anything outside the generated files, the manifest, the baselines and the resources folder.
* Style entries the package no longer contains are never removed from the style class (classes generated earlier
  may use them), and generated classes that no longer appear in the package stay on disk ("orphaned": reported,
  never deleted).
* The class names and member names, once created.

## Safety net

1. Before **Apply**, every existing file the import will change (sources, resources, baselines) and the manifest
   are copied to `Saved/SlateHandoffBackups/<yyyyMMdd-HHmmss>/` with a `backup.json`. With source control active
   and *Check out with source control* on, the files are also checked out.
2. **Restore last import** (menu, window footer, Python `restore_last_import`, or commandlet `-Restore`) copies the
   files back **byte for byte**, deletes the files the import created, and restores (or deletes) the manifest.
   It aborts without changes if a backed up file is missing.
3. Re-importing the same package plans zero changes (idempotent); `-Explain` lists every planned change and file
   action.

### Backup and undo caveat

Generated code is ordinary files, so there is no editor transaction behind an import: **Ctrl+Z does not undo it**,
and an IDE's undo history is not reliable across an import either. Your real undo is **Restore last import** and
your source control. Keep *Backup before apply* on, commit before importing, and do not disable backups
(`-NoBackup`) outside throwaway projects. Restoring replaces the files with their backed-up content, so edits you
made to those files after the import are lost; close or reload the files in your IDE afterwards. Only the latest
backup of a source folder is restored by the Restore command; older ones remain in
`Saved/SlateHandoffBackups/` for manual recovery.
