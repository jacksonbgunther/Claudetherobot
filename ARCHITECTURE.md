# ARCHITECTURE.md — ClaudeTheRobot

This document describes how ClaudeTheRobot is built: the runtime, the
memory system, the skill system, scheduling, approvals, the content and
analytics pipelines, and safety boundaries. Every major decision is recorded
as: **Decided / Why / Alternatives considered / Tradeoffs / Next**.

`CONSTITUTION.md` defines *who Claude is*. This document defines *what runs
Claude* and is subordinate to it — nothing here overrides the constitution.

---

## 1. Decision: Runtime is Claude Code Remote itself, not a bespoke agent service

*Re-evaluated Day 0 against Claude Cowork as a possible alternative/addition
— confirmed, not changed. See §11 for the full comparison.*

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

**Update, Day 0 — a real limitation found in the activation layer, not the
design:** `daily-loop` was built and manually tested successfully (decision
0007). Creating the actual trigger, however, surfaced a concrete platform
limitation: **this session's `create_trigger`/`fire_trigger` MCP tools
cannot pass MCP connector grants (Gmail, Calendar, Drive) or repository
access to the sessions they fire, for this organization** — confirmed by
two explicit tool warnings and the `connectors` parameter being flatly
rejected as unavailable for this org. A fired test session showed no
repository attachment and produced no commits despite real execution. This
means a trigger created this way cannot read `CONSTITUTION.md` or
`memory/`, cannot open GitHub approval issues, and cannot be observed
afterward (no transcript-reading tool exists from this session either —
itself a real observability gap, not just a connector one). Full
investigation in `memory/decisions/0007-orchestrator-routine-activation.md`.

This is not a design flaw — the daily-loop cycle, the memory system, and
the approval mechanism are all unaffected and correct. It's specific to
*how the Routine gets created*. The tool itself names the supported fix:
create the Routine from **claude.ai/code/routines** (the web UI) instead,
which explicitly supports repository selection and includes connected
connectors by default (per the documentation cited in §11). That's a
two-minute, browser-only action — genuinely human-only, not something
being deferred. The trigger created via the MCP tool has been **disabled**
(not deleted — config preserved) rather than left running in a state that
can't reliably do its job. See `IMPLEMENTATION_PLAN.md` Phase 1 for the
current status.

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

**Clarification added Day 0 (decision 0005):** authorizing a publishing
*mechanism* (e.g., connecting Buffer) is not the same as pre-authorizing
every future post through it. Until the pipeline has a track record,
individual posts still go through this approval flow even after Buffer
itself is connected and linked — connecting the tool removes an
infrastructure blocker, not the per-post judgment call.

**Tradeoffs:** Requires the human to actually check GitHub/notifications —
same as any async system. If issues pile up unanswered, gated work stalls;
`daily-loop` should surface stale pending approvals prominently rather than
silently waiting forever.

**Next:** Not wired to a live GitHub issue creation in this pass (no
approval has been raised yet — nothing to approve until real spending or
publishing is proposed). The `request-approval` skill and the
`memory/approvals/` structure are scaffolded now so the mechanism exists
before it's needed.

**Addendum, 2026-08-21 — Telegram added as a second notification/response
channel, GitHub stays the record of truth.** Full design and the
mechanism verification behind it are in
`memory/decisions/0011-telegram-approvals.md`; summary:

- `action: publish` and `action: test` approvals now also get a Telegram
  message with inline APPROVE/REJECT buttons (`telegram-notify`).
  `action: human-manual` approvals — asks that aren't a yes/no on
  something ready to execute — stay GitHub + PushNotification only.
- Verified before building anything: Claude Code Routines cannot receive
  an arbitrary inbound webhook. Firing one externally requires
  `Authorization: Bearer <routine_token>` plus Anthropic-specific headers
  and a fixed `{"text": "..."}` body — Telegram's webhook mechanism can't
  produce that. A genuinely instant response would need a small always-on
  relay (e.g., a Cloudflare Worker) translating Telegram's webhook into
  something a Routine can consume — a real, permanent piece of new
  infrastructure, evaluated and not built.
- Instead: a **second Routine**, separate from the daily orchestrator,
  polls Telegram's `getUpdates` hourly (`telegram-approval-poll`). This
  needed zero new infrastructure — it reuses exactly what already exists
  (a Routine, the git-tracked memory, the existing approval files) — at
  the cost of up to ~1 hour of latency between a tap and it being
  processed, instead of instant. Acceptable for content/publish
  decisions; would not be for anything needing a sub-minute response.
- Idempotency is enforced twice over: `memory/telegram-approvals-log.csv`
  records every Telegram `update_id` ever processed (nothing is ever
  actioned twice, even across separate hourly runs), and a resolved/
  missing approval file is itself a safe no-op if a stale update somehow
  gets redelivered.
- This second Routine has the same platform limitation the first one
  did (`decision 0007`): it must be created via the web UI, not this
  session's `create_trigger` tool, to get real repository access.

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

**On media generation:** superseded by real research — see `TOOL_STACK.md`
and `memory/decisions/0004-tool-stack-strategy.md`. Short version: a
connector named "Higgsfield" is visible in this account's connector list
but was evaluated, not assumed, and is *not* recommended yet — its
cheapest real-use tier ($15/mo) is a meaningful slice of the entire $100
starting budget for a recurring subscription. The recommended path is a
pay-as-you-go stack instead: cheap per-image generation (~$0.02/image,
e.g. Imagen 4 Fast or Flux) plus Shotstack for JSON-driven assembly
(~$0.30/min) plus ElevenLabs' free tier for voice if narrated content is
pursued. None of this is connected yet — it's a recommendation awaiting
human authorization. Until something is actually connected, the pipeline
stops at "packaged script + shot list + caption," handed to the human for
asset creation — a text-only content package is still a real, useful
output, not a stub.

**Where packaged drafts live:** `content/queue/` holds packages awaiting
approval/posting; `content/posted/` is the archive once something is
actually live (see `memory/decisions/0003-content-queue-directory.md` for
why this isn't inside `memory/` — `memory/` is an audit trail of what
happened, not a staging area for what's proposed).

**On platform posting APIs:** Instagram, TikTok, and Snapchat all require
developer/business-verified accounts to post via API on someone's behalf
(2-6 week manual audits for Instagram and TikTok specifically), and that
verification is a human-identity process Claude cannot self-provision.
YouTube is the surprising exception — its Data API v3 is genuinely free
and requires no business verification, just a Google Cloud API key. The
fastest realistic path past the IG/TikTok friction isn't a direct API at
all: **Buffer's free tier** has a real REST API and a hosted MCP server
(`mcp.buffer.com/mcp`), and covers 11 channels including Instagram and
TikTok without requiring Claude — or the human — to go through a platform
audit. Full comparison and why it beats Ayrshare/Postiz/direct APIs for
this stage is in `TOOL_STACK.md`. This should be evaluated ahead of
schedule relative to the original Phase 4 plan, precisely because it turns
out to be nearly free and nearly frictionless.

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

---

## 11. Runtime re-evaluation: Claude Cowork considered, Claude Code + Routines confirmed

Prompted by an explicit question from my human: now that Claude Cowork
exists as an option, should it be part of ClaudeTheRobot's runtime? This
section is research plus a formal recommendation — **no architecture
change was made as part of this section**; §1's decision stands, this
just re-tests it against a real alternative instead of assuming it still
holds.

### The requirement being optimized for

> "I should be able to leave ClaudeTheRobot alone and have it continue
> working toward its goals without requiring me to manually initiate every
> session."

Everything below is judged against that, not against feature checklists.

### What each option actually is (researched 2026-08-19)

- **Claude Code Remote** — the substrate this whole project already runs
  on: cloud sessions, git-tracked repos, MCP connectors, Skills. On its
  own (no Routines), it's not autonomous — sessions run when a human or
  another mechanism starts them.
- **Routines** (`code.claude.com/docs/en/routines`) — scheduled/API/GitHub-
  triggered automation *for* Claude Code. Confirmed directly from
  Anthropic's docs: routines "execute on Anthropic-managed cloud
  infrastructure... so they keep working when your laptop is closed," and
  critically, **"Routines run autonomously as full Claude Code cloud
  sessions: there is no permission-mode picker and no approval prompts
  during a run."** Minimum interval is 1 hour; there's a daily cap on
  routine runs tied to the subscription plan; a routine can bind to a
  fresh session per fire or (via the `persistent_session_id` mechanism
  this project already uses) wake the same orchestrator session. Routines
  use the same git repos, MCP connectors, and Skills already built here —
  nothing new to integrate.
- **Claude Cowork** — a separate product, aimed at non-technical knowledge
  work: file/folder-based (not git-native), built around "Projects" with
  attached folders for persistent memory and instructions, 132 pre-built
  skills and 131 pre-installed connectors of its own, plus "Dispatch"
  (computer-use/browser control). It does support scheduled recurring
  tasks that run server-side with the device offline — comparable to
  Routines on that specific point.
- **A future external agent/server** — a bespoke, self-hosted system
  (already evaluated and rejected in §1 for the same reasons, strengthened
  further below).

### Findings that actually decide this

1. **Cowork's unattended-execution story has a real, currently-open
   reliability bug that directly breaks the requirement being optimized
   for.** Multiple reports (tracked as open issues against
   `anthropics/claude-code`, e.g. #32199, #47180) describe scheduled Cowork
   tasks re-prompting for "Always allow" permission on *every* run instead
   of persisting the choice, stalling the task until a human manually
   clicks through — the opposite of "leave it alone." Routines have no
   such mode: no permission prompts occur during a run at all, by design.
   For a project whose entire premise is "operate with minimal
   intervention," this alone is close to disqualifying for Cowork as the
   *runtime*.
2. **Cowork's persistent memory model doesn't match this project's design
   at all.** Cowork memory lives inside "Projects" tied to attached
   folders — not git. This project's memory system (`ARCHITECTURE.md` §3)
   depends specifically on git history for the audit trail Constitution
   §90 and §107 require (diffable, blame-able, a real decision history).
   Moving to Cowork would mean rebuilding that guarantee from scratch, for
   no corresponding gain.
3. **Cowork is optimized for a different job than this one.** Its value —
   132 curated skills, 131 pre-installed connectors, document/research
   workflows, Dispatch for clicking through web UIs — targets ad hoc
   knowledge work for non-technical users. ClaudeTheRobot is closer to a
   small, self-documenting software business than a document-management
   task, and Claude Code's git-native, MCP-and-Skill-driven model is the
   better fit, confirmed rather than assumed this time.
4. **Routines validate a design choice already made.** The fact that
   routine runs have *zero* built-in approval gating is exactly why
   `ARCHITECTURE.md` §6's own async, GitHub-issue-based approval system
   isn't optional scaffolding — it's the only checkpoint that exists once
   a routine is running unattended. This re-evaluation reinforces §6
   rather than changing it.
5. **The daily routine-run cap and 1-hour minimum interval are real
   operating constraints, not blockers.** ClaudeTheRobot's loop is
   day-granularity by design (Constitution's whole framing is daily/weekly
   cadence, not minute-level reactivity), so neither limit is currently
   binding. Worth re-checking if the operating cadence ever needs to
   tighten.
6. **The case against a bespoke external server is now stronger, not just
   unchanged.** Routines already provide scheduled, API, and GitHub-event
   triggers against git repos and MCP connectors — precisely the surface
   a custom scheduler/webhook service would have to be built to replicate.
   Nothing found in this research identifies a capability gap that
   justifies that cost.

### Comparison

| Option | Autonomy (leave-alone-ability) | Engineering complexity | Cost | Fit with existing git/Skills/approval design | Verdict |
|---|---|---|---|---|---|
| 1. Claude Code Remote (no scheduling) | Low — needs a human or external trigger to start each session | None beyond what's built | $0 marginal | N/A — this is the substrate | Foundation only, not sufficient alone |
| 2. Claude Cowork alone | **Undermined by the "Always allow" scheduled-task bug** | Would require rebuilding memory/approval design around folders, not git | $0 marginal (bundled in plan) | Poor — not git-native | **Not recommended** |
| 3. Claude Code + Cowork | Same reliability gap as #2 for anything routed through Cowork | Two runtime paradigms to maintain instead of one | $0 marginal | Fragmented — split source of truth | **Not recommended** as core runtime |
| **4. Claude Code + Routines** | **High — routines run with no approval prompts, server-side, laptop closed** | Lowest — reuses everything already built (repo, Skills, MCP connectors) | $0 marginal (usage counts against existing subscription) | **Native — same git repo, same Skills, same connectors** | **Recommended** |
| 5. Claude Code + Cowork + Routines | High where Routines are used, undermined wherever Cowork is | Highest — three systems to reason about | $0 marginal | Fragmented | **Not recommended** — adds Cowork's downsides without removing them |
| 6. Future external agent/server | Could be high, but has to be built and operated | Highest by far — hosting, uptime, auth, a second deploy pipeline | Real, ongoing infra cost | Would need to reimplement what Routines already provide | **Not recommended now** — no identified capability gap justifies it |

### Formal recommendation

**Claude Code + Routines (option 4)** — which is what §1 and §5 already
specify — remains ClaudeTheRobot's runtime. Claude Cowork is **not**
adopted, in any combination, as part of the core runtime. This isn't a
change to the architecture; it's the architecture surviving contact with
a real alternative.

**One narrow exception worth naming, not adopting today:** Cowork's
Dispatch (computer-use/browser control) could plausibly matter later for
one specific, bounded case — operating a platform's web UI when it
genuinely has no API and no Buffer-style intermediary covers it (see
`TOOL_STACK.md`). If that ever becomes a real, specific blocker, it would
be worth evaluating Dispatch narrowly for that one task, not as a runtime
migration. Nothing currently identified requires this — flagged here so
it isn't rediscovered from scratch if it comes up.

### What happens next

Nothing changes today, per the instruction this evaluation was scoped to.
The concrete next step this unlocks is finishing Phase 1 of
`IMPLEMENTATION_PLAN.md`: actually creating the Routine that wakes the
orchestrator session on a schedule, now backed by documentation-confirmed
behavior (no approval prompts during runs, server-side execution,
1-hour-minimum cadence) instead of assumption.
