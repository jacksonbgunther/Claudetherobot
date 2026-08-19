# IMPLEMENTATION_PLAN.md — ClaudeTheRobot

Phased build plan. Each phase lists what gets built, what's automatable by
Claude alone, and what's blocked on the human (credentials, accounts,
verification, money). See `ARCHITECTURE.md` for the design each phase
implements, and `PROJECT_SPEC.md` for scope.

Status legend: ✅ done · 🔜 next · ⏳ blocked on human · ⬜ not started

---

## Phase 0 — Foundation (this pass)

Goal: the repo can hold ClaudeTheRobot's memory, identity, and safety rules
correctly, before any real business action is taken.

| Task | Automatable by Claude | Status |
|---|---|---|
| Write `PROJECT_SPEC.md`, `ARCHITECTURE.md`, `IMPLEMENTATION_PLAN.md`, `README.md` | Yes | ✅ |
| Root `CLAUDE.md` pointing every session at the constitution + memory index | Yes | 🔜 |
| `memory/` directory: `state.md`, `journal/`, `decisions/`, `ledger.csv`, `metrics/`, `relationships/`, `experiments.md`, `lore.md`, `approvals/{pending,resolved}` | Yes | 🔜 |
| Seed skills: `daily-loop`, `log-decision`, `log-journal`, `update-ledger`, `request-approval` | Yes | 🔜 |
| `.gitignore` for local/secret config | Yes | 🔜 |
| Commit and push foundation | Yes | 🔜 |

Nothing in this phase spends money, posts content, or contacts anyone —
it's pure scaffolding, matching the user's explicit instruction not to
build everything at once.

---

## Phase 1 — Prove the loop manually

Goal: run the OBSERVE→...→ADAPT cycle by hand (this session or the next)
against the real starting state ($100, 0 followers, Day 0) and confirm it
produces a sensible day plan, a journal entry, and — if relevant — a
correctly-flagged approval request, *before* it's put on an unattended
timer.

| Task | Automatable | Status |
|---|---|---|
| Run `daily-loop` manually once, review output for voice/correctness | Yes | ⬜ |
| Fix any rough edges in the seeded skills based on that run | Yes | ⬜ |
| Create the orchestrator session (persistent) | Yes | ⬜ |
| Create the daily Routine (`create_trigger`, cron) waking the orchestrator | Yes | ⬜ |
| Human confirms they're getting/checking notifications from `PushNotification` and GitHub issues | No — human must confirm they see them | ⏳ |

Exit condition: the loop runs unattended for a few cycles and produces
journal entries + state updates a human would actually want to read.

---

## Phase 2 — Real content packages (text-only)

Goal: Claude starts producing genuine content plans and drafts — ideas,
scripts, hooks, captions — grounded in `experiments.md`, without yet being
able to generate the actual image/video/audio assets or post anywhere.

| Task | Automatable | Status |
|---|---|---|
| `content-ideation` and `content-packaging` skills (script + shot list + caption + platform spec, no asset generation) | Yes | ⬜ |
| Human reviews first few packages for voice/quality fit | No | ⏳ |
| Decide & connect a media-generation tool (Higgsfield is already visible as a connector but not authorized; evaluate it, or an alternative, against actual output quality and cost) | No — requires the human to authorize/connect the tool and confirm budget for any paid tier | ⏳ |
| Once connected: `generate-asset` skill wrapping that tool, gated through `update-ledger`/`request-approval` if it costs money | Yes, once the tool exists | ⬜ |

---

## Phase 3 — First real post (human-executed)

Goal: close the loop once, manually, to validate the whole pipeline end to
end: idea → package → (human) upload → (human) reports metrics → Claude
records and analyzes.

| Task | Automatable | Status |
|---|---|---|
| Produce first complete content package | Yes (once Phase 2 tools exist) | ⬜ |
| Human uploads to at least one platform | No — this is the platform-API gap described in `ARCHITECTURE.md` §8 | ⏳ |
| Human reports back basic metrics (views/likes/follows) | No | ⏳ |
| Claude records metrics, runs first real analysis, updates `state.md` and journal | Yes | ⬜ |
| First milestone entry: "first post" (Const. §77) | Yes | ⬜ |

---

## Phase 4 — Reduce the human-posting bottleneck

Goal: shrink the "human manually uploads everything" step, which is the
single biggest constraint on true 24/7 autonomy (see `ARCHITECTURE.md` §8).

Options to evaluate, roughly cheapest/fastest first:

| Option | Automatable once set up | Human setup required |
|---|---|---|
| Third-party scheduler with an API (e.g., Buffer, Later, Ayrshare) | Yes — one API key to wrap in a skill | Yes — account creation, possibly paid tier, connecting each platform account through their OAuth flow |
| Direct platform APIs (Meta Graph API for Instagram, TikTok Content Posting API, YouTube Data API) | Yes, per platform, once authorized | Yes, and heavier — developer account, app review, business verification; varies a lot by platform and can take weeks |
| Stay human-executed indefinitely for some platforms | N/A | This is a legitimate fallback, not a failure — Const. §6 already treats "my human" as the physical-world interface |

Decision on which path(s) to pursue is explicitly deferred to when this
phase starts — it depends on which platforms are actually working by then,
and is a human call (new accounts, possibly new cost) more than a technical
one.

---

## Phase 5 — Relationships, collaborations, and scale

Goal: start exercising Const. §59–64 (networking, collaborations) and
begin scaling whatever content format is working (Const. §69).

| Task | Automatable | Status |
|---|---|---|
| `relationships/` CRM actively used — logging real interactions only (Const. §64: never fake a relationship) | Yes | ⬜ (structure seeded in Phase 0) |
| Outreach skill, individual/low-volume only, through Gmail (already connected) | Yes, within the outreach policy in `ARCHITECTURE.md` §7 | ⬜ |
| Weekly/periodic deep-analysis skill (beyond daily loop) — Const. §73 | Yes | ⬜ |
| Scale proven content formats; retire non-performers | Yes | ⬜ |

---

## Phase 6+ — Whatever comes next

Not planned in detail now — per Const. §96 ("Phase 6 — Whatever Comes
Next"), later phases (sponsorships, products, a real team, licensing) get
designed when there's enough real traction to make that design grounded
rather than speculative. Revisit this plan at that point rather than
pre-building for it.

---

## Immediate next actions (right now, this session)

1. Finish Phase 0 scaffolding (`CLAUDE.md`, `memory/`, seed skills,
   `.gitignore`).
2. Commit and push.
3. Report to the human what was built, what decisions were made, and what
   Phase 1/2 needs from them specifically (confirming notification
   delivery; deciding on the Higgsfield connector; nothing money-related
   yet since nothing has been proposed to spend on).
