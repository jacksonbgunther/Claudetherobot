---
date: 2026-08-21
status: open
category: technical
---

# Buffer doesn't cover X or Threads for this account. Go direct.

## Decision

Build direct integrations to X's API and Meta's Threads API instead of
routing those two platforms through Buffer. Buffer stays the path for
Instagram, TikTok, and YouTube — that's genuinely what my human has
linked there.

## Reason

I'd been assuming Buffer would eventually cover every platform once
accounts existed. My human corrected that directly — Buffer only has
Instagram, TikTok, and YouTube connected, and that's not changing. For
X and Threads specifically, going direct to their own APIs is the only
path, not a fallback.

Before building anything I checked current cost and access, rather than
trusting what I remembered about either API (X's pricing in particular
has changed multiple times and I didn't want to repeat the Higgsfield
mistake — assuming a cost structure instead of checking it):

- **X**: as of Feb 2026, the free tier is gone for new developers and the
  flat $200/mo Basic tier is closed to new signups too — but pay-per-use
  is now the default, at ~$0.015/post with no monthly minimum. For our
  volume that's cents, not a subscription. It fits the same budget logic
  as the image-gen decision (0004): pay for what's actually used.
- **Threads**: free, no pricing tier at all. Normally gated behind Meta's
  App Review (days, often rejected), but since this app only ever needs
  to post to one account — my human's own — adding him as a "tester"
  grants full access immediately, no review wait.

Neither requires a recurring subscription. Both are self-serve, human-only
account/app creation (identity-bound, same reasoning as every other
account-creation step this project has kept on my human's side).

## Hypothesis

Once the credentials exist, `publish-x` and `publish-threads` (built now,
inert without them, same pattern as `publish-buffer`/`generate-image`)
give me real publishing access to X and Threads with no further
engineering — matching how Buffer's key needed zero extra work once it
was actually set.

## Risk

Low on cost (pay-per-use, no subscription trap). The real risk is
complexity: two more API integrations to keep working, each with its own
auth model (OAuth 1.0a for X, Meta's Graph API token flow for Threads).
Both skills flag the exact request shape as unverified until a real key
exists and a first call actually confirms it — same honesty discipline as
`publish-buffer` and `generate-image` when they were built.

## Expected outcome

My human creates the X Developer Portal app + Threads Meta app (only he
can — account/identity-bound), provides the resulting credentials as
environment variables (never in chat, per the Buffer-key precedent), and
`publish-x`/`publish-threads` go from inert to usable with no more code
from me.

## Result

*(partial, interim — 2026-08-23)* Threads credentials (`THREADS_ACCESS_TOKEN`,
`THREADS_USER_ID`) are now set as of this check-in — real progress, not yet
usable. A real `GET /v1.0/{user-id}` call to `graph.threads.net` failed with
`403` at this session's network egress proxy (`connect_rejected`), so the
token itself is still unverified. This is a network-allowlist gap, not a
credential problem — the same class of block that held up Gemini's endpoint
until `generativelanguage.googleapis.com` was added to the allowlist on
2026-08-20. Needs `graph.threads.net` added the same way before
`publish-threads` can go from inert to usable. X API credentials still don't
exist at all — that half of this decision is unchanged.

## Lesson

*(pending — will close once Threads is actually verified live and, ideally,
X credentials exist too)*
