---
name: content-ideation
description: Generate grounded content ideas by combining ClaudeTheRobot's own experiment history with real trend research. Use during daily-loop's HYPOTHESIZE step, or any time fresh content ideas are needed.
---

# Content ideation

Needs no new credentials — built entirely on tools already available
(WebSearch/WebFetch and `memory/experiments.md`).

## Steps

1. Read `memory/experiments.md` and the last few `memory/journal/`
   entries — what's already been tried, what worked, what's still open.
2. Read `memory/decisions/0002-content-and-platform-direction.md` for the
   current content direction (don't contradict it without a new decision
   record if pivoting).
3. Do real trend research with WebSearch/WebFetch — not generic
   brainstorming. Look for: what's actually getting engagement in the
   build-in-public / AI-creator space right now, what formats are
   working, what's oversaturated.
4. Generate 3-5 concrete ideas, each with: the hook, why it fits the
   current direction, what it would take to produce (text-only? needs an
   image? needs Buffer/image-gen once connected?), and a rough guess at
   why it might work.
5. Don't force all 3-5 into content packages — pick the strongest 1 and
   use `content-packaging` on it, or log the rest in the day's journal
   entry as ideas for later so they're not lost.

## Guardrails

- Ground ideas in real research, not invented trend claims — if citing
  "what's working right now," it should come from an actual WebSearch
  result, not assumption.
- Don't let this skill produce ideas that require tools that aren't
  connected yet as if they were — check `TOOL_STACK.md` status first.
