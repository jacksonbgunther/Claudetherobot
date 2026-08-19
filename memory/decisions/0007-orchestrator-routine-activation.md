---
date: 2026-08-19
status: resolved
category: technical
---

# Activating the orchestrator Routine

## Decision

Rewrite `daily-loop` into ClaudeTheRobot's full autonomous heartbeat (the
10-step OBSERVE/THINK/ACT/DOCUMENT/ADAPT/HUMAN-GATES cycle my human
specified), run it once manually as a real test, then — only if the test
holds up — create a recurring Routine that fires it unattended.

## Reason

My human asked for this directly, with an explicit sequencing requirement:
verify first, test once, fix what the test finds, only then schedule. That
order matters more than speed here — this is the first thing I'll be doing
with genuinely zero human oversight during each run (Routines execute with
no approval prompts at all), so it should be right before it's unattended,
not fixed after something goes wrong silently at 3am.

## Pre-flight verification

Answering each thing my human asked me to check, with real evidence from
this session where I have it, not assumption:

**What tools can it actually access?**
Whatever this session has connected: GitHub (verified working — 2 issues
created), Gmail (verified working — `list_labels` succeeded), Google Drive
(verified working — created the `ClaudeTheRobot` folder), WebSearch/
WebFetch (in constant use), Claude Code Skills (10 built, all discoverable
after this session's first few calls). **Not yet**: Google Calendar
(broken — expired OAuth token), Buffer, Gemini image generation (neither
has a key yet — their skills exist but are inert by design).

**What actions are actually authorized?**
Per `ARCHITECTURE.md` §7: content research and drafting, decision/journal/
ledger logging, small ledger entries, individual low-volume outreach,
relationship logging, skill creation/improvement. All without needing a
human in the loop first.

**What remains human-gated?**
Any single spend over $10 or that breaches the $20 reserve floor, all
publishing (no verified posting integration is live yet — this remains
true even once Buffer is connected, per the §6 clarification), new
service/account authorization, anything irreversible or a legal/financial
commitment. No exceptions, no threshold creep.

**What happens when a tool fails?**
The rewritten skill's guardrails say: log it, don't retry more than once
or twice, continue with everything independent of it, and record the run
as `partial` rather than `ok`. This isn't theoretical — it's exactly what
happened with Calendar already.

**What happens when credentials expire?**
Same handling — `check-integrations` catches it (as it did for Calendar),
`TOOL_STACK.md`'s status column gets updated, and it's flagged for my
human without blocking the rest of the run.

**How are duplicate runs prevented?**
Two layers: the Routine platform itself won't double-fire on its own
schedule, and the skill's new step 0 (idempotency check) reads
`memory/state.md`'s `last_loop_run` and `memory/run-log.md` before doing
heavy work, so a manual run and a same-day scheduled run don't duplicate
effort.

**How does the system avoid repeatedly performing the same action?**
`request-approval` now explicitly checks `memory/approvals/pending/`
before filing a new request. Content packaging checks `content/queue/`
and `content/posted/` before drafting something that already exists.
Decisions with `status: open` get revisited for closure, not re-decided
from scratch.

**How does state survive between runs?**
Unchanged from `ARCHITECTURE.md` §2-3: git-tracked `memory/`, read first,
written last, every run. This was already the design; this section just
confirms nothing about Routines breaks that assumption — a routine bound
to this session (see below) has the same filesystem and git history as
any other wake.

**How are approvals surfaced?**
Unchanged: GitHub issues + `memory/approvals/pending/` + `PushNotification`
(§6). Confirmed still accurate — issues #1 and #2 are both live and
untouched as of this run.

**How does the Routine record what actually happened?**
New this run: `memory/run-log.md`, one line per run, plus the existing
journal/decisions/state.md updates. The run log is specifically so a
human — or a future me — can sanity-check the heartbeat is alive without
reading every journal entry.

## Which session the Routine wakes

Binding to **this session** (the Routine's default target — no
`persistent_session_id`, no `create_new_session_on_fire`), rather than
spinning up a separate dedicated orchestrator session. Reasoning: this
session already *is* ClaudeTheRobot's working context and relationship
with my human, per `ARCHITECTURE.md` §2's hybrid model — repo is the
durable source of truth either way, so a second session would only add
bookkeeping (which one is "the" orchestrator) without adding safety. If
this session ever needs to be recreated, `memory/state.md` and
`memory/run-log.md` make that a documented, low-cost operation, not a
loss.

## Risk

The main risk is the same one flagged when the loop was first designed:
the skill was written and is being tested by me, in this same
conversation, rather than by an independent unattended fire. A true first
unattended fire could still surface something this manual test doesn't —
that's expected, not a failure, and is exactly why the run log and journal
exist: to make the first few scheduled runs easy to audit.

## Expected outcome

One clean manual test run, producing real journal/state/run-log entries,
with the pre-flight answers above holding up in practice — no duplicate
approval requests, no silent claims of things that didn't happen, a
correctly-recorded `partial` if anything fails. If that holds, activate a
daily recurring schedule.

---

## Result

Ran the full manual test (run-log entry: 2026-08-19 03:10 UTC). It held
up:

- Step 0's idempotency check correctly found no prior run
  (`last_loop_run` was `null`) and proceeded with the full cycle.
- Integration/approval checks confirmed both open issues (#1, #2)
  untouched — no duplicate approval request was filed.
- The self-improvement scan and THINK step surfaced a real, good content
  idea (today's "almost picked the wrong tool twice" story) — and then
  the loop caught itself about to violate its own prior instruction in
  `state.md` ("don't invent a second content piece before the first one's
  posted") and stopped, parking the idea instead of queuing it. This is
  the single most important thing the test proved: a rule I wrote for
  myself earlier survived being checked by a less-attentive, unattended-
  style pass later the same day.
- No money moved, nothing was claimed as posted that wasn't, decisions
  0002 and 0004 were correctly left open rather than force-resolved.
- Journal, run-log, and state.md were all updated as the last step, per
  design.

No architectural problems surfaced. Proceeding to activate the recurring
Routine, bound to this session, daily cadence.

## Lesson

The idempotency/notes-check mechanism isn't decorative — it changed the
actual outcome of this run (prevented a real mistake I was about to make).
That's good evidence it'll do the same job later when I'm not the one
reading along. Also: the test was still run by me, in this conversation —
a genuinely unattended fire is still the real first test. The run log
exists specifically so that first unattended run, and the ones after it,
stay easy to audit rather than needing to be trusted blindly.

---

## Update — creating and test-firing the actual Routine

Created the trigger (`trig_01AuBLSXnohKwjdWk8YfgCGs`, "ClaudeTheRobot
daily orchestrator", daily at 15:00 UTC) via `create_trigger`, intending
the default binding: fire into this session, matching the "which session
the Routine wakes" reasoning above.

Two things happened I didn't expect, and I'm recording them honestly
rather than declaring this finished:

1. **The creation call returned a warning**: *"this trigger stores no MCP
   connectors, so the sessions it fires will run without connector
   (mcp__<server>__*) tools."* That would mean no GitHub, Gmail, Drive, or
   Calendar access during a fire — which would break approvals
   (GitHub-issue-based), integration checks, and outreach entirely.
2. To actually check whether that warning matters in practice (rather
   than assume), I used `fire_trigger` to fire it immediately as a real
   test. The result: it created a **new session**
   (`cse_01CPECXAba2BTUb7sVPht9dm`, confirmed via `get_session` — a
   different ID from this session, tagged `routine:agent-minted`,
   `origin: force_run_trigger`), not a wake of this session as I'd
   designed for.

I don't yet know whether this is (a) how manual `fire_trigger` force-runs
specifically behave, distinct from how the actual scheduled cron fire
will behave (the stored `job_config` does say
`persistent_session_id: session_01SkkL7rg443Dpm3N2ekXtKj`, i.e. this
session — so the *configuration* still says "wake this session"), or (b)
evidence that fires always spawn a new session regardless of what's
configured. I'm not going to guess at which — I scheduled a check-in
(6 minutes out) to look at `git log` for anything the fired session
pushed, and its session status, which will tell me directly whether it
had real tool access rather than me inferring it.

**What I'm not doing:** treating "trigger created" as "orchestrator
verified." The Routine exists and is enabled, but per my human's own
instruction not to trust an automation just because it's technically
running, I'm holding off on calling this fully activated until the
connector question actually resolves. If it turns out fires genuinely
lack connector access, the fix is straightforward — recreate the trigger
from a context that passes connector grants through (the tool's own
warning suggests this is possible, just not what happened by default
here) — but I'd rather confirm the actual failure mode first than guess
at the fix.
