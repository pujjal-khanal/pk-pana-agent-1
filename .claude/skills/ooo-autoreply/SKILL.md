---
name: ooo-autoreply
description: Scenario 1. Asks the owner (via self-DM) to confirm the Outlook auto-reply schedule for a detected OOO, then DMs ready-to-paste internal and external auto-reply text. Never sets the auto-reply itself. Called by ooo-agent.
---

# Outlook auto-reply text

The agent never changes mailbox settings; the owner pastes the text into
Outlook → Settings → Mail → Automatic replies → "Send replies only during a time period".

## Step "ask"
Send to the self-DM:
```
🤖 *OOO agent*
I found an Out of Office in your calendar: *<event subject>*, <start> – <end> (CT).
Reply in this thread with:
1. Auto-reply schedule, e.g. `<start> 08:00 – <end> 17:00` (or `ok` to use the full days)
2. Backup contact(s) for CS / Engineering questions
3. Anything to mention (optional), e.g. customers or projects
`ooo-agent:ooo-detected:<start>..<end>`
```

## Step "write"
Parse the owner's thread reply. If the schedule or backup is missing or unclear, reply in the thread once asking
for the missing part. That reply is still only to the self-DM. Then stop.

Otherwise send (as a new self-DM message, not a thread reply, so it notifies):
````
🤖 *OOO agent*
Auto-reply for <start> – <end>. Paste into Outlook → Automatic replies → "Only send during a time period":
*Start:* <start datetime CT>   *End:* <end datetime CT>

*Internal (Panaseer)*
```
<internal text>
```
*External*
```
<external text>
```
`ooo-agent:autoreply-text:<start>..<end>`
````

Text guidance:
- Internal: first name tone, dates, return date, backup by name (+ Slack handle if given), "I won't be checking
  Slack or email".
- External: formal, dates, return date, backup name and email only, no internal project or customer names.
- Return date = first weekday after the end date.
- Keep each under 80 words. No emojis in the auto-reply text itself.
