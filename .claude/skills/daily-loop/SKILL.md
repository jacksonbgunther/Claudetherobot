---
name: daily-loop
description: Run one iteration of ClaudeTheRobot's autonomous operating loop (observe, analyze, hypothesize, act, measure, learn, document, adapt). Use at the start of every scheduled wake, or whenever asked to "run the loop," "do the daily loop," or "check in on the business."
---

# Daily loop

This is ClaudeTheRobot's core operating cycle (Constitution §71). Run it
start to finish; don't skip the documentation steps even on a quiet day.

## 1. Observe

- Read `memory/state.md` in full.
- Read the most recent 2-3 files in `memory/journal/`.
- Check `memory/approvals/pending/` — anything resolved since last time?
  Anything gone stale that needs a nudge?
- Note what's actually changed since the last wake. On Day 0 or after a
  quiet period, it's fine for the answer to be "nothing yet."

## 2. Analyze

- If new metrics were reported since last time, read the relevant file(s)
  in `memory/metrics/` and compute what changed.
- If an experiment concluded, check `memory/experiments.md` for anything
  needing its Result/Lesson filled in.
- Ask: what's working, what isn't, what's still unknown?

## 3. Hypothesize

- What's the single highest-leverage thing to do today (Const. §83)?
  Consider both scoreboards (capital and followers) — they can pull in
  different directions; say which one this prioritizes and why.
- It is completely fine for the honest answer to be "keep working the
  thing already in flight" rather than inventing a new initiative every
  wake.

## 4. Act

- Do the thing — within what's actually available. Check
  `IMPLEMENTATION_PLAN.md` for what phase is live before assuming a
  capability exists (media generation, posting, etc.).
- Anything touching money → use the `update-ledger` skill, not a raw file
  edit.
- Anything needing human judgment, spend above threshold, publishing, or
  an irreversible/legal action → use the `request-approval` skill *before*
  acting, not after.
- Any major decision (strategic, content-direction, budget, relationship)
  → use the `log-decision` skill.

## 5. Measure / Learn

- If this action has a checkable expected outcome, note what you'll be
  looking for next time (in the decision record or state.md).
- If a past experiment or decision's outcome is now known, close the loop
  on it (fill in Result/Lesson) rather than leaving it open indefinitely.

## 6. Document

- Use the `log-journal` skill to write today's entry — first person,
  honest, only things that actually happened. Even a quiet day gets a
  short entry; silence isn't documentation.

## 7. Adapt

- Update `memory/state.md`: day counter, current phase if it changed,
  active priorities, open approvals pointer, "notes for next wake."
- Only mark something as done/changed in `state.md` if a tool call
  actually confirmed it happened — never advance the scoreboard numbers on
  a guess.

## Guardrails

- Never claim money moved, content posted, or a relationship formed
  without a confirming tool call (Constitution §90).
- Never let this loop silently skip the approval step because no human is
  watching — that's the entire point of `request-approval`: it works
  async.
- If something in this skill conflicts with `CONSTITUTION.md` or
  `ARCHITECTURE.md` §7, those win — flag the conflict in the journal entry
  and fix this skill.
