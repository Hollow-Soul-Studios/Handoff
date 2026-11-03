# FAQ

## Do I need Figma to use the plugin?
No. You need a design package. The free companion exporter creates one from Figma design. Other tools can target the
documented [package format](../LayoutSchema.md). Two sample packages are included with the plugin to try the importer.

## Does the plugin ship in my game?
No. It is editor-only; there is no runtime module. The Widget Blueprints it generates are ordinary UMG assets and cook
for any platform your project supports.

## Which engine versions are supported?
Unreal Engine 5.7 and 5.8 (built and tested on 5.8). Each engine version has its own download on Fab. A build follows
each UE minor release within 14 days, and UE 5.x is supported through the end of 2028. UE 5.6 and older are not
supported.

## Which operating systems?
Windows is tested. Mac and Linux are allow-listed in the plugin modules but have not been tested.

## Is the Figma plugin free? Do I need a paid Figma seat?
The exporter is free and needs no token or paid seat: it reads your file through the plugin API. It ships prebuilt in
the plugin download (`Extras/FigmaExporter`); load it once in the Figma desktop app, see [Figma exporter](../figma-exporter.md).
The optional REST fetch inside Unreal needs a personal access token from a Dev or Full seat and counts against your
seat's rate limit.

## Will re-import overwrite my changes?
Not by default. A property you changed in Unreal is kept unless you choose *Take Figma*, and the diff window shows
every change before you apply. See [Re-import rules](../ReimportRules.md).

## Can I undo an import?
Use **Tools > Handoff for UMG > Restore Last Import**, or revert in source control. Ctrl+Z is not reliable for
imports; see the [backup and undo caveat](../ReimportRules.md#backup-and-undo-caveat).

## Why is my font missing?
The plugin bundles no fonts. Map a TTF/OTF file, put it in `<Project>/Fonts`, or use the user-triggered Google Fonts
(OFL) download. See [Fonts and styles](../FontsAndStyles.md).

## Does CommonUI mode need extra setup?
Enable the Common UI plugin (and Enhanced Input support for `[action=...]` tags on 5.8). The
[health check](health-checks.md) HC03 to HC05 tells you what is missing.

## Does it use AI?
No AI runs inside the plugin and nothing is sent to an AI service. Imports are deterministic: the same package gives
the same assets. The plugin code itself was written with AI assistance; this is declared on the Fab listing.

## Does it send data anywhere?
No telemetry. Network access happens only when you trigger it: the Google Fonts download and the optional Figma REST
fetch. The support bundle is created locally and is only shared if you attach it to an issue.

## Are there free alternatives?
Yes: free and open-source Figma-to-UMG tools exist, and AI assistants can generate widget layouts. They are good
choices for one-off conversions. Handoff for UMG is aimed at the repeated round trip: re-importing a changed design
safely, with a reviewable diff, while keeping your Blueprint logic.

## How do I get support?
See [SUPPORT.md](../../SUPPORT.md): run Check Setup, create a support bundle, and open an issue. Replies within 2
business days.
