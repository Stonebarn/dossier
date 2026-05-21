---
name: dossier-add
description: Manually add or upgrade a Dossier profile for a specific person — even if they didn't make the top-N cutoff during discovery. Pulls all available signal for them and places them in the right subfolder. Use when the user runs `/dossier-add <name>`, or says "add a profile for <person>", "profile <person>", "create a dossier for <person>", "upgrade <person> from external to partners", "promote <person> from the candidates list".
---

# Dossier — Add

## Goal

Add or upgrade a single person's profile on demand. This is the manual escape hatch for the auto-categorization in `/dossier-discover` and the volume-based filtering in `/dossier-refresh`.

## Workflow

### 1. Parse the input

Accept either:
- `/dossier-add <name>` — natural-language name (`/dossier-add Tyler Meckes`)
- `/dossier-add <email>` — email-style
- `/dossier-add <name> --folder Partners` — explicit folder override
- `/dossier-add <name> --depth deep` — pull 90 days instead of default 30

If no name is given, ask the user who to profile.

### 2. Resolve the person

Search across connected sources to identify them:
- Chat: `slack_search_users` (or equivalent) with name/email
- CRM (if connected): search contacts
- Meetings: search attendee lists for matching name/email across the lookback window

If multiple matches, ask the user to pick.

If no matches, ask the user to provide the email or context manually, then proceed with whatever they share.

### 3. Check for existing profile

Glob `<profiles_root>/**/<firstname-lastname>.md`. If a profile already exists:
- Ask whether to upgrade it (pull a deeper window of data and rewrite) or to skip.
- If upgrading, pull 90 days of data and use the same logic as `dossier-discover` for that one person.

### 4. Determine the folder

If `--folder` was specified, use it.

Otherwise apply the categorization rules from `dossier.config.json` (same logic as discover):
- Same domain as user + senior title → Leadership
- Same domain as user → Coworkers
- Different domain + commercial cadence → Partners
- Different domain otherwise → External

If ambiguous, ask the user.

### 5. Pull data + write profile

Same per-person logic as `/dossier-discover`:
- Fetch meeting notes (batch up to 10 IDs)
- Fetch chat messages (DMs with the user, plus shared channels if active)
- Read `<profiles_root>/TEMPLATE.md` for structure
- Fill every section with evidence — use "Unknown — not enough signal yet" for blanks
- Write to `<profiles_root>/<folder>/<firstname-lastname>.md`

### 6. Update the folder's INDEX

Read the target subfolder's `INDEX.md`. Add a new row matching the existing format. If the INDEX has sub-sections, pick the right one based on role/context (or ask if unclear).

### 7. If the person was in the "candidates" list

`External/INDEX.md` (and possibly other INDEX files) may have a "Candidates that almost made the cut" section listing people who were filtered out by the top-N cap. If the person being added is in that list, remove them from candidates before adding to the main table.

### 8. Report

Output one line:

```
Added <Name> to <Folder>/. Profile: <Folder>/<file>.md. <N> meetings + <M> chat messages pulled.
```

If upgrading:

```
Upgraded <Name> in <Folder>/. Pulled <N> meetings + <M> chat messages over 90d.
```

## Rules

- Don't auto-create a profile for someone with zero direct interaction signal. If the only thing you can find is them being on a 50-person all-hands invite, tell the user and ask if they want to proceed.
- Respect explicit `--folder` overrides without second-guessing — the user knows their world.
- Same TEMPLATE-evidence discipline as discover and refresh: every claim cited, no filler.

## Done state

The person has a profile in the right subfolder, and their folder's INDEX reflects the addition.
