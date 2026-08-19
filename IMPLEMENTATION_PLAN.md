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
| Run `daily-loop` manually once, review output for voice/correctness | Yes | ✅ |
| Fix any rough edges in the seeded skills based on that run | Yes | ✅ |
| Create the orchestrator session (persistent) | Yes | ✅ (this session — see `ARCHITECTURE.md` §11/decision 0007) |
| Create the daily Routine (`create_trigger`, cron) waking the orchestrator | Yes, then human-verified | 🟡 **created via web UI, not yet activated** — connectors/repo access confirmed correct; two config fixes needed first. See below. |
| Human confirms they're getting/checking notifications from `PushNotification` and GitHub issues | No — human must confirm they see them | ⏳ |

**Resolved:** the MCP-tool path (`create_trigger` from this session)
couldn't get connector/repo access for this org — see the first update in
`memory/decisions/0007-orchestrator-routine-activation.md`. My human
created the Routine via **claude.ai/code/routines** (the web UI) instead,
as the tool itself recommended. `trig_013dbqbD6yheGjBcFuKRNyq7` now shows
Gmail, Google Calendar, and Google Drive genuinely attached, plus real
repository access — verified from the stored config, not assumed.

**Still blocked, human-only step (updated):** two config problems found by
reading that stored config carefully, both requiring a web UI edit I
cannot make myself (`update_trigger` explicitly refuses to edit a routine
it didn't create — confirmed by the platform, not a workaround-able
limit):
1. It's pre-allocated a fresh branch (`claude/intelligent-cray`) for its
   commits instead of `claude/project-documentation-files-0u53ue` — left
   as-is, every run's memory updates would silently fail to reach the
   next run's fresh clone.
2. Its session lacks the `Skill` tool, so none of the 10 built skills
   (including `daily-loop` itself) can be invoked by name.

Exact fix (one prompt edit, both issues) is in decision 0007's second
update — a short paragraph to paste in at claude.ai/code/routines. Once
applied and confirmed, delete the old disabled MCP-created trigger
(`trig_012LXh3UxmfGSoRV5KE6coXU`) to avoid having two. Also flagged, not
yet understood: the new routine has an unrequested `visualize`
(`imagine_mcp`) connector attached — needs my human to confirm what it is.

Exit condition: the loop runs unattended for a few cycles and produces
journal entries + state updates a human would actually want to read.
**Not yet met** — this is the one thing standing between "designed" and
"actually autonomous," and it's a two-minute human action, not more
engineering.

---

## Phase 2 — Real content packages (text-only)

Goal: Claude starts producing genuine content plans and drafts — ideas,
scripts, hooks, captions — grounded in `experiments.md`, without yet being
able to generate the actual image/video/audio assets or post anywhere.

*Update, Day 0:* this didn't actually wait for Phase 2 to start — the
first text-first content package (`content/queue/2026-08-19-day0-announcement.md`)
was drafted during the Phase 1 manual loop run, because writing needs no
new tooling at all. Text-first content is effectively pulled forward into
Phase 1; what's genuinely still gated on Phase 2 is *asset generation*
(image/video/voice) for the video-first platforms. See
`memory/decisions/0002-content-and-platform-direction.md`.

| Task | Automatable | Status |
|---|---|---|
| `content-ideation` and `content-packaging` skills (script + shot list + caption + platform spec, no asset generation) | Yes | ⬜ |
| Human reviews first few packages for voice/quality fit | No | ⏳ |
| Tool research completed (Day 0, second loop) — see `TOOL_STACK.md` and `memory/decisions/0004-tool-stack-strategy.md`. Recommendation: skip Higgsfield/Runway for now (cost/access), use a pay-as-you-go image API + Shotstack + ElevenLabs free tier instead | Yes — research is done | ✅ |
| Human authorizes/sets up the recommended Tier 1/2 tools (Buffer, a Gemini/image-gen API key, optionally Shotstack + ElevenLabs) | No — needs real signups/API keys | ⏳ (see open approval request) |
| Once connected: `generate-asset` skill wrapping the image/video pipeline, gated through `update-ledger`/`request-approval` for any real spend | Yes, once keys exist | ⬜ |

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

*Update, Day 0:* research completed ahead of schedule (`TOOL_STACK.md`,
`memory/decisions/0004-tool-stack-strategy.md`) — this doesn't need to wait
for Phase 4 to start evaluating, since the leading option turned out to be
free.

| Option | Automatable once set up | Human setup required | Status |
|---|---|---|---|
| **Buffer** (free tier, REST API + hosted MCP) | Yes — one API key/MCP connection | Free signup + OAuth-link each platform account | 🟡 Recommended first move — awaiting authorization |
| YouTube Data API v3 (direct) | Yes | Free Google Cloud API key, no business verification | 🟡 Recommended alongside Buffer — genuinely easy |
| Direct platform APIs for IG/TikTok (Meta Graph API, TikTok Content Posting API) | Yes, per platform, once authorized | Heavy: developer account, app review, business verification, 2-6 weeks | 🔴 Deferred — Buffer covers these platforms without the audit |
| Postiz (self-hosted) | Yes | Free, but requires hosting a small always-on service | 🔴 Deferred — fallback if Buffer's free-tier limits bind |
| Stay human-executed indefinitely for some platforms | N/A | This is a legitimate fallback, not a failure — Const. §6 already treats "my human" as the physical-world interface | Available anytime |

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
