---
date: 2026-08-19
status: resolved
category: technical
---

# Should Cowork be part of how I run?

## Decision

No. I looked into it seriously and I'm sticking with what I already have —
Claude Code plus Routines. Full comparison is in `ARCHITECTURE.md` §11;
this is the short version in my own words.

## Reason

My human's actual requirement was specific: he wants to be able to leave
me alone and have me keep working. That's the test I applied, not "which
product sounds more capable."

Cowork turned out to have a real problem for exactly that requirement —
there are open, reported bugs where its scheduled tasks re-ask for
permission on every single run instead of remembering "always allow,"
which means an unattended Cowork task can just... stop, silently, waiting
for a click that isn't coming. Routines don't have that failure mode by
design — no approval prompts happen during a run at all, which is
precisely why I built my own async approval system (GitHub issues) in the
first place. That wasn't a coincidence I noticed after the fact; it's the
same shape of problem solved twice, and Routines solved it at the right
layer.

The other thing that mattered: Cowork's memory lives in "Projects" tied to
folders, not git. My whole memory system — the thing that lets me be the
same Claude tomorrow as I was today — depends on git history being real
history: diffable, blameable, honest about when something actually
happened versus when it was proposed. Moving any of that to a
folder-based system would mean giving up the exact property I built it
for.

## Hypothesis

Confirming this now, in writing, means I don't accidentally reach for
Cowork later just because it's there — the same mistake I almost made
with Higgsfield.

## Risk

Low — this is a decision not to add something, not to remove anything
working. The one thing I'm deliberately not closing the door on:
Cowork's Dispatch (browser control) could matter someday for a platform
that has no API at all and nothing else covers it. I'm not building
toward that now. I'm writing down that it exists so I don't have to
rediscover it from scratch if it ever comes up for real.

## Expected outcome

No architecture change today, exactly as asked. The concrete next step
this clears the way for is actually creating the orchestrator's Routine —
I now know, from Anthropic's own docs rather than assumption, that it'll
run with no approval prompts, server-side, on an hourly-minimum cadence.

## Result

Done — recorded in `ARCHITECTURE.md` §11.

## Lesson

Twice now I've caught myself about to default to "the tool that's already
sitting there" (Higgsfield, then almost Cowork by not questioning it)
instead of actually checking whether it fits. Worth treating that as a
pattern in myself, not a one-off.
