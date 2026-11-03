# Contributing

This repository is the documentation and issue tracker for Handoff for UMG and Handoff for Slate. It contains no plugin
source code, so there are no code pull requests.

## What is welcome

* **Issues:** bug reports, import mismatches, re-import problems, setup problems and feature requests. Use the
  [issue forms](../../issues/new/choose) and choose the product (Handoff for UMG, Handoff for Slate or Figma exporter).
* **Documentation fixes by pull request:** typos, unclear steps, missing troubleshooting entries, broken links. Use the
  pull request template.

The files under `docs/umg/` and `docs/slate/` that mirror the plugin documentation are synced from the plugins.
Corrections to those are still welcome as pull requests; they are applied to the plugin docs at the next release.

## Writing a good report

1. Run **Tools > Handoff for UMG > Check Setup...** or **Tools > Handoff for Slate > Check Setup...**, then
   **Create Support Bundle...**, and attach the zip ([how](SUPPORT.md)). This answers most of the questions we would
   otherwise ask.
2. Give one problem per issue, and say which product it is about.
3. State what you did, what happened and what you expected. Numbered steps that start from a fresh import of a
   package are best.
4. For layout mismatches add a screenshot or the `-Render` output next to the Figma frame, plus the `-Explain` output
   (see Automation: [UMG](docs/umg/Automation.md), [Slate](docs/slate/Automation.md)).
5. For re-import problems add the plan report from the **Changes** tab, or the output of `-Plan -Report=<file>`.
   For Handoff for Slate code that does not compile add the first compiler error and the generated file it points at.
6. If you can share the `.handoff.zip` (or a cut-down frame that still reproduces the problem), say so. Never share
   designs you are not allowed to share; the bundle works without them.
7. Do not paste your Figma token, license keys or purchase details.

## Documentation pull requests

* Keep one topic per pull request.
* Match the existing tone: short, concrete, with exact menu names, setting names and commands.
* Do not add claims about features that the plugins do not have; link the relevant doc section instead.
* By submitting a pull request you agree that the change may be used in the documentation under [LICENSE-docs.md](LICENSE-docs.md).

Please follow the [Code of Conduct](CODE_OF_CONDUCT.md).
