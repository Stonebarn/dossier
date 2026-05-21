<img width="1374" height="418" alt="image" src="https://github.com/user-attachments/assets/a0aab43e-c916-4999-878e-25a789c6fcaa" />

# Dossier

> A Claude plugin that builds rich profiles of the people you work with — from your real meetings and chats — and keeps them fresh automatically.

Most "contact tools" make you enter data manually. Dossier reads what already exists in your calendar transcripts and chat history, then writes a living profile for everyone you actually interact with. Personality. Communication style. Recurring topics. Open commitments. How to work well with them.

It also builds one for *you* — a voice profile so any Claude can draft messages, docs, and posts in your tone.

## What you get

Run `/dossier-discover` once and you'll have a `profiles/` folder like this:

```
profiles/
├── TEMPLATE.md
├── YOU.md                      # your own voice + style profile
├── Leadership/
│   ├── INDEX.md
│   └── <boss>.md, <ceo>.md, ...
├── Coworkers/
│   ├── INDEX.md
│   └── <teammate>.md, ...
├── Partners/                    # brand partners — vendors, integrations
│   ├── INDEX.md
│   └── ...
└── External/                    # one-off contacts, prospects, community
    ├── INDEX.md
    └── ...
```

Each profile is ~200-400 lines of evidence-based content:

- **Snapshot** — role, relationship type, interaction volume, last touch
- **Personality & Communication Style** — tone, pace, signature phrases, emoji habits
- **Working Style** — how they decide, how they like to be reached
- **Relationship With You** — recurring topics, notable moments, trust level
- **Open Threads** — what you owe them, what they owe you
- **How To Work Well With Them** — specific do's and don'ts
- **Sources** — every meeting and channel cited inline

## How it works

Dossier is **source-agnostic**. On first run it inspects your connected MCP servers and asks which ones to use as data sources. Common ones:

- **Meetings**: Granola, Gong, Fireflies, Otter, Zoom, Google Calendar
- **Chat**: Slack, Microsoft Teams, Discord
- **Email** (optional): Gmail, Outlook

You only need at least one meeting source and one chat source for dossier to produce useful output. The more it has, the richer the profiles.

## Install

```
/plugin install Stonebarn/dossier
```

Or clone manually:

```bash
git clone https://github.com/Stonebarn/dossier.git ~/.claude/plugins/dossier
```

Then restart Claude. The four commands below become available.

## Commands

| Command | What it does |
| --- | --- |
| `/dossier-discover` | First-run setup. Inspects your MCP connections, asks how far back to look (default 30 days) and how many people to profile (default 25). Generates the full folder structure with categorized profiles + INDEX files. |
| `/dossier-refresh` | Pulls the last 24h of activity, updates existing profiles with dated bullets in "Recent Topics" and "Open Threads", auto-creates new profiles for anyone new with enough signal. Bundles into a scheduled task if you want it to run daily. |
| `/dossier-me` | Builds *your* voice profile — channel-specific writing patterns (boss DM, peer DM, team broadcast, banter), signature phrases, mental models, do's and don'ts. Used by Claude any time it drafts content in your voice. |
| `/dossier-add <name>` | Manually add or upgrade a profile for someone, including people dossier filtered out of the top-N by volume. Pulls all available signal for them and places them in the right subfolder. |

## Categorization

New profiles are auto-routed based on email domain and role signal:

- **Leadership/** — your direct manager, your manager's chain, founders, execs
- **Coworkers/** — teammates with your email domain who aren't in Leadership
- **Partners/** — people at companies with an active commercial relationship (your vendors, customers, integrations) — detected from frequency + meeting cadence
- **External/** — everyone else (community contacts, one-off coffee chats, prospects, advisors)

You can override the routing in `dossier.config.json` (created on first run).

## Daily refresh

After `/dossier-discover`, dossier offers to set up a scheduled task that runs `/dossier-refresh` every morning. Each run takes ~2 minutes and produces a one-line summary:

```
Dossier refresh: 3 profiles updated, 1 new profile created (Coworkers/), 0 skipped.
```

## Privacy

Dossier writes everything to your local machine. Nothing is sent anywhere except your existing MCP connections (which you already control). The generated `profiles/` folder is gitignored by default — you'll want to keep these files private.

## Customization

After the first run, dossier creates `dossier.config.json` at the profiles root. Tweak:

- `lookback_days` — how far back to scan on `/dossier-refresh` (default 1)
- `top_n` — how many people to maintain profiles for (default 25)
- `folders` — rename Leadership/Coworkers/Partners/External to whatever fits your world
- `categorization_rules` — override the routing logic (email domain patterns, channel signals)
- `sources` — disable specific MCPs even if connected

## License

[MIT](./LICENSE) — fork it, ship a variant, charge for it, whatever you want. PRs welcome.

## Credits

Built by [Jack Foley](https://github.com/jackafoley), GTM Engineer at [Wiza](https://wiza.com).

Inspired by every "I should write down what I learned about this person" note that never got written.
