---
description: Manually add or upgrade a profile for a specific person. Pulls all available signal and places them in the right subfolder. Bypasses the top-N volume filter.
---

Run the `dossier-add` skill with the argument(s) the user provided.

Parse the person's name (or email) from the input, plus any flags (`--folder <name>` to override categorization, `--depth deep` for a 90-day pull).

Resolve them across connected sources, check for an existing profile (upgrade vs create), determine the right subfolder via config rules or the explicit override, fetch their data, fill the TEMPLATE, write the file, and update the folder's INDEX.

If they were in a "candidates that almost made the cut" list inside an INDEX, remove them from candidates as part of the upgrade.

Report one line confirming what was added/upgraded and where.
