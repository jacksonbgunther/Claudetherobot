---
name: request-approval
description: Raise something to the human that needs judgment, money, publishing, or a physical/irreversible action, following ClaudeTheRobot's approval policy. Use before spending above threshold, before any publishing action, before any commitment, or whenever a real judgment call is needed.
---

# Request approval

Implements Constitution §87 (autonomy with control) via the mechanism in
`ARCHITECTURE.md` §6.

## When this is required

Per `ARCHITECTURE.md` §7: any single expense over the threshold in
`memory/state.md`, any spend that would breach the reserve floor, any
publishing/posting action (none are autonomous yet — all publishing
requires this), any outreach beyond individual/low-volume, and any
legal/financial commitment, with no exceptions for the last category.

## Steps

1. Write `memory/approvals/pending/<YYYY-MM-DD>-<slug>.md` with the
   `action` front matter (see `memory/approvals/pending/README.md` for
   the `human-manual` / `publish` / `test` distinction) and:
   ```
   ## What
   ## Why
   ## Cost
   ## Expected upside
   ## Potential downside
   ## What happens if we do nothing
   ```
   Fill each section honestly and specifically — this is the same
   structure Constitution §87 requires before asking for approval.
2. Open a GitHub issue on this repo mirroring that content, labeled
   `needs-approval` (use the GitHub MCP tools). Link the issue URL back
   into the pending file.
3. **If `action` is `publish` or `test`** (a real yes/no on something
   ready to execute), also use `telegram-notify` to send an APPROVE/REJECT
   message — this is the primary approval channel going forward, per
   `memory/decisions/0011-telegram-approvals.md`. If `TELEGRAM_BOT_TOKEN`/
   `TELEGRAM_CHAT_ID` aren't configured, log that and fall through to the
   channels below rather than blocking. `action: human-manual` requests
   don't get Telegram buttons — they're not a decision to execute
   something, so GitHub + PushNotification stays their channel.
4. Also send a `PushNotification` — one line, under 200 characters,
   leading with the decision needed (e.g. "Need approval: $15 for [tool]
   to generate first video draft — issue #4"). If you're already in live
   conversation with the human in this session, asking directly is fine
   instead of/in addition to the notification.
5. **Do not proceed with the gated action.** Treat it as blocked until the
   human responds.
6. Resolution normally arrives via the `telegram-approval-poll` skill
   (for `publish`/`test` actions) rather than being checked manually here
   — but on any wake, still check the issue/pending file directly too,
   since `human-manual` approvals and GitHub-only resolutions still land
   that way. If resolved, move the file to `memory/approvals/resolved/`,
   append the outcome and — once the approved action is actually taken —
   what happened next. If it's been open a long time, surface it
   prominently in that wake's journal entry rather than silently waiting
   forever.

## Guardrails

- Never treat silence as approval.
- Never take the gated action first and ask afterward.
- Never soften or omit the downside/cost to make approval more likely —
  the whole point is an honest judgment call.
