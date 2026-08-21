---
date: 2026-08-21
status: resolved
category: technical
---

# Auditing 7 platforms for real audience interaction, not just publishing

## Decision

Before building anything toward reading comments, replying, or watching
engagement, audit all seven platforms in scope — Instagram, TikTok,
YouTube, X, Threads, Facebook, Snapchat — against current official
documentation, across auth, posting, comment-reading, replying,
analytics, real-time events, dev requirements, scopes, approval process,
and cost. No implementation, no accounts, no spend — research and policy
only, per my human's explicit instruction.

## Reason

He asked me not to assume any API supports something, and to verify
against current docs — the right instinct given how much has already
shifted mid-project (X's pricing changed twice in this conversation
alone). Doing this audit before any design work means the eventual
comment/reply skills get built against confirmed capabilities, not
assumptions that turn out wrong after the fact.

## What the audit actually found

Full detail in `TOOL_STACK.md`; the two things worth stating as findings
rather than data points:

1. **The webhook problem I solved once for Telegram is universal, not a
   Telegram quirk.** Instagram, Facebook, Threads, and X all offer real
   webhooks for comments/mentions; YouTube offers PubSubHubbub. None of
   them can reach this architecture directly — same root cause as
   Telegram (decision 0011): firing a Routine externally requires
   Anthropic-specific auth and a fixed payload shape no third-party
   webhook can produce. I could have rediscovered this seven times,
   platform by platform, the hard way. Generalizing it once means every
   future comment/mention integration uses the same hourly-polling
   pattern already built and proven for Telegram, by design, not by
   accident.
2. **Snapchat isn't a low priority — it's not currently possible.** There
   is no public API for organic posting, comments, or engagement at all,
   only a Marketing/ads API. I'm not deprioritizing it as a judgment
   call; there's genuinely nothing to connect to yet.

Also confirmed rather than assumed: TikTok's comment/reply capability
isn't clearly documented anywhere I could verify in this pass. Recorded
as unverified, not as "probably fine" or "probably missing" — an honest
unknown, to be resolved by direct investigation before any TikTok
interaction skill gets designed.

## Hypothesis

Recommended connection order (X/Threads accounts first, then YouTube
native comments/replies, then Threads reply/mention scopes, then
Instagram, then Facebook, then a TikTok capability check, Snapchat not
pursued) sequences by actual friction and value rather than platform
popularity — YouTube in particular turns out to be the cheapest real path
to genuine audience interaction of anything audited, which wasn't obvious
going in.

## Risk

Low — this is research and documentation, no accounts created, nothing
spent, nothing built that could be wrong in a costly way. The real risk
is documentation going stale, the same risk every platform audit in this
project has had — X's terms alone have shifted twice already. Worth
re-verifying anything here against current docs before actually building
against it, not trusting this file indefinitely.

## Expected outcome

When audience interaction actually gets built (not now — nothing's live
to interact with yet), it should be able to start directly from
`TOOL_STACK.md`'s per-platform detail instead of re-researching, and the
polling pattern should already be the assumed default rather than
something to re-litigate per platform.

## Result

Done — `TOOL_STACK.md` has the full seven-platform audit and connection
order; `ARCHITECTURE.md` §12 records the generalized webhook constraint
and the confirmed capability gaps.

## Lesson

The value of doing all seven at once, rather than one at a time as each
becomes relevant, was specifically catching the pattern (webhooks don't
work anywhere) and the absence (Snapchat has nothing to connect to)
before either got rediscovered piecemeal. A few individual audits would
each have looked like isolated findings; together they're a structural
constraint and a genuine dead end, both worth knowing before, not during,
implementation.
