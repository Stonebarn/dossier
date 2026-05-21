# Getting started with Dossier

## Install

```
/plugin install jackafoley/dossier
```

Or clone manually:

```bash
git clone https://github.com/jackafoley/dossier.git ~/.claude/plugins/dossier
```

Restart Claude to register the plugin.

## Prerequisites

You need at least one meeting source and one chat source connected as MCPs:

**Meeting sources (any one):**
- Granola
- Gong
- Fireflies
- Otter
- Zoom (with recordings + transcripts)

**Chat sources (any one):**
- Slack
- Microsoft Teams
- Discord

**Optional but useful:**
- A CRM (HubSpot, Salesforce) — used to enrich titles/companies
- Google Calendar — used to discover attendees even when transcripts aren't available

## First run

```
/dossier-discover
```

You'll be asked:
1. Which sources to use (Dossier shows what it detected)
2. Scope: everyone, internal only, external only, or 3+ interactions
3. Profile depth: rich, lightweight, or sales-focused
4. Top-N cap (default 25)
5. Where to save profiles (default `<workspace>/profiles/`)

The first run takes 5-15 minutes depending on N and how much data your sources hold.

## Second step (recommended)

```
/dossier-me
```

Builds *your* voice profile so any Claude session can draft in your tone.

## Make it daily

```
/dossier-refresh --schedule
```

Installs a 7am daily refresh. Each run updates existing profiles with new dated bullets, creates new profiles for anyone with enough signal, and gives you a one-line summary.

## Promoting someone manually

If someone got filtered out of your top-N but you actually do care about them:

```
/dossier-add Tyler Meckes
```

Or override the auto-categorization:

```
/dossier-add Tyler Meckes --folder External
```

## What gets in each folder

- **Leadership/** — your manager, manager's chain, founders, execs
- **Coworkers/** — internal teammates outside Leadership
- **Partners/** — external folks at companies you have a commercial relationship with (vendors, customers, integration partners)
- **External/** — everyone else (community, prospects, one-off chats)

Dossier auto-routes based on email domain + role signals, but you can override in `dossier.config.json` or per-add with the `--folder` flag.

## Privacy

Everything stays on your machine. Dossier doesn't send your data anywhere — it only reads from MCPs you've already connected. The `profiles/` folder is gitignored by default in this repo's `.gitignore`, but make sure your own setup excludes it too.
