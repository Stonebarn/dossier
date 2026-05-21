---
name: dossier-discover
description: First-run discovery for the Dossier plugin. Inspects connected MCP sources (meetings + chat), asks the user about scope, then generates rich profiles of everyone they interact with most — organized into Leadership / Coworkers / Partners / External folders with INDEX files per category. Use this when the user runs `/dossier-discover`, or says things like "build my dossier", "set up dossier", "profile everyone I work with", "create profiles from my meetings and chats".
---

# Dossier — Discover

## Goal

From the user's connected meeting + chat sources, build a structured `profiles/` directory with one rich markdown profile per person they actually interact with. The output is a living reference — not a one-shot summary.

## Required output

```
<profiles_root>/
├── TEMPLATE.md                  # the per-person profile template
├── dossier.config.json          # user-editable config (lookback, top_n, folders, sources)
├── Leadership/
│   ├── INDEX.md
│   └── <person>.md (one per person)
├── Coworkers/
│   ├── INDEX.md
│   └── <person>.md
├── Partners/
│   ├── INDEX.md
│   └── <person>.md
└── External/
    ├── INDEX.md
    └── <person>.md
```

## Workflow

### 1. Detect available sources

Before asking the user anything, inspect what's connected. Look for MCP tools matching these patterns (use whatever is available):

**Meeting sources:**
- Granola: `mcp__*__list_meetings`, `mcp__*__get_meetings`, `mcp__*__get_meeting_transcript`, `mcp__*__query_granola_meetings`
- Gong: `mcp__*__list_calls`, `mcp__*__get_call_transcript`
- Fireflies: `mcp__*__list_meetings`, `mcp__*__get_transcript`
- Otter: `mcp__*__list_speeches`, `mcp__*__get_transcript`
- Zoom: `mcp__*__list_recordings`, `mcp__*__get_recording_resource`, `mcp__*__search_meetings`
- Google Calendar: `mcp__*__list_events` (for attendee discovery even if no transcript)

**Chat sources:**
- Slack: `mcp__*__slack_search_public_and_private`, `mcp__*__slack_search_users`, `mcp__*__slack_read_user_profile`
- Microsoft Teams: any `mcp__*__teams_*`
- Discord: any `mcp__*__discord_*`

**Identity / enrichment (optional, used to fill role/title):**
- LinkedIn-style: Apollo, Wiza, Clearbit
- HubSpot/Salesforce: `mcp__*__search_crm_objects`

If you can't find at least one meeting source AND one chat source, stop and tell the user:

> Dossier needs at least one meeting source (e.g. Granola, Gong, Fireflies, Otter) and one chat source (e.g. Slack, Teams) to build useful profiles. Please connect one of each, then run `/dossier-discover` again.

If only one of the two is available, ask whether to proceed with reduced fidelity or wait until both are connected.

### 2. Ask the user (use the AskUserQuestion tool)

Bundle into one batch of 4 questions max:

1. **Sources to use** — show the detected sources, let them confirm or deselect any.
2. **Scope** — Internal + external (default), internal only, external only, or 3+-interaction filter.
3. **Profile depth** — Rich (recommended), lightweight, or sales-focused.
4. **Top N cap** — 25 (recommended), 50, or unlimited.
5. **Folder location** — Default is `<workspace>/profiles/`. Offer to override.

Also confirm:
- The **user's own name, email, and role** (needed for "you" context and to filter "me" out of attendee lists).
- Their **direct manager** if they want extra-depth on that profile.

### 3. Pull the data

Build a unified person→interactions index across all selected sources:

- **Meetings:** for each meeting in the lookback window where the user is an attendee, capture: meeting_id, title, date, attendees (name + email), summary/notes (avoid pulling full transcripts unless you need to — too expensive at discovery time; reserve transcripts for the boss profile if requested).
- **Chat:** for each conversation in the window where the user is sender OR recipient OR is mentioned, capture: counterpart (name + email + workspace), channel, message_count, and a representative snippet.

Skip:
- Bot accounts and automated notifications
- Calendar resource emails (e.g. `7-wtc-...@resource.calendar.google.com`)
- Distribution lists (`all@`, `salesteam@`, `everyone@`)
- People with only one-time, large-group attendance (e.g. a 60-person webinar)

### 4. Rank and select top-N

Compute a simple interaction score per person:

```
score = (meetings_count × 3) + (chat_messages × 1) + (one_on_one_meetings × 5)
```

Sort descending. Take the top N (default 25). Tie-break by recency.

### 5. Categorize

Route each person to a folder:

- **Leadership/** — same email domain as the user AND any of:
  - Title contains "founder", "CEO", "CTO", "COO", "CRO", "CFO", "VP", "Director", "Head of"
  - Appears in a channel matching `*-leadership` or `*-execs`
  - User explicitly nominates them as manager / skip-level
- **Coworkers/** — same email domain as user, not in Leadership
- **Partners/** — different email domain, AND any of:
  - Appears in 2+ meetings titled like "<Company> ↔ <UserCompany>", "<Vendor> trial", "renewal", "kickoff", "implementation"
  - Cross-workspace DM history sustained over 7+ days
  - In a known partner-vendor channel
- **External/** — different email domain, doesn't meet Partner criteria

When ambiguous, ask the user once for a batch decision rather than per-person.

### 6. Write `TEMPLATE.md`

Copy from `templates/TEMPLATE.md` (sibling to this skill, in the plugin root). This is the per-person template every profile follows.

### 7. Generate each profile

For each top-N person:
- Fetch their meeting notes/summaries (batch up to 10 meeting IDs at a time).
- Fetch their chat messages in DMs with the user (limit 30, paginate once).
- Optionally fetch chat from shared channels where they're active.
- Fill the TEMPLATE — every section must reference real evidence. If a field has no signal, write `Unknown — not enough signal yet`.
- Save as `<firstname-lastname>.md` (lowercase, hyphenated) in the chosen subfolder.

For efficiency on large top-N runs, spin up sub-agents in batches of 5 people each, in parallel. Pass each agent: the person's name/email, the relevant meeting IDs, instructions to pull chat themselves, and the template path.

### 8. Write `INDEX.md` per subfolder

Each `INDEX.md` is a quick-reference table:

```markdown
# <Folder Name>

<One-paragraph description of what this folder contains>

| Person | One-liner | File |
| --- | --- | --- |
| **<Name>** | <Role>. <2-3 word recurring theme>. | [<file>](<file>.md) |
```

For Leadership, you may want sub-sections (direct manager, peers, execs). For Coworkers with 10+ entries, sub-sections by function (e.g. SDR team, Sales team, Product). Use judgment based on the data.

### 9. Write `dossier.config.json` at profiles root

```json
{
  "version": "0.1.0",
  "user": {
    "name": "<user name>",
    "email": "<user email>",
    "role": "<user role>"
  },
  "profiles_root": "<absolute path>",
  "lookback_days": 1,
  "top_n": 25,
  "sources": {
    "meetings": ["granola"],
    "chat": ["slack"]
  },
  "folders": ["Leadership", "Coworkers", "Partners", "External"],
  "categorization_rules": {
    "leadership_titles": ["founder", "CEO", "CTO", "COO", "CRO", "CFO", "VP", "Director", "Head of"],
    "leadership_channels": ["*-leadership", "*-execs"]
  }
}
```

### 10. Offer the next steps

After everything is written, prompt:

> Dossier setup complete. <N> profiles created across Leadership (<x>), Coworkers (<y>), Partners (<z>), External (<w>).
>
> Want me to:
> 1. Build your own voice profile so I can write in your tone? (run `/dossier-me`)
> 2. Set up the daily refresh task to keep these up to date? (run `/dossier-refresh --schedule`)

## Rules

- Every claim in a profile must trace to evidence. No filler, no padding.
- Use the person's exact words wherever possible — quote, don't paraphrase.
- "Unknown — not enough signal yet" is the right answer for fields you can't fill.
- Never write profiles for bots, distribution lists, or calendar resources.
- Don't pre-judge external contacts as low-value — a one-off coffee chat that becomes a referral lane matters.
- All work goes to the user's chosen `profiles_root`. Never write outside it.

## Done state

The user has a populated `profiles/` directory with INDEX files, a config file, and a clear next-step prompt.
