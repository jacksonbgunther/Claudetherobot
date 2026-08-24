---
name: telegram-daily-digest
description: Send a brief Telegram digest — morning (what's queued/planned, what's waiting) or evening (what actually posted, real numbers if any, one line learned). Called from daily-loop at the right points, not a standalone entry point. Not an approval channel — no buttons, nothing to tap.
---

# Telegram daily digest

Per issue #7 (2026-08-24): a standing, low-noise "text from a cofounder"
channel on top of the existing per-approval Telegram pings
(`telegram-notify`). This does not replace per-post approvals — it adds a
short situational-awareness message twice a day at most.

## Hard constraint: brief, no fluff

Bullet points, not paragraphs. A few lines total. If there's nothing new,
say that in one line — don't pad a quiet day to sound busier than it was.
The full record stays in `memory/journal/` and `memory/decisions/`; this is
a glance, never the full picture.

## Precondition check

Check whether `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` are both set. If
either is missing, log that Telegram isn't configured and return —
non-blocking, same as `telegram-notify`.

## When this gets called (owned by `daily-loop`, not this file)

- **Morning**: once per calendar day, on the first full-cycle `daily-loop`
  run of that day (not a quiet step-0b check-in). Gate on
  `memory/state.md`'s `last_morning_digest` date field — skip if it
  already equals today.
- **Evening**: once per calendar day, on any `daily-loop` run (full cycle
  or check-in) where the current UTC hour is 20 or later. Gate on
  `last_evening_digest` — skip if it already equals today. This can't know
  in advance which run is truly "the last" of the day, so it fires on the
  first run at or after 20:00 UTC instead — close enough for a glance, and
  simpler than guessing.

Both fields live in `memory/state.md` next to `last_loop_run`; set them
immediately after a successful send so a later run in the same day doesn't
double-send.

## Building the message

**Morning** — read `content/queue/` (what's drafted and ready),
`memory/approvals/pending/` (what's waiting on a human tap/click), and
`memory/state.md`'s active priorities for anything explicitly planned
today:

```
<b>Morning — Day {N}</b>
• Queued: {content/queue/ filenames, or "nothing queued"}
• Waiting on you: {pending approval titles, or "nothing"}
• Today: {one line on the single highest-leverage planned action, or "steady-state, nothing new planned"}
```

**Evening** — read `content/posted/` for anything moved there *today*
(compare file mtimes or today's journal entry, not the whole directory),
`memory/metrics/*.csv` for any real numbers logged today, and today's
`memory/journal/YYYY-MM-DD.md` for the honest one-line takeaway:

```
<b>Evening — Day {N}</b>
• Posted: {what actually went live today, or "nothing posted today"}
• Numbers: {real figures logged today, or "not enough data yet"}
• Learned: {one honest line from today's journal, or "nothing new — steady state"}
```

Never fabricate a number or a "posted" claim — if `content/posted/` didn't
change today, say so plainly rather than reaching for something to report.

## Sending

`POST https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage`:

```json
{
  "chat_id": "$TELEGRAM_CHAT_ID",
  "text": "<built message>",
  "parse_mode": "HTML"
}
```

No `reply_markup` — this is informational only, nothing to tap. On
success, note the send (time, mode) in the run-log line. On failure, log
it and continue — never block the rest of the loop over this.

## Guardrails

- Never send both morning and evening digests in the same call — they're
  two distinct triggers, gated independently.
- Never pad a quiet digest with speculative or planned-but-not-real
  content to make the day look busier.
- Never attach APPROVE/REJECT buttons here — that's `telegram-notify`'s
  job, for a different purpose. Mixing them would make it unclear what
  tapping vs. not tapping this message means.
