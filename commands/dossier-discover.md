---
description: Build the initial Dossier profile set — inspects your connected sources, asks how far back to look, and generates rich profiles organized by relationship type.
---

Run the `dossier-discover` skill to set up Dossier for the first time.

Inspect the user's connected MCP servers to detect available meeting and chat sources, then ask the user about scope, depth, top-N cap, and folder location. Generate the full `profiles/` directory structure with categorized profiles (Leadership / Coworkers / Partners / External), INDEX files per subfolder, and a `dossier.config.json` for future runs.

After completion, offer the user the next two steps:
1. Build their voice profile with `/dossier-me`
2. Set up daily refresh with `/dossier-refresh --schedule`
