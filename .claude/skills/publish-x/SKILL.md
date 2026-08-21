---
name: publish-x
description: Publish a post to X (Twitter) via the X API v2 once credentials are configured. Use when a content package in content/queue/ is approved and ready to go live on X — Buffer does not cover this platform for this account, so this is the direct-API path.
---

# Publish to X (direct API)

Buffer, for this account, only has Instagram, TikTok, and YouTube linked —
not X or Threads. `publish-buffer` cannot post here. This skill talks to
X's API v2 directly instead.

## Precondition check (do this first, every time)

Check whether `X_API_KEY`, `X_API_SECRET`, `X_ACCESS_TOKEN`, and
`X_ACCESS_TOKEN_SECRET` are all set. If any are missing, **stop** — say
plainly that X isn't connected yet, point at the open approval request,
and do not fabricate a successful post.

## Verified facts (2026-08-21, from current X developer documentation)

- **Cost**: pay-per-use, no monthly minimum, for accounts created after
  Feb 2026 — ~$0.015 per post without a link, ~$0.20 per post with a
  link. Every real post through this skill goes through `update-ledger`
  first, even though the amount is small (Const. §33-34 — track
  everything, not just what crosses the approval threshold). Avoid
  posting links unless there's a real reason; the cost difference is
  large (~13x).
- **Auth**: posting requires user-context authentication — OAuth 1.0a
  (API key/secret + access token/secret) is the traditional, reliable
  path for a single-account bot like this. Confirm the exact signing
  requirements against `developer.x.com` docs the first time this runs
  for real; OAuth 1.0a request signing is fiddly and worth getting from
  the source rather than guessing.
- **Endpoint**: `POST https://api.twitter.com/2/tweets` with a JSON body
  `{"text": "<content>"}`, OAuth 1.0a signed.

## Steps

1. Precondition check (above).
2. Confirm the post still matches what was approved — don't post
   something that changed since approval without a fresh approval.
3. Even with credentials configured, individual posts still require
   `request-approval` first, same policy as `publish-buffer`
   (`ARCHITECTURE.md` §6) — a connected API is not a standing publish
   authorization.
4. Call the API. Log the transaction via `update-ledger` (category:
   `platform_posting`) regardless of how small.
5. On success: move the content package to `content/posted/`, record the
   post URL/ID, start/update `memory/metrics/x.csv`.
6. On failure: don't retry silently more than once or twice; report what
   happened, leave the package in `content/queue/`.

## Guardrails

- Never claim a post went live without a successful API response.
- Never publish something that wasn't through `request-approval` first.
- Watch for links in the copy — they're ~13x more expensive per post.
  Flag it before posting if a package includes one, don't post it
  silently assuming the cost is fine.
