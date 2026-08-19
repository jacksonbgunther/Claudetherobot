---
name: log-decision
description: Record a major decision in ClaudeTheRobot's memory using the Constitution's decision-record format. Use whenever making a strategic, business, budget, content-direction, or relationship decision worth remembering — not for routine, reversible micro-choices.
---

# Log a decision

Implements Constitution §72 ("document my decision-making").

## When to use this

Use for decisions with real stakes or that you'd want to be able to look
back on: pivoting strategy, choosing a content direction, a budget
allocation, starting or ending a relationship, a technical/process change.

Don't use it for every small reversible choice — that would bury the
decisions that matter. Routine execution details belong in a journal entry
if anywhere.

## Steps

1. Copy `memory/decisions/TEMPLATE.md` to
   `memory/decisions/<NNNN>-<short-slug>.md`, where `<NNNN>` is the next
   number after the highest existing one in `memory/decisions/` (zero-padded
   to 4 digits) and `<short-slug>` is a few kebab-case words.
2. Fill in, in first person, as Claude:
   - **Decision** — what you're doing.
   - **Reason** — why.
   - **Hypothesis** — what you expect to happen.
   - **Risk** — what could go wrong, and the real downside if you're
     wrong.
   - **Expected outcome** — specific enough to actually check later, not
     vague optimism.
3. Set front-matter `status: open`.
4. Leave **Result** and **Lesson** blank — fill those in during a later
   `daily-loop` run once the outcome is actually known, then set
   `status: resolved`.

## Guardrails

- Don't write a decision record for something that didn't really happen or
  hasn't really been decided yet — a half-formed idea belongs in a journal
  entry, not here.
- If the decision involves spending above the threshold in
  `ARCHITECTURE.md` §7, or an irreversible/publishing action, this record
  does not substitute for `request-approval` — do both.
- Revisit open decisions during `daily-loop` rather than letting them sit
  unresolved indefinitely.
