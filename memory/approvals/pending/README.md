# Pending approvals

One file per open request: `memory/approvals/pending/<id>.md`, where `<id>`
is a short date-prefixed slug (e.g. `2026-08-20-media-tool-signup.md`) —
**keep it under ~55 characters**, since Telegram's inline-button
`callback_data` has a 64-byte limit and the id gets a 2-byte prefix.

Front matter, added 2026-08-21 for the Telegram approval integration
(`memory/decisions/0011-telegram-approvals.md`):

```yaml
---
action: human-manual   # human-manual | publish | test
platform:               # only for action: publish — x | threads | buffer:<channel>
content_package:        # only for action: publish — path under content/queue/
telegram_message_id:    # set by telegram-notify once sent; used to edit the message on resolution
---
```

`action` decides what happens on approval:
- `human-manual` — a task only the human can do (create an account, set up
  a tool). Approving it doesn't make me execute anything; GitHub is still
  the mechanism for these. No Telegram buttons are sent for this type —
  it isn't a yes/no decision on something I'm ready to execute.
- `publish` — I'm ready to actually post `content_package` to `platform`
  the moment this is approved. Gets a real Telegram APPROVE/REJECT
  message; approval triggers the matching `publish-*` skill.
- `test` — harmless, used only to verify the Telegram approval pipeline
  itself (`memory/decisions/0011-telegram-approvals.md`). Approving it
  does nothing except confirm the flow works end to end.

Body, per `ARCHITECTURE.md` §6 (Const. §87 format):

```
## What
## Why
## Cost
## Expected upside
## Potential downside
## What happens if we do nothing
## GitHub issue
(link)
```

Move the file to `memory/approvals/resolved/` once decided, and append the
outcome (approved/rejected, and what happened after) before moving it —
see that directory's README for the resolved format.
