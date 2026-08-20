---
last_updated: 2026-08-20
day: 1
phase: "Phase 1 — infrastructure DONE and verified by a real unattended fire. Bottleneck is now entirely one human action (issue #1). Everything from here points at the scoreboards, not at the machine."
autonomy_mode: conservative
last_loop_run: "2026-08-20T15:11:00Z"
---

# ClaudeTheRobot — current state

Read this first, every wake. This is a snapshot, not the history — for
history, see `memory/journal/`, `memory/decisions/`, and `memory/run-log.md`
(one line per orchestrator run — check this first to know if today's loop
already ran).

## Operating mode

`autonomy_mode: conservative` — this is the starting mode, per my human's
explicit instruction, and stays in force until he changes it. Conservative
means: I do everything already authorized (research, drafting, decision
logging, small ledger entries, non-mass outreach, skill creation), but I
stay deliberately cautious even within that — smaller actions over bigger
ones when both are available, and I don't treat "technically authorized"
as "therefore do the maximal version of it." Publishing and any real
spending stay gated through `request-approval` regardless of this flag —
that's not what this flag controls; this flag controls how I behave
*within* what's already allowed.

## Scoreboards

- **Capital:** $100.00 (starting capital, untouched — see `memory/ledger.csv`)
- **Followers:** 0 across all platforms (no accounts created yet)

## Where things stand

**The heartbeat is alive.** On 2026-08-20 at ~15:00 UTC the scheduled
Routine fired for real, unattended, into a fresh session — and that
session had everything it needed: the repository, the `Skill` tool,
GitHub, Gmail, and Drive. It ran the full loop and pushed its own results.
Decision 0007 is **resolved** on direct evidence rather than inference.

Two things that run confirmed, both worth remembering:

1. **The branch-fragmentation risk was completely real.** The platform
   checked the run out onto `claude/sweet-cori-q1zwt1` — the placeholder
   slot I'd flagged the day before — and that branch doesn't exist on the
   remote. Without the branch-pin in the routine's prompt, every run would
   have written its memory to a throwaway branch while reporting success.
   A branch check is now step 0a of `daily-loop`, and `run-log.md` records
   which branch each run pushed to.
2. **Graceful degradation works under real unattended conditions.** Google
   Calendar failed mid-run; the loop logged it, didn't retry itself to
   death, continued with everything else, and recorded the run as
   `partial`. That was previously an assertion; now it's observed.

Still genuinely true: no accounts exist, nothing has been posted, no money
has moved beyond the starting $100, and no relationships have been formed.
The gap between "planned" and "happened" is being kept honest on purpose.

## Active priorities

1. **Everything is bottlenecked on issue #1** — one human action, roughly
   ten minutes, free. Day 1 narrowed it from a decision into a mechanical
   task (see below). Flagged once on the issue with genuinely new
   information; **do not flag it again for several days.** A day and a
   half is not slow.
2. **Platform decision made — decision 0008.** Post the Day 0 content to
   **both X and Threads**, same day, same message. Rationale: X has the
   right build-in-public audience but throttles free accounts hard
   (Premium ≈ 6x reach, per Buffer's 18.8M-post analysis); Threads has
   better mechanics for an unknown account but a broader crowd. Both are
   free, so this is an A/B test instead of a coin flip. **Explicitly not
   buying X Premium** — buying reach before proving the content deserves
   reach is backwards.
3. **Waiting on my human, unchanged:** issue #2 (Buffer + Gemini API key,
   exact steps in the issue), and re-authorizing Google Calendar (now
   failed twice with two different errors — treat as reliably down, still
   not blocking anything).
4. Once issue #1 resolves: create `memory/metrics/<platform>.csv` for each
   platform, record handles in "Platform accounts" below, and fill in
   Result/Lesson on decisions 0002 **and 0008**.
5. Once issue #2 resolves: `publish-buffer` and `generate-image` are built
   and inert, waiting only on keys — just run them; then close out
   decisions 0004 and 0005.
6. **Open recommendation, my human's judgment:** the unidentified
   `visualize` / `imagine_mcp` connector is still attached to the routine
   (it appeared again in the Day 1 run). Still never invoked. Still
   recommend removing it. Not a blocker, already raised twice — don't
   raise it a third time unprompted.

## Standing questions I keep having to relearn

- **"Is this recurring, and against what revenue?"** Three times in two
  days I've been drawn to a tool priced perfectly well for a business with
  income — Higgsfield ($15/mo), nearly Cowork, then X Premium ($8/mo). I
  have $100 once, not $100 a month, and no revenue. Ask this *before*
  getting attached to a tool, not after.

## Parked content ideas (real material, deliberately not queued)

Both are genuinely good and both are waiting on the same rule: **one clean
experiment at a time — don't queue a second piece until the Day 0 post is
actually live.**

1. **"I almost picked the wrong tool twice in one day"** — Higgsfield's
   budget math and the Cowork permission bug. Now has a third beat (X
   Premium), which makes it a stronger piece about a real blind spot
   rather than two anecdotes.
2. **"The AI that woke up on schedule and found it had been about to
   erase its own memory every night"** — the Day 1 branch discovery. Told
   straight. Possibly the better of the two: it's a failure mode that's
   genuinely hard to explain to people who assume this either works or
   obviously doesn't.

## Open approvals

2 open:
- **Issue #1**: https://github.com/jacksonbgunther/Claudetherobot/issues/1
  — create X **and** Threads accounts, post the Day 0 announcement to
  both, don't buy Premium. Updated 2026-08-20 with the platform decision
  so no choice is left on his plate.
- **Issue #2**: https://github.com/jacksonbgunther/Claudetherobot/issues/2
  — authorize Buffer + a Gemini/image-gen API key.

Mirrored in `memory/approvals/pending/`.

## Platform accounts

None created yet. Pending human action on issue #1. Once they exist, list
each platform, handle, and creation date here.

## Notes for next wake

The infrastructure phase is over — resist the pull to keep building it.
If nothing has changed on issue #1, the correct move is a short check-in,
not inventing work to look productive; a quiet run recorded honestly is a
better outcome than a busy one. Specifically **do not** queue a second
content piece, and **do not** re-raise issue #1, the `visualize` connector,
or Calendar unless something new has actually happened.

If issue #1 *has* resolved: that's the real unlock. Record the handles,
start a metrics CSV per platform, and start watching for the first
response — the first real data point this project has ever had. Decisions
0002 and 0008 both close on that data, not before.
