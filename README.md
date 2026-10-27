# pk-pana-agent-1: OOO housekeeping agent

A scheduled Claude Code agent that covers Pujjal's housekeeping while they're out of office.
It is **read-only**: its only output is DMs to the owner's own Slack.

## Scenarios

| # | Scenario | Skill | Output |
|---|---|---|---|
| 1 | OOO in Outlook calendar → ask for the auto-reply schedule → write the auto-reply text | `ooo-autoreply` | Self-DM with paste-ready text |
| 2 | Brief on important org-wide changes: daily while OOO, plus a catch-up brief on return at a time you choose | `ooo-brief` | Self-DM brief |

## Flow

```
 Routine: weekdays 07:50 / 12:50 / 15:50 CT, fresh session each run
        │
        ▼
 ooo-agent ── reads self-DM markers (state) + Outlook calendar (OOO events)
        │
        ├─ OOO coming up, not asked yet ─────► DM: "confirm schedule + backup?"   (ooo-autoreply ask)
        ├─ owner replied in thread ──────────► DM: internal/external auto-reply   (ooo-autoreply write)
        ├─ OOO today, 07:50 run ─────────────► DM: daily brief                    (ooo-brief daily)
        ├─ first day back ───────────────────► DM: "when do you want the catch-up?"
        └─ owner replied with a time, now≥it ► DM: catch-up brief                 (ooo-brief catchup)
```

Sources and tiers are in [`config/agent.yml`](config/agent.yml). Guardrails are in [`CLAUDE.md`](CLAUDE.md).

## Replying to the agent
Reply **in the thread** of the agent's DM. Examples: `ok`, `backup: Raj (@raj)`, `1pm`, `now`, `skip`.

## Routine prompt
Create a Routine (new session per run, connectors: Slack, Microsoft 365) with cron
`CRON_TZ=America/Chicago 50 7,12,15 * * 1-5` and this prompt:

> Clone https://github.com/pujjal-khanal/pk-pana-agent-1 (default branch) if it is not already checked out,
> cd into it, read CLAUDE.md, and run the `ooo-agent` skill. Follow its hard rules strictly: read-only,
> and the only allowed write is a Slack DM to U08MDVA8DD5.

## Limitations
- The Microsoft 365 connector cannot set automatic replies, so the owner pastes the text (by design: nothing on their behalf).
- The catch-up brief is sent at the first scheduled run after the requested time (up to ~5 h later overnight).
- Messages you send to yourself don't trigger Slack push notifications. Check the DM, or star it.
