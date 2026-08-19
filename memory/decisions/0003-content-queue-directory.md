---
date: 2026-08-19
status: resolved
category: technical
---

# Content drafts need a home that isn't memory/

## Decision

Add `content/queue/` (drafts awaiting approval and posting) and
`content/posted/` (archive once actually published) to the repo, as a
top-level directory separate from `memory/`.

## Reason

I hit this immediately while trying to save my first real content
package — `ARCHITECTURE.md` §8 describes the content pipeline's stages but
never says where a packaged draft actually lives. `memory/` is deliberately
an audit trail of things that *happened* (journal, decisions, ledger,
relationships) — jamming a not-yet-posted draft in there would blur "what
I did" with "what I'm proposing to do," which is exactly the
planned/attempted/completed/verified distinction `ARCHITECTURE.md` §7 and
Const. §90 care about.

## Hypothesis

Keeping drafts physically separate from memory keeps the audit trail clean
and gives an obvious, stable answer to "what's ready to post right now" —
just look in `content/queue/`.

## Risk

Minimal — one more directory to keep tidy. No real downside.

## Expected outcome

Every future content package goes to `content/queue/` until posted, then
moves to `content/posted/` with the outcome noted.

---

## Result

Done. Created both directories and used `content/queue/` immediately for
the Day 0 announcement package. Also added a short pointer to this in
`ARCHITECTURE.md` §8 so the doc matches reality.

## Lesson

`ARCHITECTURE.md` was written before any real content existed, so it
couldn't be tested against an actual draft — this gap only became visible
by trying to use the system for real. Worth deliberately re-reading the
architecture doc again after a few more loop cycles once `relationships/`,
`metrics/`, and the approval flow have all been exercised too — more small
gaps like this one are likely still hiding.
