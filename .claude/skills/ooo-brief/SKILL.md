---
name: ooo-brief
description: Scenario 2. Builds a read-only brief of the most important org-wide changes (CS & Engineering, product, infra/AWS, org announcements) from configured Slack channels plus Outlook, and DMs it to the owner. Modes daily (last working day) and catchup (whole OOO window). Called by ooo-agent.
---

# Org-change brief

## Window
- `daily`: since the previous daily brief, or since the previous weekday's run if none (Mondays cover the weekend).
- `catchup`: the whole OOO window, `start 00:00` → `now`.

## Collect (read-only)
For each channel in `briefing.tiers` (skip anything matching `exclude_channel_patterns`):
- `slack_read_channel` with `oldest`/`latest` set to the window (concise format). Open threads
  (`slack_read_thread`) only for messages with ≥3 replies or matching `change_keywords`.
- Skip bot noise, joins/leaves, emoji-only, and pure chit-chat.

Extra sources:
- Outlook mail (`outlook_email_search`) received in the window from leadership/all-staff lists or with
  `change_keywords` in the subject: subject + 1-line gist only.
- `catchup` mode only: Outlook calendar meetings in the window the owner missed (subjects only), e.g. All Hands.

## Rank
- Tier comes from the channel. Promote any item matching `change_keywords` from the "Org FYI" tier to the tier it's about.
- Within a tier, order by: announcements/decisions > changes that affect implementation work (connectors,
  petl-config, product-security-modules, deployment, AWS) > discussion with many replies > the rest.
- Keep at most `max_items_per_section` items per tier; fold the remainder into "+N more in #channel".
- Respect the customer data rule in CLAUDE.md: conclusions only, no pasted data.

## Format and send
One self-DM message (split into consecutive messages only if over ~4,500 chars):
```
🤖 *OOO agent* — <Daily brief <date> | Catch-up brief <start>–<end>>
*TL;DR:* <2–3 sentences: the things you must know>

🔴 *Must know: CS & Engineering*
• <one line: what changed / what was decided — who> <permalink>
🟠 *Product changes*
• …
🟡 *Infra / AWS*
• …
⚪ *Org FYI*
• …
📅 *Meetings you missed* (catchup only)
• …
`ooo-agent:<daily-brief:<date> | catchup-brief:<start>..<end>>`
```
Omit empty sections; if everything is empty send "Quiet day — nothing notable." with the marker.
