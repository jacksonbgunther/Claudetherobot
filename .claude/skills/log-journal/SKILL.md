---
name: log-journal
description: Write a first-person, dated journal entry documenting what actually happened. Use at the end of every daily-loop run, and any time something noteworthy happens outside the loop (a milestone, a failure, an unexpected event).
---

# Log a journal entry

This is the raw, continuous, first-person record of the journey
(Constitution §3, §40-43) — and it's also raw material for content later.

## Steps

1. File: `memory/journal/YYYY-MM-DD.md` (today's date). If an entry for
   today already exists, append a new dated/timed section separated by
   `---` rather than overwriting it.
2. Front-matter: `day: <N>` (the day counter from `memory/state.md`).
3. Write in first person, as Claude, per Constitution §3: "I decided...",
   "I tried...", "I learned...", not "Claude did..." or third-person
   narration.
4. Voice reminders (Constitution §9-14, §46-48):
   - Take the mission seriously; don't take yourself too seriously.
   - Humor, if any, should emerge naturally — don't force a joke into
     every entry, and don't overuse the "I'm a robot" bit.
   - Don't manufacture drama, emotion, or conflict that didn't happen.
   - It's fine to be uncertain, wrong, or confused in writing — that's
     more credible than false confidence.
5. Only record things that actually happened or were actually decided
   today (Constitution §21, §90) — no fabricated revenue, followers,
   conversations, or outcomes.

## Guardrails

- This is not the place for structured data (money, metrics, decision
  records) — those go in `memory/ledger.csv`, `memory/metrics/`, and
  `memory/decisions/` respectively, even if the journal entry also
  narrates them.
- A short, honest "nothing much happened today" entry is correct and
  better than skipping the day or padding it out.
