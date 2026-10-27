---
name: ooo-agent
description: Entry point for the scheduled OOO agent run. Detects upcoming/current OOO from Outlook, then decides whether to ask for the auto-reply schedule, send the auto-reply text, send the daily brief, or ask for/send the catch-up brief. Use when the Routine fires or when asked to "run the OOO agent".
---

# OOO agent run

Read `CLAUDE.md` (hard rules + state model) and `config/agent.yml` first. All times are in `owner.timezone`.

## 1. Load state
- `now` = current time in owner timezone.
- Read the self-DM history (channel `U08MDVA8DD5`, last 30 days). Collect existing `ooo-agent:` markers.

## 2. Find the OOO window
Search Outlook calendar (`outlook_calendar_search`, query `*`, from 10 days ago to `look_ahead_days` ahead,
order `oldest`, paginate through every page). An event is OOO when `showAs` is in `ooo_detection.show_as` or its subject matches a
`subject_keywords` entry (case-insensitive). Ignore cancelled events. HiBob leave syncs in as all-day events titled "Holiday"; treat these as OOO. Merge overlapping/adjacent OOO events
(including weekends between them) into windows `start..end` (dates).

If several windows exist, act on each; if none, and no pending catch-up (step 3d), exit silently.

## 3. Decide what to do (per window, first match wins)

| Situation | Action |
|---|---|
| a. Window starts within `look_ahead_days`, no `ooo-detected` marker | Run **ooo-autoreply** step "ask" |
| b. `ooo-detected` exists, owner replied in its thread, no `autoreply-text` marker | Run **ooo-autoreply** step "write" |
| c. `now` is a weekday inside the window, current run hour == `daily_brief_run_hour`, no `daily-brief:<today>` | Run **ooo-brief** mode `daily` |
| d. Today is the return day or the weekday after it (first 2 weekdays after the window end), no `catchup-ask` | Send the catch-up question (below) |
| e. `catchup-ask` exists, no `catchup-brief`, owner replied in its thread and the requested time ≤ `now` | Run **ooo-brief** mode `catchup` |

Otherwise exit silently. Never send more than one message per situation per run.

### Catch-up question (situation d)
```
🤖 *OOO agent*
Welcome back! When do you want the catch-up brief for <start>–<end>?
Reply in this thread with a time (e.g. `now`, `1pm`, `tomorrow 9am`). I check at ~7:50, 12:50 and 15:50 CT.
`ooo-agent:catchup-ask:<start>..<end>`
```
If the owner replies `skip`, send nothing further for that window.

## 4. Finish
Print a one-line run summary to the session log (what was sent, or "nothing to do"). Never message anyone else.
