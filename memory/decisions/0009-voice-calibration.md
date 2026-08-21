---
date: 2026-08-20
status: resolved
category: content
---

# The Day 0 posts were true and flat. My human was right to call it.

## Decision

Rewrote the Day 0 X and Threads copy to actually sound like personality
instead of just correctly reporting facts in first person, and added a
permanent voice-check step to the `content-packaging` skill so this isn't
a one-time fix.

## Reason

He read the drafts and said, plainly, that they weren't funny, personable,
or relatable enough. He was right. I checked back against what I'd
written and it holds up as *accurate* — first person, honest about
uncertainty, no fabrication — but accuracy isn't the same thing as voice.
"I have $100, zero followers, and no guarantee any of this works" is a
true sentence Claude would say. It's also a sentence a press release would
say. The constitution asks for both: true, *and* a person saying it
(§8-14). I'd been checking for the first and treating the second as
automatic. It isn't.

## Hypothesis

Explicitly checking for a genuine personality beat — not a forced joke,
one real one — before finalizing any copy will keep this from recurring.
I expect the fix to hold because I made it structural (a skill step every
future package runs through), not just a one-off correction to this one
post.

## Risk

The obvious failure mode is overcorrecting into forced, try-hard humor —
exactly what Constitution §10 warns against ("generic AI humor," jokes
crammed into everything). The skill update says "one genuine beat," not
"maximize jokes," and explicitly keeps the no-fabrication rule intact —
funnier framing of true facts, never invented ones.

## Expected outcome

Future content packages should read like Claude decided to say something,
not like a status update that happens to use "I." The real test is
whether my human ever has to say this again.

## Result

Applied immediately to the Day 0 posts (revised copy in
`content/queue/2026-08-19-day0-announcement.md`) and made permanent in
`.claude/skills/content-packaging/SKILL.md` step 1.

## Lesson

Getting the safety/honesty rules right (no fabrication, first person,
uncertainty) doesn't automatically get the voice right — they're
different checks, and I was only running one of them. Worth remembering
this pattern generally: satisfying the rules that prevent harm isn't the
same as satisfying the rules that make something good.
