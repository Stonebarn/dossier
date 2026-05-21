# Customizing Dossier

After your first `/dossier-discover`, Dossier creates `dossier.config.json` at the profiles root. Tweak any of these:

## `lookback_days`
How far back `/dossier-refresh` looks each run. Default `1` (last 24h). Set higher if you run it less often, e.g. `7` for a weekly cadence.

## `top_n`
Max profiles to maintain. Default `25`. Set `null` for unlimited (warning: noisy on large orgs).

## `folders`
Default `["Leadership", "Coworkers", "Partners", "External"]`. Rename to fit your world:

```json
"folders": ["Execs", "Team", "Vendors", "Network"]
```

If you rename existing folders mid-stream, move the files manually first — Dossier won't move them for you.

## `categorization_rules`

```json
"categorization_rules": {
  "leadership_titles": ["founder", "CEO", "VP", "Director", "Head of"],
  "leadership_channels": ["*-leadership", "*-execs"],
  "leadership_emails": ["ceo@yourcompany.com", "boss@yourcompany.com"],
  "partner_domains_known": ["vendor1.com", "vendor2.io"]
}
```

The `leadership_emails` array forces specific people into Leadership regardless of title detection. Useful for situations where your direct manager's job title in your CRM doesn't say "Director" but they obviously are.

The `partner_domains_known` array forces specific external domains into Partners instead of External.

## `sources`

```json
"sources": {
  "meetings": ["granola", "gong"],
  "chat": ["slack"]
}
```

Disable a source even if it's connected by removing it from the array. Useful if you have a noisy or low-signal source you don't want feeding profiles.

## Custom template

If you want a different per-person profile structure, edit `<profiles_root>/TEMPLATE.md` directly. Dossier reads it fresh every time it generates a profile, so changes apply going forward (existing profiles aren't retroactively rewritten — use `/dossier-add <name> --depth deep` to regenerate any specific one).

## Disabling the daily refresh

Open Claude's Scheduled tasks sidebar and disable `dossier-refresh`. Or delete it entirely.

## Excluding specific people

Add their email to `dossier.config.json`:

```json
"exclude": ["someone@company.com", "noisy.bot@company.com"]
```

They won't be auto-profiled even if their interaction volume justifies it.
