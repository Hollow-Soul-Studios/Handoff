# Roadmap

Planned work after 1.0 for Handoff for UMG and Handoff for Slate. This is a statement of intent, not a promise of
dates; the order may change with user feedback. Vote with a thumbs-up on the matching
[feature request](../../issues?q=label%3Aenhancement).

## Shared (both products)

* **Photoshop and Penpot exporters** that write the same tool-neutral package format.
* **MCP / agent-callable wrapper** around the existing Python API (the Python API and the commandlets
  `HandoffImport` / `SlateHandoffImport` already ship in 1.0).

## Handoff for UMG

### Next (v1.x)

* **Spatial navigation-rule suggestions** for CommonUI screens.
* **MVVM ViewModel stubs** generated for imported screens.
* **String Table localization** for imported text.

## Handoff for Slate

### Next (v1.x)

* **Input action generation** for `[action=...]` tags (today they are reported, not generated).
* **Per-state button text colours** in generated button classes.
* **Localization** of generated text through `LOCTEXT` and String Tables.
* **Navigation hints** for focusable generated widgets.

## Always

* A build within 14 days of each Unreal Engine minor release; the latest 2 to 3 engine versions supported.
* Updates for changes in the Figma developer platform.
* Patches every 2 to 4 weeks for the first three months, then monthly.

## Non-goals

Both products:

* No runtime module; the plugins stay editor-only.
* No gameplay logic generation.
* No exact fidelity for vector boolean operations or blur (they are rasterised).
* No UEFN or Verse target.
* No prototype-flow graphs.

Handoff for UMG:

* No full UMG-to-Figma round trip beyond the import status write-back.

Handoff for Slate:

* The generated code depends only on `Slate` and `SlateCore`.
* No UMG output: Widget Blueprints are the job of Handoff for UMG.
* No full code-to-Figma round trip beyond the import status write-back.
