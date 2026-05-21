---
name: dossier-me
description: Builds a deep self-profile for the user — voice, tone, vocabulary, channel-specific writing patterns, mental models, humor, do's and don'ts. Saved as YOU.md at the profiles root for any Claude to use when drafting content in the user's voice. Use when the user runs `/dossier-me`, or says "build my voice profile", "write a profile of me", "create a style guide for how I write", "set up a voice profile so you can write like me".
---

# Dossier — Me (Self / Voice Profile)

## Goal

Build a comprehensive style guide derived from the user's actual written and spoken output. The file becomes a reference any Claude session can pull from when drafting messages, docs, posts, or customer-facing content in the user's voice.

## Output

`<profiles_root>/YOU.md` (~400-600 lines, structured for retrieval).

## Workflow

### 1. Load config + cross-reference

Read `<profiles_root>/dossier.config.json` for user email, name, role, sources.

If existing per-person profiles are present in `<profiles_root>/Leadership/`, `<profiles_root>/Coworkers/`, etc., glob them — they're useful cross-reference data (the user's manager profile contains feedback signal about how the user operates; close friends' profiles reveal banter patterns).

### 2. Pull the user's voice in the wild

#### Chat (primary source — written voice)

For each chat source in config, search for messages **from** the user across 90 days where possible:
- DMs with their boss → "boss voice" register
- DMs with peers → "peer voice" register
- DMs with direct reports → "coaching voice" register
- DMs with close friends/banter partners → "banter voice" register
- DMs with external contacts → "external/casual professional" register
- Posts in leadership channels → "broadcast voice"
- Posts in working/ops channels → "operating voice"
- Long-form announcements or weekly-update posts → "structured voice"

Capture ~15-30 verbatim messages per register. Don't paraphrase. Voice is in the specifics.

#### Meetings (secondary source — spoken voice + thinking patterns)

For the most substantive 1:1s and small-group sessions in the lookback (e.g. weekly 1:1 with manager, peer 1:1s, coaching sessions with reports), fetch full transcripts where available. Note: some transcript services only capture the "other" speaker — flag that limitation in the profile if it applies.

Look for:
- How the user frames problems before proposing solutions
- Metaphors and mental models they reach for repeatedly
- Where they hedge vs commit
- Their sense of humor
- What they get excited about vs what they find tedious

### 3. Detect blind spots (cross-reference manager + close colleague profiles)

If a manager profile exists, mine it for "how the user operates" signal. Things like:
- Feedback the manager has given (praise + pushback)
- The manager's stated expectations
- Tensions or recurring frustrations

These reveal blind spots the user may not see in their own output. Surface them honestly in the profile — don't soften.

### 4. Write `YOU.md`

Do NOT use the per-person TEMPLATE — write a custom structure optimized for an AI to retrieve from. The structure should include (rename freely if better fits):

```
# <User Name> — Voice & Operating Profile

> Use this when drafting any content in <user>'s voice. Pull the relevant section based on channel/audience.

## 1. Identity & Role
2 paragraphs on who the user is, current focus areas, what they're known for, who they report to, who they collaborate with most.

## 2. Personality Snapshot
5-8 bullets of personality traits, each evidenced by an observed pattern or quote.

## 3. Voice Fundamentals (applies across all channels)
- Default tone register
- Signature phrases & vocabulary (10-15 actual phrases, sourced from real messages)
- Emoji habits (what they use, what they never use)
- Formatting defaults (bullets vs prose, line breaks, capitalization)
- Sentence structure tendencies
- Anti-patterns — things the user NEVER does

## 4. Channel-Specific Voice Profiles
For each register, give: opening style, body style, closing style, 3-5 verbatim quotes, "mimic this when..." cue.

### 4a. Boss DM
### 4b. Peer DM
### 4c. Direct-report / coaching DM
### 4d. Close friend / banter DM
### 4e. External partner DM
### 4f. Leadership channel broadcast
### 4g. Working / ops channel post
### 4h. Long-form doc / process write-up
### 4i. Customer-facing message (if signal exists)

## 5. Mental Models & Recurring Framings
- How the user thinks about problems
- Frameworks they've adopted (cite who from if traceable)
- Trade-off framings

## 6. Humor & Banter Patterns
- Self-deprecating examples
- Pop culture / interest references
- When humor is on vs off

## 7. How the User Gives Feedback / Coaches
- Tone when correcting a report
- Tone when pushing back on a peer or boss
- Conflict-resolution patterns

## 8. What the User Optimizes For (Values)
4-6 values inferred from actions, each with evidence.

## 9. Style Rules for Claude When Writing as <User>

### DO
6-10 specific rules grounded in observed patterns.

### DON'T
6-10 specific anti-patterns to avoid.

### Formatting defaults
- Short-form (Slack, email)
- Long-form (docs, posts)
- Emoji policy

## 10. Worked Examples (verbatim — preserve exactly)
Organize 15-25 real messages by intent:
- Giving feedback to a report
- Pushing back to a peer
- Updating boss on a problem
- Asking for help
- Sharing a decision/announcement
- Banter / friendship
- External professional
- Long-form opener

Each: 1-line context (who/when/why) + verbatim quote.

## 11. Quick Reference Card
A 10-line condensed summary. If Claude only reads this section, it should still be able to draft credibly.
```

### 5. Cross-link

After writing YOU.md, add a one-line reference at the top of the user's project memory file (e.g. `CLAUDE.md`, `AGENTS.md`, or wherever the user keeps Claude's session-start instructions) if the user agrees:

```
3. Read `profiles/YOU.md` to load <Name>'s voice and style profile. Refer back to it whenever drafting content in <Name>'s voice — messages, docs, posts.
```

Ask the user before modifying their memory file.

## Rules

- Every claim must trace to a verbatim quote or observed pattern.
- Use the user's actual words. Quote, don't paraphrase.
- Don't be sycophantic — capture blind spots and tensions where the data shows them.
- "Unknown — not enough signal yet" for sections you can't fill.
- 400-600 lines. Dense, scannable, optimized for an AI to pull sections from.

## Done state

`YOU.md` written, optionally cross-linked from the user's main memory file. The user can now ask any Claude session "draft a Slack message to <X>" and get something that actually sounds like them.
