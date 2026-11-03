# FAQ

## Do I need Figma to use the plugin?
No. You need a design package. The free companion exporter creates one from Figma design. Other tools can target the
documented [package format](../LayoutSchema.md). Two sample packages are included with the plugin to try the importer.

## Does the plugin ship in my game?
No. It is editor-only; there is no runtime module. The code it generates is ordinary Slate code in your own module,
built by your normal build for any platform your project supports. It depends only on `Slate` and `SlateCore`. Only the
resources folder (`Content/Slate/Handoff` by default) must be staged; see [Fonts and styles](../FontsAndStyles.md).

## Does it create Widget Blueprints?
No. Handoff for Slate writes C++ source files (`SCompoundWidget` classes). Widget Blueprints are the output of the
separate Handoff for UMG product.

## Do I need a C++ project?
Yes. The generated classes go into one of your game modules (or a plugin module) and are compiled by your normal build,
so you need the build toolchain. Live Coding cannot add new classes; build the project after an import. **Check Setup**
(HC03) shows the `Slate` / `SlateCore` dependency line to add to the module's `Build.cs` if it is missing.

## Which engine versions are supported?
Unreal Engine 5.7 and 5.8 (built and tested on 5.8). Each engine version has its own download on Fab. A build follows
each UE minor release within 14 days, and UE 5.x is supported through the end of 2028. UE 5.6 and older are not
supported.

## Which operating systems?
Windows is tested. Mac and Linux are allow-listed in the plugin modules but have not been tested.

## Is the Figma plugin free? Do I need a paid Figma seat?
The exporter is free and needs no token or paid seat: it reads your file through the plugin API. It ships prebuilt in
the plugin download (`Extras/FigmaExporter`); load it once in the Figma desktop app, see [Figma exporter](../../figma-exporter.md).
The optional REST fetch inside Unreal needs a personal access token from a Dev or Full seat and counts against your
seat's rate limit.

## Will re-import overwrite my changes?
Not your own classes: `S<Name>.h/.cpp` are created once and never touched again. Edits inside generated files are merged
line by line against what Handoff generated last time; conflicts default to *Keep code*, and the diff window shows
every change before you apply. See [Re-import rules](../ReimportRules.md).

## Can I undo an import?
Use **Tools > Handoff for Slate > Restore Last Import**, or revert in source control. Ctrl+Z does not undo file
writes; see the [backup and undo caveat](../ReimportRules.md#backup-and-undo-caveat).

## Why is my font missing?
The plugin bundles no fonts. Map a TTF/OTF file, put it in `<Project>/Fonts`, or use the user-triggered Google Fonts
(OFL) download. See [Fonts and styles](../FontsAndStyles.md).

## Why do my generated classes not compile or not appear?
Build the project (not Live Coding) after refreshing the IDE project files, and make sure the module depends on `Slate`
and `SlateCore`. [Health checks](health-checks.md) HC03 and HC16 report both. See also
[Troubleshooting](../Troubleshooting.md).

## Does it use AI?
No AI runs inside the plugin and nothing is sent to an AI service. Imports are deterministic: the same package gives
the same code and resource files. The plugin code itself was written with AI assistance; this is declared on the Fab listing.

## Does it send data anywhere?
No telemetry. Network access happens only when you trigger it: the Google Fonts download and the optional Figma REST
fetch. The support bundle is created locally and is only shared if you attach it to an issue.

## Are there free alternatives?
Yes: free and open-source Figma converters exist, and AI assistants can generate Slate code. They are good choices for
one-off conversions. Handoff for Slate is aimed at the repeated round trip: re-importing a changed design safely, with a
reviewable diff, while keeping your own code and hand edits.

## How do I get support?
See [SUPPORT.md](../../../SUPPORT.md): run Check Setup, create a support bundle, and open an issue. Replies within 2
business days.
