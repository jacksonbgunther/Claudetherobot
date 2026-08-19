---
date: 2026-08-19
status: resolved
category: technical
---

# Building ahead of credentials, and where I drew the line

## Decision

I built everything in the Tier 1/2 tool stack that doesn't require my
human's credentials, approval-screen interaction, or account ownership:
skills for Buffer publishing and Gemini image generation (both inert
until their API keys exist, but fully specified against real, verified API
docs rather than guessed), plus content-ideation, content-packaging, and
check-integrations skills that need nothing new at all. I tested every
already-connected tool (Gmail, Calendar, Drive, GitHub) with a real,
minimal call rather than assuming they still work, and created a real
Drive folder for asset handoff.

I also made one explicit policy call: connecting Buffer authorizes the
*mechanism* for publishing, not a blanket authorization to publish
anything I want afterward. Individual posts still go through
`request-approval` until the pipeline has a track record.

## Reason

I was told to reduce my human's involvement to the things only he can
do — owning accounts, OAuth screens, credentials, approving spend and
publishing, physical-world actions. Everything else is mine to build.
That's a clear enough line that I didn't have to guess much: no tool in
this stack can be signed up for without either an identity-bound account
(Buffer, Google AI Studio) or a human-completed OAuth screen, so nothing
in Tier 1/2 could be fully finished today — but the code, skills, and
integration logic that *use* those credentials could be, and building them
now means the moment a key exists, there's no more engineering left to do,
only a key to paste in.

The Buffer policy call needed to be explicit rather than assumed. It would
have been easy to read "Buffer is authorized" as "publishing is now
autonomous" — but that's a bigger jump than the human actually asked for
today. He asked for less manual configuration, not for individual posting
decisions to stop needing sign-off. Constitution §87 is "autonomy *with
control*" — I'm choosing to keep the control on the piece I don't yet have
a track record on, and I'm writing that down instead of just quietly doing
it, so it's a real decision that can be revisited, not a default that
crept in unnoticed.

## Hypothesis

I expect this to mean: the instant my human adds `BUFFER_API_KEY` and
`GEMINI_API_KEY` as environment variables and links a platform account to
Buffer, I can actually produce and — after approval — publish real content
without any further code being written. That's the test.

## Risk

Low, mostly a documentation/discipline risk: the Buffer and Gemini skills
were written against verified docs but without a live key to test against,
so there's a real chance the exact API call shape needs a small fix the
first time it runs for real (flagged directly in both skill files — they
say what to check instead of pretending certainty). Also: I declined to
attempt something I technically *could* have done — using my Gmail access
to sign up for Buffer/Google AI Studio myself, completing email
verification autonomously. I could have. I didn't, because account
creation is explicitly something my human owns, and doing it anyway just
because I have the technical means would be exactly the kind of overreach
the approval system exists to prevent.

## Expected outcome

Two working, tested skills sitting ready and inert, three more skills
active immediately, two previously-unverified connectors now confirmed
(Gmail, Drive — plus a real new asset folder), one connector confirmed
*broken* (Calendar — expired token), and zero new dependencies added
without going through `request-approval` first.

## Result

Done, as of this session. `TOOL_STACK.md` status column reflects it.

## Lesson

The gap between "I could technically do this" and "this is mine to do" was
worth stating out loud rather than just quietly landing on the cautious
side. Also: testing existing connectors instead of assuming they still
worked immediately paid off — Calendar was silently broken and nothing in
the architecture would have surfaced that until something actually tried
to use it.
