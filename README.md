# Handoff

Documentation, support and issue tracker for **Handoff**, a family of Unreal Engine editor plugins that import a design
package and let you re-import after the design changes without losing your own work. A free companion exporter turns a
Figma design into the package; the package format is tool-neutral, so other exporters can target it. Pick the product
that matches the output you want: Widget Blueprints or Slate C++ code.

This repository contains **documentation and issues only**. The plugins are distributed through Fab and their source is
not hosted here.

## Products

| Product | Output | Highlights | Start here |
|---|---|---|---|
| **Handoff for UMG** | Widget Blueprints, optionally CommonUI (text styles, button styles, `CommonButtonBase` buttons, `CommonActivatableWidget` screens) | Re-import is a 3-way merge per property; graphs, bindings and hand edits are kept | [Plugin README](docs/umg/plugin-readme.md) |
| **Handoff for Slate** | Slate C++ classes (`SCompoundWidget` per screen and component, plus a style class) | Generation gap (`Generated/S<Name>Base.h/.cpp` is regenerated, `S<Name>.h/.cpp` is yours); line-level 3-way merge of generated files | [Plugin README](docs/slate/plugin-readme.md) |

Both products share:

* **The Figma exporter**, a free plugin shipped prebuilt in each plugin download (`Extras/FigmaExporter`):
  [exporter guide](docs/figma-exporter.md).
* **The package format** (`.handoff.zip` with `layout.json`): [layout schema](docs/umg/LayoutSchema.md) (the Slate copy,
  [LayoutSchema](docs/slate/LayoutSchema.md), describes the same contract).
* The workflow: **Plan** shows every change in a diff window before **Apply** writes anything, a backup is taken before
  every apply and **Restore last import** puts the files back, plus built-in **Check Setup** health checks and a
  **support bundle** (nothing is uploaded).
* Editor only: no runtime module, and nothing from the plugins ships in your packaged game.
* Automation: Python, a commandlet for CI and console commands (names differ per product; see each product's
  Automation page).

Requirements: Unreal Engine 5.7 or 5.8 (built and tested on 5.8). Windows is the tested platform; Mac and Linux are
allow-listed in the plugin modules but untested. Handoff for Slate additionally needs a C++ project with a build
toolchain.

*Figma is a trademark of Figma, Inc. Handoff for UMG and Handoff for Slate are independent products and are not
affiliated with or endorsed by Figma. Unreal Engine is a trademark of Epic Games, Inc.*

## Documentation

Start at the [documentation index](docs/index.md).

| | Handoff for UMG | Handoff for Slate |
|---|---|---|
| Overview | [Plugin README](docs/umg/plugin-readme.md) | [Plugin README](docs/slate/plugin-readme.md) |
| Quick start | [Quick start](docs/umg/QuickStart.md) | [Quick start](docs/slate/QuickStart.md) |
| Output mapping | [Feature mapping](docs/umg/FeatureMapping.md) | [Code generation](docs/slate/CodeGeneration.md) |
| Re-import | [Re-import rules](docs/umg/ReimportRules.md) | [Re-import rules](docs/slate/ReimportRules.md) |
| Fonts | [Fonts and styles](docs/umg/FontsAndStyles.md) | [Fonts and styles](docs/slate/FontsAndStyles.md) |
| Automation | [Automation](docs/umg/Automation.md) | [Automation](docs/slate/Automation.md) |
| Troubleshooting | [Troubleshooting](docs/umg/Troubleshooting.md) | [Troubleshooting](docs/slate/Troubleshooting.md) |
| Health checks | [Health checks](docs/umg/support/health-checks.md) | [Health checks](docs/slate/support/health-checks.md) |
| Support bundle | [Support bundle](docs/umg/support/support-bundle.md) | [Support bundle](docs/slate/support/support-bundle.md) |
| FAQ | [FAQ](docs/umg/support/faq.md) | [FAQ](docs/slate/support/faq.md) |

Shared: [Figma exporter](docs/figma-exporter.md).

## Getting help

1. Check Troubleshooting and the FAQ of your product (links above).
2. In Unreal run **Tools > Handoff for UMG > Check Setup...** or **Tools > Handoff for Slate > Check Setup...** and fix
   anything marked Warning or Fail (what each check means: [UMG](docs/umg/support/health-checks.md),
   [Slate](docs/slate/support/health-checks.md)).
3. Still stuck? Run **Create Support Bundle...** and [open an issue](../../issues/new/choose), choose the product in the
   form and attach the zip. Full steps in [SUPPORT.md](SUPPORT.md).

For license and purchase questions use the publisher contact on the Fab product page; never post purchase details in an
issue.

## Support policy

* **Replies within 2 business days** on issues.
* **Engine support:** a build within 14 days of each Unreal Engine minor release. The latest 2 to 3 engine versions are
  supported.
* **UE 5.x is supported through the end of 2028.**
* Patches every 2 to 4 weeks for the first three months after release, then monthly. See the
  [changelog](CHANGELOG.md) and [roadmap](ROADMAP.md).

## Contributing

Issues only, plus documentation fixes by pull request. See [CONTRIBUTING.md](CONTRIBUTING.md) and the
[Code of Conduct](CODE_OF_CONDUCT.md). Documentation license: [LICENSE-docs.md](LICENSE-docs.md).
