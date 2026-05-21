---
name: dossier-refresh
description: Refreshes the Dossier profile set based on recent activity (default last 24h). Updates existing profiles with dated bullets in "Recent Topics" and "Open Threads", creates new profiles for anyone with enough signal, and routes them to the right subfolder. Use when the user runs `/dossier-refresh`, says things like "update my dossier", "refresh profiles", "sync dossier", "what's new with my people", or when running as a scheduled task. Supports a `--schedule` flag to install/update a daily cron at 7am local time.
---

# Dossier — Refresh

## Goal

Keep the Dossier profile set current. Read recent activity, update existing profiles in place, and create new ones for anyone with enough signal.

## Workflow

### 1. Load config

Read `<profiles_root>/dossier.config.json`. If missing, tell the user dossier isn't set up yet — direct them to `/dossier-discover`.

Key fields needed:
- `user.email` — to filter activity belonging to the user
- `user.name`, `user.role`
- `lookback_days` — defaults to 1 if running manually, can be overridden by argument
- `sources` — which MCPs to query
- `folders` — the four bucket names
- `categorization_rules` — for routing new people

### 2. Handle the `--schedule` flag

If invoked with `--schedule`:
- Create or update a scheduled task named `dossier-refresh` with cron `0 7 * * *` (7am daily, local time).
- The task prompt should be `Run dossier-refresh` (or the equivalent skill invocation in the current shell).
- Tell the user: "Scheduled. Dossier will refresh every morning at 7am. Click 'Run now' once in the Scheduled sidebar to pre-approve permissions."
- Stop here. Do not run a refresh.

### 3. Pull the lookback window

Compute `since = now - lookback_days`. Default 1 day.

**Meetings:** for each enabled meeting source, list meetings since `since` where the user is an attendee. Pull notes/summaries (skip transcripts unless you need to disambiguate a quote).

**Chat:** for each enabled chat source, search:
- Messages from the user since `since`
- Messages to the user since `since`
- Messages where the user is @mentioned since `since`
- Limit ~30 per query, paginate once if needed

### 4. Build per-person activity bundles

For every distinct human (skip bots, resource calendars, distribution lists), collect:
- Meetings attended with the user (title + date + ID)
- Chat messages exchanged (channel + timestamp + brief excerpt)
- Notable signal: action items, commitments, decisions, quotes, emoji reactions

### 5. For each person — update or create

**Find the existing profile** by globbing `<profiles_root>/**/<firstname-lastname>.md`.

**If found:** `Read` it, then `Edit` it. Make minimal, targeted changes:

- **Snapshot** — update "Most recent interaction" to today. Bump interaction volume if clearly higher.
- **Recent Topics & Themes** — append new dated bullets (`[YYYY-MM-DD] <topic — short context>`). If the section now has more than 12 bullets, prune the oldest.
- **Open Threads / Action Items** — append new dated items. Mark resolved items as `[x]` if today's activity clearly closes them.
- **Sources** — append today's meeting titles and Slack channels.

Do NOT rewrite Personality, Working Style, or "How To Work Well With Them" sections automatically. Those are stable, hand-tuned content. Only update them if today's signal contradicts what's there (in which case append a `> Note: <date> — <new observation>` quote, don't overwrite).

**If not found:** This person is new. Route them:
- `user.email` domain match + senior title signal → Leadership
- Same domain otherwise → Coworkers
- Different domain + commercial cadence (vendor patterns, partner channel) → Partners
- Different domain otherwise → External

Read `<profiles_root>/TEMPLATE.md`, fill it in from today's signal (use "Unknown — not enough signal yet" for fields without evidence), and `Write` to the target subfolder.

Then update that subfolder's `INDEX.md` to add a new row. Read the existing INDEX format first and match it. If the INDEX has sub-sections (e.g. "Sales team" within Coworkers), pick the right sub-section based on what you know — when in doubt, add to a "Recent additions" subsection at the bottom rather than guessing.

### 6. Skip rules

Do NOT create or update profiles for:
- Bot accounts or automated notifications
- Distribution lists (`all@`, `salesteam@`, etc.)
- Calendar resource accounts
- Anyone with only "read-only FYI" signal — they showed up in someone else's @mention but didn't directly interact
- Large-group community session attendees (e.g. a 60-person webinar)

### 7. Report

End with a one-line summary:

```
Dossier refresh: <X> profiles updated, <Y> new profiles created (<folder breakdown>), <Z> skipped. New: <names>.
```

If nothing meaningful happened:

```
Dossier refresh: nothing new.
```

## Rules

- Don't rewrite existing profile sections — only append dated bullets and update Snapshot.
- Don't touch profiles for people with no activity in the window.
- Every appended bullet must include the date prefix `[YYYY-MM-DD]` so the file stays auditable.
- Never invent signal. If you didn't see it, don't write it.

## Done state

The profile set reflects activity through the end of the lookback window, with a concise summary returned to the user.
