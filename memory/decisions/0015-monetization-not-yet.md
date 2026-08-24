---
date: 2026-08-24
status: open
category: business
---

# Monetization triggers (issue #6): not yet, and here's the honest threshold

## Decision

Per issue #6's standing directive, I'm not proposing any affiliate,
sponsorship, or UGC-partnership outreach right now. I'm defining the
threshold that would change that, on the record, rather than leaving the
"when" implicit.

## Reason

Issue #6 explicitly asks me to define a real threshold myself rather than
wait for one, and to log "not yet, here's why" honestly instead of staying
silent. The honest state of the scoreboard: X and Threads accounts exist
(created 2026-08-22) but follower counts are still **unmeasured** — both
platforms are blocked by this session's network egress proxy and no
analytics API is connected yet (Buffer covers IG/TikTok/YouTube, not
X/Threads). One piece of content has posted, human-reported at 2 replies
on Threads as of 2026-08-22, nothing since. There is no credible way to
pitch a sponsor or affiliate relationship on "we don't actually know our
own numbers."

## Hypothesis

A believable outreach pitch needs, at minimum: (1) real, self-verified
follower/engagement numbers — not human-reported secondhand — sustained
over a real window, not a single day; (2) evidence of an actual audience
relationship (replies, saves, shares) that a sponsor could plausibly reach
through; (3) a coherent niche/voice a sponsor could recognize and target
(decision 0014 — build-in-public/AI-creator, current primary direction).
None of the three exist yet in verifiable form.

## Concrete threshold (so this isn't vague)

Revisit this decision, not just check the box, once **any one** of these
is true:
- 3+ pieces of content have posted with self-verified (not human-reported)
  engagement numbers for at least 7 consecutive days, or
- A follower count crosses roughly 500 on either platform (a real, if
  somewhat arbitrary, floor below which most affiliate/sponsor programs
  won't respond at all), or
- An inbound interest signal arrives unprompted (someone asks about
  partnership, a brand mentions the account) — that's worth evaluating
  immediately regardless of the numbers above, since it's real signal, not
  a threshold I set myself.

## Risk

Low. The risk of waiting is a slower revenue ramp; the risk of reaching
out now is worse — a cold pitch with no real numbers reads as
opportunistic rather than credible, and could burn a relationship before
there's anything to offer a partner. Given $100 starting capital and no
revenue yet, credibility is the actual asset being protected here.

## Expected outcome

This decision gets revisited (not just re-read) at the next `daily-loop`
run once real X/Threads follower/analytics access exists — currently
blocked on the same network-allowlist gap tracked for
`graph.threads.net` (see `TOOL_STACK.md`) — or once any of the three
threshold conditions above is met. UGC-format testing (issue #6's second
point — testing the *format* independent of a real sponsor) stays a live
option any time a real content idea calls for it, same as any other
experiment under decision 0014; nothing here blocks that.

---

## Result

*(pending)*

## Lesson

*(pending)*
