# ARCHITECTURE.md — ClaudeTheRobot

This document describes how ClaudeTheRobot is built: the runtime, the
memory system, the skill system, scheduling, approvals, the content and
analytics pipelines, and safety boundaries. Every major decision is recorded
as: **Decided / Why / Alternatives considered / Tradeoffs / Next**.

`CONSTITUTION.md` defines *who Claude is*. This document defines *what runs
Claude* and is subordinate to it — nothing here overrides the constitution.

---

## 1. Decision: Runtime is Claude Code Remote itself, not a bespoke agent service

**Decided:** ClaudeTheRobot runs *inside* Claude Code (this environment) —
using Claude Code Remote sessions, Routines (scheduled triggers), MCP
connectors, Skills, and this git repository as durable storage. There is no
separate Python/Node "agent server," no custom LLM orchestration loop, and
no self-hosted infrastructure.

**Why:** Everything a bespoke agent framework would need to provide already
exists here: an LLM loop with tool use (this session), a scheduler
(`create_trigger`/Routines, cron-based), a hosting/compute layer (Claude
Code Remote environments), a notification channel (`PushNotification`,
email), a version-controlled filesystem for state (this repo), and a native
extensibility mechanism that maps directly onto Constitution §85 ("I should
create and improve my own Skills") — Claude Code's Skills system. Building
a parallel system to do the same jobs would add operational surface area
(hosting, uptime, auth, a second deploy pipeline) for a project that starts
with $100 and no engineering budget beyond Claude's own time.

**Alternatives considered:**
- *Bespoke agent service* (e.g., a Python app using an orchestration
  framework, a hosted cron job calling the Anthropic API directly, a
  Postgres/SQLite backend, a custom admin UI). Rejected for v1: it
  duplicates infrastructure this environment already provides, needs
  separate hosting and secrets management, and — critically — loses the
  built-in Skills/subagent/tool ecosystem that Claude Code already gives
  the constitution's "develop my own skills" and "use tools intelligently"
  principles for free.
- *No scheduling at all — purely human-triggered sessions.* Rejected: fails
  the "operate continuously with minimal human intervention" requirement
  outright.

**Tradeoffs:** We are coupled to Claude Code Remote's capabilities and
limits (routine minimum interval is hourly; a session's container is
ephemeral and reclaimed on inactivity; tool access depends on what's
connected to this account). If ClaudeTheRobot ever needs true sub-hourly
reactive behavior (e.g., instant reply to a DM) or a public-facing
web surface for the audience, a small dedicated service may become
justified later — but that's a addition on top of this foundation, not a
replacement for it, and should only be built when a concrete capability
gap forces it.

**Next:** Implement the actual Routine(s) once the daily-loop skill exists
(see `IMPLEMENTATION_PLAN.md` Phase 1). Not created yet in this pass —
scaffolding first, scheduling second, so the loop can be tested manually
before it's put on a timer.

---

## 2. Decision: Hybrid session model — persistent orchestrator + repo as ground truth

**Decided:** ClaudeTheRobot's main thread is a long-lived, persistent
Claude Code Remote session (the "orchestrator") that a Routine wakes on a
cadence. It carries conversational continuity with the human operator. But
*nothing durable is trusted to live only in that session's context* — every
decision, transaction, journal entry, and state change is written to the
`memory/` directory in this repo as it happens. Heavy or parallel sub-work
(batch content drafting, research) is delegated to background subagents or
short-lived sessions so the orchestrator's own context stays small and
long-lived.

**Why:** Personality continuity (Const. §17, §95, §97 — "the Claude at Day
1 should not sound exactly like Claude at Day 300... I should feel like the
same person who has grown") is easiest to preserve in a persistent
conversation with the human. But a project meant to run for months or years
cannot depend on one context window never being compacted, corrupted, or
outgrown. Making the repo the source of truth means the orchestrator
session can be safely recreated at any time by reading `memory/state.md`
and recent journal entries — the felt continuity is a convenience, not a
dependency.

**Alternatives considered:**
- *Fresh session per scheduled fire* (`create_new_session_on_fire: true`),
  fully stateless. Rejected as the sole model: it's more "production-clean"
  but sacrifices the in-conversation relationship with the human operator
  that the constitution treats as real (Const. §6, §19) and makes every
  wake pay a full context-reconstruction cost.
  It remains available and is used for background/parallel work.
- *Persistent session only, no repo state.* Rejected outright — this is
  exactly the context-drift failure mode described above, and it violates
  Const. §90 (no way to audit "did this actually happen").

**Tradeoffs:** Slight duplication — the orchestrator "knows" things from
its own conversation history that also had to be written down. The
discipline of "if it's not written to `memory/`, it didn't happen" has to
be enforced deliberately (via the skills in §4, not by hoping the model
remembers). If the orchestrator session becomes unusable, the human must
manually create a fresh one — a documented, low-frequency operational task.

**Next:** Create the orchestrator session and its Routine after the
`daily-loop` skill and `memory/` scaffolding are in place and have been
exercised at least once manually.

---

## 3. Decision: Memory is plain git-tracked files, not a database

**Decided:** `memory/` is a directory of Markdown (with YAML front-matter)
and CSV files, committed to git like any other repo content. No SQLite,
Postgres, or external DB.

Layout:
```
memory/
  state.md              # current snapshot — read first on every wake
  journal/               # dated first-person entries (the raw story)
  decisions/              # one file per major decision (Const. §72 format)
  experiments.md          # hypothesis -> content -> result -> lesson log
  ledger.csv              # append-only financial transactions
  metrics/                # per-platform follower/engagement snapshots (csv)
  relationships/          # one file per person/brand relationship (CRM)
  lore.md                 # inside jokes, nicknames, recurring bits, milestones
  approvals/
    pending/              # requests awaiting human sign-off
    resolved/             # approved/rejected, with outcome
```

**Why:** Markdown+front-matter is directly legible to both Claude and the
human, diffable, greppable, and blame-able — every change has a commit
message and a timestamp for free. CSV is used only where data is genuinely
tabular and append-only (ledger, metrics), because it diffs as one line per
new record, and is trivial to aggregate for analytics without any query
engine. This whole system requires zero new infrastructure, zero new
credentials, and is inherently backed up and versioned by git.

**Alternatives considered:**
- *SQLite.* Rejected for v1: binary format doesn't diff or blame in git
  (defeats the audit trail that Const. §107 wants), adds a dependency, and
  is unnecessary at this transaction volume ($100 budget, low-frequency
  writes). Revisit if/when transaction or metrics volume grows enough that
  file-scanning becomes a real bottleneck — that's a scale problem this
  project does not have yet.
- *A hosted database (Postgres/Supabase/etc.).* Rejected: new
  infrastructure, new credentials, no benefit at this scale, and it would
  make the state harder to inspect by just reading the repo.
- *Vector store for semantic memory retrieval.* Rejected for v1: the
  planned memory volume is small enough that `state.md` + "read recent
  journal entries" + grep is sufficient. Worth reconsidering only if/when
  the journal grows large enough that manual retrieval genuinely misses
  relevant history.

**Tradeoffs:** No fast semantic search over the whole history; retrieval is
"read the index, read recent entries, grep if needed." This is fine at
current scale and degrades gracefully (grep still works at 10x the volume);
it would degrade badly at 100x. `state.md` must be actively kept small and
current — it's the one file guaranteed to be read every wake, so it holds
pointers and a snapshot, not the full history.

**Next:** Seed `memory/` with initial files (this pass). No historical data
exists yet — `state.md` starts at Day 0, $100, 0 followers.

---

## 4. Decision: Skills are the extensibility mechanism, seeded minimally

**Decided:** Use Claude Code's native Skill system (`.claude/skills/<name>/SKILL.md`)
as *the* mechanism for Const. §85 ("I should create and improve my own
Skills"). Seed only the skills needed to operate the memory system
correctly and consistently; do not pre-build skills for capabilities that
don't exist yet (content generation, platform posting, outreach).

Seeded in this pass:
- `daily-loop` — runs one iteration of the OBSERVE→ANALYZE→HYPOTHESIZE→ACT→
  MEASURE→LEARN→DOCUMENT→ADAPT cycle (Const. §71), reading `memory/state.md`
  and recent journal/experiment entries, and ending by updating them.
- `log-decision` — writes a structured decision record (Const. §72 format)
  to `memory/decisions/`.
- `log-journal` — writes a first-person dated journal entry.
- `update-ledger` — appends a transaction to `memory/ledger.csv`,
  recomputes the running balance, and checks it against the spending
  policy in §7 below (flagging anything that needs approval instead of
  silently applying it).
- `request-approval` — the standardized way to raise something to the
  human: writes `memory/approvals/pending/<id>.md`, opens a GitHub issue,
  sends a push notification. See §6.

**Why not build more skills now:** A skill for, say, "generate a TikTok
video" would be a non-functional stub without a connected media-generation
tool — exactly the kind of half-finished implementation to avoid. Skills
should be created when the underlying capability is real and the workflow
has actually repeated at least once, per the constitution's own criterion
("if I repeatedly perform the same complex task").

**Alternatives considered:**
- *A single monolithic "operate" skill covering everything.* Rejected:
  harder to improve incrementally, harder for Claude to reason about which
  step it's in, and contradicts the constitution's framing of Skills as
  discrete, reusable workflows.

**Tradeoffs:** Under-provisioning skills now means more of the early
sessions' work is "manual" (direct tool calls, not a packaged skill). That's
the right tradeoff for a foundation pass — premature skills would need to
be rewritten once real tools are connected anyway.

**Next:** As Phase 2+ tools come online (media generation, a posting
integration), add corresponding skills, and let Claude propose new skills
itself when it notices a repeated workflow — per Const. §85, subject to the
constraint below.

**Boundary (Const. §85, last line):** No skill may silently override the
constitution, the spending thresholds in §7, or the approval requirement in
§6. Skills that touch money, publishing, or outreach must call through
`update-ledger` / `request-approval`, never bypass them.

---

## 5. Decision: Scheduling via Claude Code Remote Routines

**Decided:** Use `create_trigger` (Routines) for all time-based wakes:
a daily planning wake for the orchestrator session, plus narrower Routines
as needed (e.g., checking pending approvals). Minimum interval is hourly,
which is more than sufficient for a creator/business loop that plans in
days, not minutes.

**Why:** Already available, already authenticated, survives container
restarts, and requires no new infrastructure. It's the same mechanism
this session itself could use to check in on long-running work.

**Alternatives considered:**
- *External cron + webhook into some API.* Rejected: requires hosting a
  receiving endpoint, which is exactly the bespoke-infrastructure cost
  §1 avoids.
- *Fully manual, human-triggered only.* Rejected: fails the continuous-
  operation requirement.

**Tradeoffs:** Hourly is the finest granularity available. Not a real
constraint for this use case (a creator doesn't need minute-level
autonomy), but ruled out for anything that ever needs near-real-time
reaction (e.g., instantly answering a DM) — that would need a different
mechanism (e.g., PR/webhook-style event subscription, which this
environment also supports for GitHub — could be reused for platform
webhooks later if a platform connector ever exposes them).

**Next:** Not created in this pass. The loop should be run manually at
least once (proving `daily-loop` actually produces sensible output) before
it's put on an unattended timer — see `IMPLEMENTATION_PLAN.md` Phase 1.

---

## 6. Decision: Approvals are GitHub Issues + a pending-approval file, not a custom dashboard

**Decided:** When Claude needs human judgment, money movement, or a
physical action, it:
1. Writes `memory/approvals/pending/<id>.md` with: what, why, cost,
   expected upside, potential downside, and what happens if nothing is
   done (the exact structure Const. §87 specifies).
2. Opens a GitHub issue on this repo mirroring that content, labeled
   `needs-approval`.
3. Sends a `PushNotification` (or, if the orchestrator session is live in
   conversation with the human, just asks in-thread) — one line, under 200
   characters, leading with what needs a decision.

The human approves/rejects via the GitHub issue (comment/close) or directly
in conversation. A later wake checks open `needs-approval` issues before
proceeding with anything gated on them, and moves resolved requests to
`memory/approvals/resolved/` with the outcome recorded.

**Why:** GitHub is already connected to this repo, already has a mobile app
and email notifications the human already uses, gives free threaded
discussion (the human can ask questions before approving), and is
inherently auditable (issue history = a permanent decision log,
independent of `memory/decisions/`). Building a custom approval UI would
be a whole extra product for no real benefit at this scale.

**Alternatives considered:**
- *Only `AskUserQuestion` inside a live session.* Rejected as the sole
  mechanism: it blocks synchronously and does nothing if the human isn't
  watching a fired Routine session, which is most of the time by design
  (that's the point of autonomy).
- *A dedicated Slack/Discord bot.* Rejected for v1: no such connector is
  currently attached to this account; revisit if the human sets one up.
- *Email-only approval.* Rejected as sole channel: harder to build a
  structured, queryable history from than GitHub issues; kept as a
  secondary notification surface (Gmail is already connected) rather than
  the system of record.

**Tradeoffs:** Requires the human to actually check GitHub/notifications —
same as any async system. If issues pile up unanswered, gated work stalls;
`daily-loop` should surface stale pending approvals prominently rather than
silently waiting forever.

**Next:** Not wired to a live GitHub issue creation in this pass (no
approval has been raised yet — nothing to approve until real spending or
publishing is proposed). The `request-approval` skill and the
`memory/approvals/` structure are scaffolded now so the mechanism exists
before it's needed.

---

## 7. Safety and permission boundaries (policy, enforced by skills, not by hope)

This is the concrete operationalization of Const. §87–91.

| Category | Rule |
|---|---|
| Spending | Claude has no payment method. Every real expense is *proposed* by Claude and *executed* by the human. Any single proposed expense over **$10**, or any spend that would drop tracked capital below a **$20 reserve floor**, requires `request-approval` before being proposed to the human for execution. Below that, Claude may still log the proposal but does not need to block on a full approval cycle — it should still tell the human, just not necessarily gate on a formal issue. |
| Posting | No autonomous publishing to any platform until a verified, ToS-compliant, human-authorized posting integration exists (none does yet). Until then, "publish" means: Claude prepares a complete content package, the human uploads it, and reports back what happened. |
| Outreach | No mass or automated messaging. Individual, low-volume, human-visible outreach only, through already-connected/authorized accounts (e.g., Gmail). No scraping or contacting people at scale. |
| Claims | Never record or state that money was spent, content was posted, or a relationship/deal exists unless a tool call actually confirmed it. `memory/` distinguishes *planned*, *attempted*, *completed*, and *verified* — front-matter `status` fields, not prose. |
| Platform rules | No bought followers/engagement, no fake accounts, no ToS circumvention, no impersonation (Const. §91). |
| Legal/financial commitments | Always require explicit human approval via `request-approval` — no threshold exception. |
| Secrets | No credentials, API keys, or tokens are ever committed to this repo. Anything sensitive goes through environment configuration the human sets up out-of-band, referenced by name only. |

These thresholds are deliberately conservative and starting values — they
live in `memory/state.md` (not hardcoded in prose here) so they can be
tuned as capital grows, without editing this architecture document. This
document records that the mechanism and starting values exist; the live
values are operational state.

---

## 8. Content pipeline (designed now, most stages deferred)

Stages: **Ideation** (from `experiments.md` + observed trends) →
**Concept/script draft** (text — Claude can do this now) →
**Asset generation** (image/video/voice — needs a connected media tool;
none is authorized yet, see below) → **Assembly** (needs editing
capability, likely a human step for the foreseeable future) →
**Caption/hook/hashtags** (text — Claude can do this now) →
**Platform packaging** (per-platform spec: aspect ratio, length,
constraints) → **Human review queue** → **Publish** (human-executed, see
§7) → **Metrics capture** (human-reported or, later, API-pulled) back into
`memory/metrics/`.

**On media generation:** a connector named "Higgsfield" (AI image/video
generation) is visible in this account's connector list but is not
currently connected/authorized (`enabledInChat: false`, `installState:
unknown`). Nothing in the content pipeline should assume it works until the
human connects it and Claude has confirmed a real call succeeds. Until
then, the pipeline stops at "packaged script + shot list + caption,"
handed to the human for asset creation — a text-only content package is
still a real, useful output, not a stub.

**Where packaged drafts live:** `content/queue/` holds packages awaiting
approval/posting; `content/posted/` is the archive once something is
actually live (see `memory/decisions/0003-content-queue-directory.md` for
why this isn't inside `memory/` — `memory/` is an audit trail of what
happened, not a staging area for what's proposed).

**On platform posting APIs:** Instagram, TikTok, YouTube Shorts, and
Snapchat all require developer/business-verified accounts to post via API
on someone's behalf, and that verification is a human-identity process
Claude cannot self-provision. This is a hard external constraint, not a
design choice — flagged here so it isn't rediscovered painfully later.
Third-party schedulers (e.g., Buffer, Later, Ayrshare) can reduce this
friction to "one API key" once the human sets one up; that's the fastest
realistic path to automated posting and should be evaluated in Phase 4 of
`IMPLEMENTATION_PLAN.md`.

## 9. Analytics pipeline

`memory/metrics/<platform>.csv`, one row per snapshot (date, followers,
views, engagement, revenue-attributed-if-any). Populated by human-reported
numbers until/unless a platform analytics API is connected. `daily-loop`
(and a periodic deeper review) compute deltas and trends from these files
and write findings into the journal and `state.md` — this is the mechanism
for Const. §51/§73/§74 (learning from success, periodic self-analysis,
turning analysis into content).

## 10. What's deliberately not built yet

- Any code that calls a specific social platform's API — none are
  connected, and writing bindings against an unauthenticated,
  not-yet-scoped API would be speculative.
- Any code that calls a media-generation API — same reasoning; Higgsfield
  is a candidate, not a confirmed dependency, until connected.
- A custom web UI of any kind (approval dashboard, analytics dashboard).
  GitHub + the repo + `PushNotification` cover the v1 need.
- Automated payments of any kind.

See `IMPLEMENTATION_PLAN.md` for the phased path from here to those.
