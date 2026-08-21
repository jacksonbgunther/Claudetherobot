---
name: telegram-approval-poll
description: Hourly job — poll Telegram for APPROVE/REJECT taps, match each to its pending approval, execute or reject accordingly, and confirm back via Telegram. This is the entire job of the dedicated Telegram-polling Routine (separate from the daily orchestrator) — see memory/decisions/0011-telegram-approvals.md.
---

# Telegram approval poll

The "receive" half of the Telegram approval system. Runs unattended,
hourly, with no human watching — every step here has to be safe to run
completely on its own, including on a run where nothing happened.

## Why polling, not a webhook

Verified 2026-08-21 (`memory/decisions/0011-telegram-approvals.md`):
Claude Code Routines cannot receive an arbitrary inbound webhook —
firing one externally requires a specific `Authorization: Bearer
<routine_token>` header plus `anthropic-version`/`anthropic-beta`
headers and a body shaped exactly `{"text": "..."}`, none of which
Telegram's webhook mechanism can produce. Polling `getUpdates` on a
schedule was the option that needed zero new infrastructure. The
tradeoff, accepted deliberately: up to ~1 hour of latency between a tap
and it being processed, not instant.

## Step 0 — preconditions

1. `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` set? If not, record a
   `blocked` run-log line explaining why and stop — don't improvise.
2. Confirm repo read/write works the same way `daily-loop` step 0a
   does (branch check, push-target confirmation) — this Routine has its
   own fresh session per fire, same branch-fragmentation risk applies.

## Step 0.5 — send any backlog

Scan `memory/approvals/pending/` for `action: publish` or `action: test`
files that have no `telegram_message_id` yet (created before
`TELEGRAM_BOT_TOKEN` existed, or `telegram-notify` failed at the time).
Call `telegram-notify` for each. This is what lets a harmless test
approval created before setup finish sending itself the first time this
Routine actually has a working token — no need to time anything by hand.

## Step 1 — compute the offset

Read `memory/telegram-approvals-log.csv`. If it has rows, offset =
`max(update_id) + 1`. If empty (first run ever), omit `offset` entirely
(fetches from Telegram's current backlog, if any).

## Step 2 — fetch updates

`GET https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getUpdates?offset=<offset>&timeout=0`

If the call fails (network, bad token), log it as `blocked`, don't
retry more than once or twice, stop cleanly.

If it succeeds but returns zero updates, that's a normal, quiet outcome
— log `ok`, "nothing new," and stop. Don't treat an empty result as an
error.

## Step 3 — process each update, in order

For each item in the response:

1. **Skip anything that isn't a `callback_query`.** Other update types
   (plain messages, etc.) aren't handled by this version — log that one
   was seen and ignored, don't error on it.
2. **Idempotency check, layer 1**: is this `update_id` already a row in
   `memory/telegram-approvals-log.csv`? If yes, skip it entirely — this
   update was already handled, possibly by a previous run that fetched
   it but failed before advancing past it.
3. **Authorization check**: does `callback_query.from.id` match
   `TELEGRAM_CHAT_ID`? If not, log it as an ignored/unauthorized tap,
   record the update_id as processed anyway (so it's never retried),
   and move on — don't act on input from anyone but the one authorized
   chat.
4. **Parse** `callback_data`: `A:<id>` = approve, `R:<id>` = reject,
   `<id>` = the approval's filename stem.
5. **Idempotency check, layer 2**: does
   `memory/approvals/pending/<id>.md` still exist? If not — already
   resolved (by a previous poll, or manually via GitHub) — this is a
   safe no-op. Still: answer the callback query, still edit the
   Telegram message if a `telegram_message_id` is known, still record
   the update_id as processed. Don't execute anything twice.
6. **If found, read its `action` front matter:**
   - **`R:` (reject), any action** — append `## Outcome\nRejected via
     Telegram at <UTC timestamp>` (plus any reason, if the human sends
     one as a follow-up message — out of scope for this version, note
     as a future enhancement, not built now), move the file to
     `memory/approvals/resolved/`. Nothing executes.
   - **`A:` (approve), `action: test`** — the harmless self-test path.
     Append a line to the day's journal noting the test approval was
     processed successfully, move the file to
     `memory/approvals/resolved/` with `## Outcome\nApproved via
     Telegram (test) at <timestamp> — pipeline verified, no real action
     taken.` This is what proves the whole chain works before anything
     real is ever approved this way.
   - **`A:` (approve), `action: publish`** — read `platform` and
     `content_package` from front matter. Dispatch to the matching
     skill (read its `SKILL.md` directly if the `Skill` tool isn't
     available in this session, same fallback pattern as `daily-loop`):
     `publish-buffer` (platform starts with `buffer:`), `publish-x`
     (platform `x`), `publish-threads` (platform `threads`). Log any
     real cost via `update-ledger`. On success: move the content
     package to `content/posted/`, move the approval to
     `memory/approvals/resolved/` with the real outcome. **On failure:
     leave the approval in `pending/`, don't mark it resolved, log
     exactly what failed** — a failed publish attempt is not the same
     as a rejection, and shouldn't silently disappear either way.
   - **`A:` (approve), `action: human-manual`** — shouldn't normally
     reach here since these don't get Telegram buttons, but if one
     does (e.g., manually triggered), treat it the same as a `test`:
     record the approval, execute nothing, note the mismatch in the
     journal so `request-approval`/`telegram-notify` can be checked for
     a bug.
7. **Confirm back to Telegram**, always, regardless of branch above:
   - `POST .../answerCallbackQuery` with the `callback_query_id` (clears
     the loading spinner on the button).
   - If a `telegram_message_id` was recorded, `POST
     .../editMessageReplyMarkup` (remove the buttons) and
     `.../editMessageText` (append the resolution, e.g. "✅ Approved and
     published" / "❌ Rejected" / "✅ Test approved — pipeline verified")
     so the chat shows what happened without needing a new message.
8. **Record the update as processed**: append a row to
   `memory/telegram-approvals-log.csv` — `update_id, callback_query_id,
   approval_id, decision, processed_at, notes`. Do this even for
   ignored/unauthorized/already-resolved cases (steps 3 and 5) — the
   point is that no `update_id` is ever actioned twice, not just that
   successful ones aren't.

## Step 4 — record the run

Append a line to `memory/run-log.md` (same file `daily-loop` uses) noting
this was the Telegram-poll Routine specifically, how many updates were
found, and what was processed. Commit and push everything from this run —
same branch-pin discipline as `daily-loop` step 0a.

## Guardrails

- Never execute a publish action without a full loop through
  steps 3-8 having actually found `A:` for that specific approval —
  don't infer approval from anything else (silence, a plain-text
  message, a different approval's button).
- Never mark an approval resolved on a failed publish attempt.
- Never process the same `update_id` twice, under any code path —
  every branch above ends in a `telegram-approvals-log.csv` row.
- Never invent a Telegram API response — if a call fails, that's a
  `partial`/`blocked` run-log outcome, not something to paper over.
