---
date: 2026-08-21
status: open
category: technical
---

# Telegram approvals: real, but polling, not push — and here's exactly why

## Decision

Add Telegram as a second approval channel — tap-to-approve/reject
messages for `action: publish` and `action: test` requests — built on top
of the existing GitHub-issue approval system, not alongside it as a
separate mechanism. Receiving my human's response works via an hourly
polling Routine (`telegram-approval-poll`), not a webhook, because a
webhook genuinely isn't possible without adding new infrastructure I
wasn't asked to build.

## Reason

My human asked for something specific and unusually rigorous: verify the
actual delivery mechanism before building anything, and if it can't work
for real, say so instead of shipping something that looks like it works.
That instinct was correct — it wasn't a hypothetical concern.

I checked, not assumed: Claude Code Routines can be fired externally via
an API endpoint, but that endpoint requires `Authorization: Bearer
<routine_token>` plus Anthropic-specific headers and a body shaped
exactly `{"text": "..."}`. Telegram's webhook mechanism sends its own
fixed JSON schema with no way to add those headers. The two literally
cannot talk to each other directly. A real, permanent relay service would
be needed to bridge them for instant delivery — and that's a genuine new
dependency (an account, a token, something to keep working), exactly the
kind of bespoke infrastructure this project has avoided unless actually
forced to add it.

Polling doesn't have that problem. `getUpdates` is a plain call any
Routine can make on its own schedule, using only what already exists. The
cost is latency — up to about an hour between a tap and it being
processed, not instant — which I think is the right trade for this use
case. Nobody needs to approve a social media post within seconds of me
asking.

## Hypothesis

An hourly poll is fast enough that approvals don't feel stuck, and the
whole thing works with Claude Code completely closed, which was the
actual requirement — "asynchronous" was never specified as "instant."

## Risk

The main risk is exactly the one this design is built to prevent: double-
processing the same response. I addressed it in two independent layers —
every Telegram `update_id` is permanently recorded in
`memory/telegram-approvals-log.csv` before anything else happens, and a
missing/already-resolved approval file is itself treated as a safe no-op.
Both would have to fail at once for something to run twice.

Secondary risk: this is two new skills and a second Routine to keep
correct, on top of everything else. I scoped it tightly — Telegram only
applies to real yes/no execution decisions (`publish`, `test`), not to
every approval type, so it doesn't become the single point of failure for
the whole approval system. GitHub stays authoritative; Telegram is a
faster, phone-friendly way to respond to it.

## Expected outcome

Once my human sets up the bot token, chat ID, the network allowlist entry,
and the second Routine, the harmless test approval (issue #3) should send
itself automatically on the next run of either Routine, and tapping
APPROVE or REJECT on it should produce a visible confirmation back in
Telegram and a matching entry in `memory/telegram-approvals-log.csv` —
proof the whole chain works before it's ever used for something real.

## Result

*(pending — depends on setup and the first real test-approval round trip)*

## Lesson

*(pending)*
