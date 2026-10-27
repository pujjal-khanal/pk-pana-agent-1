---
name: ooo-brief
description: Scenario 2. Builds a read-only brief of the most important org-wide changes (CS & Engineering, product, infra/AWS, org announcements) from configured Slack channels plus Outlook, and DMs it to the owner. Modes daily (last working day) and catchup (whole OOO window). Called by ooo-agent.
---

# Org-change brief

## Window
- `daily`: since the previous daily brief, or since the previous weekday's run if none (Mondays cover the weekend).
- `catchup`: the whole OOO window, `start 00:00` → `now`.

## Collect (read-only)
For each channel in `briefing.tiers` (skip anything matching `exclude_channel_patterns`; for channels with
`only_if`, keep only messages matching one of those keywords, case-insensitive):
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
The owner wants it **short, TL;DR style**. One self-DM message, aim for under ~1,200 characters
(catchup may be up to ~2,500):
```
🤖 *OOO agent*: <Daily brief <date> | Catch-up brief <start>–<end>>
*TL;DR*
• <the 1–3 things that matter most, any tier>

🔴 <headline ≤ max_words_per_item> <link>
🟠 <headline> <link>
🟡 <headline> <link>
⚪ <headline> <link>
📅 Missed: <meeting>, <meeting> (catchup only)
`ooo-agent:<daily-brief:<date> | catchup-brief:<start>..<end>>`
```
- One line per item, prefixed by its tier emoji; at most `max_items_per_section` per tier, highest tier first.
- Headlines only: what changed + what (if anything) the owner needs to do. No background or quotes.
- Flag breaking changes or actions for the owner with ⚠️.
- Link to the source thread/page where available, else name the channel.
- If nothing is notable, send "Quiet day, nothing notable." with the marker.
