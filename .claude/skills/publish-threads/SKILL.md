---
name: publish-threads
description: Publish a post to Threads via Meta's Threads API once a token is configured. Use when a content package in content/queue/ is approved and ready to go live on Threads — Buffer does not cover this platform for this account, so this is the direct-API path.
---

# Publish to Threads (direct API)

Same situation as X: Buffer only has Instagram, TikTok, and YouTube linked
for this account, not Threads. This skill talks to Meta's Threads API
directly.

## Precondition check (do this first, every time)

Check whether `THREADS_ACCESS_TOKEN` and `THREADS_USER_ID` are both set.
If either is missing, **stop** — say plainly that Threads isn't connected
yet, point at the open approval request, and don't fabricate a post.

## Verified facts (2026-08-21, from current Meta developer documentation)

- **Cost**: $0. No pricing tier exists for the Threads API.
- **Access**: because this app only ever posts to one account (my
  human's own Threads profile, added as a "tester" under the app's App
  Roles), it doesn't need Meta's public App Review process — testers get
  full permission scopes immediately. If posting ever starts failing with
  a permissions error, check whether the app or the tester role expired
  before assuming something else broke.
- **Flow**: Threads publishing is two calls — create a media container,
  then publish it. Confirm the exact current endpoint shapes against
  Meta's Threads API docs the first time this runs for real; Graph API
  versions shift and guessing the wrong version number is a common,
  avoidable failure.

## Steps

1. Precondition check (above).
2. Confirm the post still matches what was approved.
3. Even with credentials configured, individual posts still require
   `request-approval` first, same policy as `publish-buffer` and
   `publish-x` (`ARCHITECTURE.md` §6).
4. Call the API (container create, then publish).
5. On success: move the content package to `content/posted/`, record the
   post URL/ID, start/update `memory/metrics/threads.csv`.
6. On failure: don't retry silently more than once or twice; report what
   happened, leave the package in `content/queue/`.

## Guardrails

- Never claim a post went live without a successful API response.
- Never publish something that wasn't through `request-approval` first.
- $0 cost doesn't mean skip `update-ledger` — still log it (amount: 0)
  for a complete record of what was actually published where.
