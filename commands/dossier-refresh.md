---
description: Refresh Dossier profiles based on recent activity (default last 24h). Updates existing profiles, creates new ones for people with new signal. Use --schedule to install a daily 7am cron.
---

Run the `dossier-refresh` skill.

Read `<profiles_root>/dossier.config.json` to determine the lookback window and source list. Pull recent meetings + chat activity, update existing profiles in place (appending dated bullets to Recent Topics and Open Threads), and create new profiles for anyone with enough signal — routing them to the right subfolder and updating that folder's INDEX.

If the user passed `--schedule`, install or update a scheduled task that runs `/dossier-refresh` at 7am daily, then exit without doing a sync.

End with a one-line summary of what changed.
