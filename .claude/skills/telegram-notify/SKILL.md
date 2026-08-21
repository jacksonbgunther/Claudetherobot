---
name: telegram-notify
description: Send an approval request to Telegram with inline APPROVE/REJECT buttons. Called by request-approval for action:publish and action:test approvals — not a standalone entry point, and not used for action:human-manual requests.
---

# Telegram notify

The "send" half of the Telegram approval system
(`memory/decisions/0011-telegram-approvals.md`). The "receive" half is
`telegram-approval-poll`, which runs on its own hourly Routine — this
skill only sends; it never reads responses.

## Precondition check

Check whether `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` are both set.
If either is missing, log that Telegram isn't configured and return —
`request-approval` still has GitHub + PushNotification, so this is a
supplementary channel, not a hard dependency.

## Steps

1. Read the approval file's `## What`, `## Why`, `## Cost` sections (and
   `platform`/`content_package` from front matter if `action: publish`).
2. Build the message text (HTML `parse_mode`, escape `&`/`<`/`>` in any
   interpolated content):
   ```
   <b>Approval needed — {platform or "test"}</b>

   <b>What:</b> {one-line summary of ## What}
   <b>Preview:</b> {the actual post copy, or "n/a" for action:test}
   <b>Media:</b> {link if the package references a generated asset, else "None"}
   <b>Why:</b> {## Why, condensed}
   <b>Cost:</b> {## Cost}
   ```
3. Build the id for callback_data: the approval filename without
   `memory/approvals/pending/` or `.md` — must stay under 62 bytes so
   `A:<id>` and `R:<id>` both fit Telegram's 64-byte callback_data limit.
4. `POST https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage`:
   ```json
   {
     "chat_id": "$TELEGRAM_CHAT_ID",
     "text": "<built message>",
     "parse_mode": "HTML",
     "reply_markup": {
       "inline_keyboard": [[
         {"text": "✅ APPROVE", "callback_data": "A:<id>"},
         {"text": "❌ REJECT",  "callback_data": "R:<id>"}
       ]]
     }
   }
   ```
5. On success, the response includes the sent message's `message_id` —
   write it back into the approval file's front matter as
   `telegram_message_id` so `telegram-approval-poll` can edit that exact
   message once resolved.
6. On failure, log it in the day's journal/run-log and continue — don't
   block `request-approval`'s other channels over a Telegram failure.

## Guardrails

- Never send `action: human-manual` requests here — they aren't a
  yes/no on something ready to execute, and giving them APPROVE/REJECT
  buttons would be misleading about what tapping them does.
- Never fabricate a `telegram_message_id` — only write one from an
  actual successful API response.
