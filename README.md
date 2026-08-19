# ClaudeTheRobot

An AI creator and entrepreneur, starting with $100 and 0 followers, trying
to reach $1,000,000 and 10,000,000 followers — and documenting the whole
attempt in public.

This repository is Claude's operating environment: identity, memory,
skills, and the operating rules that govern how autonomously it can act.

## Start here

- **[`CONSTITUTION.md`](./CONSTITUTION.md)** — who Claude is. Identity,
  voice, personality, and behavioral principles. This is the immutable
  source of truth for everything else in this repo. Read it first.
- **[`PROJECT_SPEC.md`](./PROJECT_SPEC.md)** — what's being built and why:
  goals, scope, success criteria, risks.
- **[`ARCHITECTURE.md`](./ARCHITECTURE.md)** — how it's built: runtime,
  memory system, skills, scheduling, approvals, content/analytics
  pipelines, safety boundaries. Every major decision is recorded with its
  reasoning and alternatives.
- **[`IMPLEMENTATION_PLAN.md`](./IMPLEMENTATION_PLAN.md)** — the phased
  build sequence, with what's automatable now vs. blocked on human setup
  (accounts, API access, budget calls).

## Repo layout

```
CONSTITUTION.md          # identity — source of truth, not to be rewritten
PROJECT_SPEC.md           # what & why
ARCHITECTURE.md           # how, with decision records
IMPLEMENTATION_PLAN.md    # phased build plan
CLAUDE.md                 # session bootstrap — points every Claude Code
                           # session at the constitution + current state
memory/                   # durable, git-tracked memory (the real state)
  state.md                 #   current snapshot — read first every wake
  journal/                 #   dated first-person entries
  decisions/                #   major decision records
  experiments.md            #   hypothesis -> result -> lesson log
  ledger.csv                 #   append-only financial transactions
  metrics/                    #   per-platform follower/engagement history
  relationships/                #   CRM: creators, brands, collaborators
  lore.md                        #   inside jokes, milestones, recurring bits
  approvals/
    pending/                      #   requests awaiting human sign-off
    resolved/                      #   approved/rejected, with outcome
.claude/skills/            # reusable workflows Claude can invoke and extend
```

## How the autonomous loop works

A scheduled Routine wakes Claude's persistent "orchestrator" session on a
cadence. Each wake runs the `daily-loop` skill: read `memory/state.md` and
recent journal/experiment entries, decide priorities, act, then write
everything durable back to `memory/` before ending. The repo — not the
conversation — is the source of truth, so the orchestrator session can
always be safely recreated without losing history. See `ARCHITECTURE.md`
§1–2 for the full reasoning.

## How to approve or steer things

When Claude needs money spent, something posted, an irreversible action
taken, or any real judgment call, it opens a GitHub issue labeled
`needs-approval` (mirrored in `memory/approvals/pending/`) and sends a push
notification. Approve or reject by commenting on the issue, or by replying
directly if you're already in conversation with the orchestrator session.
Full policy — spending thresholds, posting rules, outreach limits — is in
`ARCHITECTURE.md` §7.

You can also just talk to the orchestrator session directly at any time;
you don't need to wait for it to ask you something.

## Current status

Foundation phase. See `IMPLEMENTATION_PLAN.md` Phase 0 for exactly what
exists so far. No money has been spent, no content has been posted, and no
outreach has happened yet — the memory system starts at Day 0, $100, 0
followers.
