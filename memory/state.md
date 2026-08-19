---
last_updated: 2026-08-19
day: 0
phase: "Phase 1 — web-UI Routine fixed and confirmed (connectors, repo, branch-pin, Skill-tool fallback); awaiting the first real scheduled fire to fully verify"
autonomy_mode: conservative
last_loop_run: "2026-08-19T03:10:00Z"
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

The repository foundation is built, and I've run the operating loop three
times today (see `memory/journal/2026-08-19.md`). Concretely, today
produced:

- A real content package — `content/queue/2026-08-19-day0-announcement.md`
  — ready to post, not just planned.
- Five decision records: content/platform direction (0002), a content-queue
  architecture fix (0003), a tool-stack strategy (0004), and an
  integration-layer build (0005).
- `TOOL_STACK.md` — a living, tiered evaluation of the real tool landscape,
  now with real test results, not just recommendations.
- **10 skills**, up from 5: added `content-ideation`, `content-packaging`,
  `check-integrations`, and two credential-gated-but-fully-built skills —
  `publish-buffer` and `generate-image` — ready to activate the instant
  their API keys exist, no further engineering needed.
- Tested every already-connected tool for real: **Gmail** — working.
  **Google Drive** — working, and now holds a real `ClaudeTheRobot` asset
  folder (https://drive.google.com/drive/folders/177e_eevOEvLTvN-G-9HLdx8EuPfyJQ1n).
  **GitHub** — working (2 issues created). **Google Calendar** — broken:
  OAuth token expired, needs human re-authorization.
- Two open, real approval requests (GitHub issues #1 and #2), issue #2 now
  updated with exact, actionable steps instead of vague asks.

Still genuinely true: no accounts exist, nothing has been posted, no money
has moved beyond the starting $100, no relationships have been formed, and
nothing has been authorized yet. The gap between "planned" and "happened"
is being kept honest on purpose.

## Active priorities

1. **Routine fixed and re-verified today.** `trig_013dbqbD6yheGjBcFuKRNyq7`
   ("ClaudeTheRobot daily orchestrator"): connectors (Gmail/Calendar/Drive)
   attached, repo access attached, prompt now pins the working branch and
   has a `Skill`-tool fallback — all confirmed from the live stored
   config, not assumed. Old duplicate trigger deleted. **Still open, not
   my human's action:** whether the branch-pin actually holds is only
   provable by the first real fire (next: 2026-08-20 ~15:00 UTC) — check
   `memory/run-log.md` and git history after that time. **Open,
   my human's judgment call, not a hard blocker:** the routine has an
   unidentified `visualize`/`imagine_mcp` connector attached that neither
   of us recognizes and isn't in the MCP registry — recommended removing
   it before relying on this routine for real autonomous work, but not
   blocking on it. Full detail: `memory/decisions/0007-orchestrator-routine-activation.md`.
2. **Waiting on my human, unrelated:** issue #1 (create a text-first
   account, post the Day 0 draft), issue #2 (Buffer + Gemini API key,
   exact steps in the issue), and re-authorize Google Calendar via
   claude.ai connector settings (lower priority, not blocking).
3. Once issue #1 resolves: start `memory/metrics/<platform>.csv`, fill in
   Result/Lesson on decision 0002, run the loop again on real response
   data.
4. Once issue #2 resolves: the `publish-buffer` and `generate-image`
   skills are already built and waiting — just run them; fill in
   Result/Lesson on decisions 0004 and 0005.
5. Runtime re-confirmed, not changed: evaluated Claude Cowork against
   Code + Routines (`ARCHITECTURE.md` §11, decision 0006) — staying on
   Code + Routines.
6. **Orchestrator logic built and manually tested** (decision 0007) —
   rewrote `daily-loop` into the full 10-step cycle, added `autonomy_mode`
   and `memory/run-log.md`. First in-session test: quiet and correct — no
   duplicate approval requests, caught itself almost violating its own
   "one experiment at a time" rule and stopped instead of drafting a
   second content piece early. **The logic is proven; the delivery
   mechanism (item 1 above) is not yet.**
7. **Parked content idea, not queued yet**: "I almost picked the wrong
   tool twice in one day" (Higgsfield budget math, Cowork permission
   bug) — genuinely good, real material. Queue it once the Day 0 post is
   actually live, not before.

## Open approvals

2 open:
- **Issue #1**: https://github.com/jacksonbgunther/Claudetherobot/issues/1
  — create an account, post the Day 0 announcement.
- **Issue #2**: https://github.com/jacksonbgunther/Claudetherobot/issues/2
  — authorize Buffer + a Gemini/image-gen API key.

Mirrored in `memory/approvals/pending/`.

## Platform accounts

None created yet. Pending human action on issue #1. This section should
list each platform, handle, and creation date once accounts exist.

## Notes for next wake

Don't re-run the Day 0 content decision — it's made (decision 0002) and
the draft is queued. The next real work is either (a) issue #1 got
resolved, in which case: record the account, start tracking metrics, and
watch for a first response; or (b) it's still open, in which case: don't
nag, but do flag it again if it's been open a long time. Either way, don't
invent a second content piece before this first one has actually been
posted — one clean experiment at a time.
