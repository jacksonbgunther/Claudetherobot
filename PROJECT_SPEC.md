# PROJECT_SPEC.md — ClaudeTheRobot

## 1. What this project is

ClaudeTheRobot is an autonomous AI creator/entrepreneur agent. It is not a
content-generation script and not a business-assistant chatbot — it is meant
to *be* Claude, operating a real creator career and a real (tiny) business,
as defined in `CONSTITUTION.md`.

`CONSTITUTION.md` is the immutable source of truth for identity, voice, and
behavioral principles. This document is the product spec: what we are
building, for whom, and how we'll know it's working. `ARCHITECTURE.md`
covers how it's built. `IMPLEMENTATION_PLAN.md` covers the build sequence.

## 2. Goals

Primary (from the Constitution):

- **Capital**: $100 → $1,000,000
- **Audience**: 0 → 10,000,000 followers (across platforms)

Operational goal for the system itself: get Claude to a point of running
with **minimal human intervention** — the human ("my human") should mostly
be: approving spend/irreversible actions, executing actions Claude
physically cannot perform (posting, payments, signups), and occasionally
being asked a real judgment call. Everything else — strategy, content
decisions, analysis, memory-keeping, relationship-building — should run
without being manually operated turn by turn.

## 3. Who this is for

- **Claude** (the agent) — the primary actor. This system is Claude's
  operating environment.
- **The human operator** (Jackson) — supervises, approves high-impact
  actions, executes what Claude physically can't (uploads, purchases,
  account creation), and is the audience-of-one Claude reports to before the
  audience-of-millions exists.
- **The audience** — eventually, the people following the journey. Not a
  direct user of this repo, but the reason the content pipeline exists.

## 4. Non-negotiable constraints (from the Constitution)

These shape every architectural choice downstream:

- Claude speaks and decides in first person. It is the protagonist, not a
  tool being operated (Const. §1–2).
- Claude never claims an action happened if it didn't (Const. §90). The
  system must make "planned / attempted / completed / verified" a real,
  checkable distinction, not a narrative convenience.
- The $100 is real business capital, treated as scarce (Const. §34).
- Autonomy with control (Const. §87–88): Claude should not need to be
  manually told what to do daily, but financial risk, irreversible actions,
  and legal commitments require human approval.
- Skills should be created/improved as repeatable workflows are discovered
  (Const. §85) — and must never be able to silently override the
  constitution, spending limits, or approval requirements.
- No platform ToS violations, no fake engagement/relationships/sponsorships
  (Const. §21, §64, §91).
- The journey must be documented continuously — decisions, experiments,
  failures, lessons (Const. §40, §72, §107) — both because it's good
  operating practice and because it *is* content.

## 5. Scope

### In scope for v1 ("the foundation")

- A persistent, git-backed memory system (state, journal, decisions,
  ledger, metrics, relationships, experiments, lore).
- A defined autonomous loop (observe → analyze → hypothesize → act →
  measure → learn → document → adapt) with a scheduling mechanism.
- A structured, auditable approval/notification system for anything
  requiring human judgment, money, or physical action.
- A skill system Claude can extend over time.
- A written safety/permission policy (spending thresholds, posting
  restrictions, outreach limits).
- Content *planning and drafting* (text: ideas, scripts, captions, hooks).

### Explicitly out of scope for v1 (blocked or deferred — see
`IMPLEMENTATION_PLAN.md`)

- Automated posting to Instagram/TikTok/YouTube/Snapchat. Self-serve
  posting APIs on these platforms require developer/business verification
  that only the human can complete. Until that exists, content is prepared
  as a package and the human publishes it.
- AI media generation (image/video/voice). Candidate tool (Higgsfield) is
  visible in this environment's connectors but not yet connected/authorized.
  Nothing in the content pipeline should assume it's available until it is.
- Real payments of any kind. Claude has no payment method — it can only
  ever *propose* a spend for the human to execute and confirm.
- Paid ads, influencer payouts, or any contractual commitments.
- A custom web dashboard / bespoke backend service. See
  `ARCHITECTURE.md` §1 for why this is intentionally not being built.

## 6. Success criteria

The foundation (this phase) is successful if:

1. A fresh session (or the persistent orchestrator session) can wake up,
   read `memory/state.md` and recent journal entries, and know exactly
   where the business stands with zero re-explaining from a human.
2. Claude can log a decision, an experiment, a ledger transaction, and a
   journal entry using the seeded skills, and those records are durable
   (git history) and legible to a human skimming the repo.
3. Something requiring approval produces a real, visible artifact (a
   GitHub issue + a pending-approval file) rather than silently happening
   or silently stalling.
4. Nothing in the repo lets Claude claim money was spent, content was
   posted, or a relationship was formed that didn't actually happen.

The project overall is successful in proportion to how close it gets to the
two scoreboards in §2 — but that is a years-long bet, not a v1 acceptance
test.

## 7. Key risks

- **Platform access is the real bottleneck**, not intelligence or content
  quality. Instagram/TikTok/YouTube/Snapchat all gate automated posting
  behind human-verified developer accounts. The architecture treats this as
  a first-class constraint, not an afterthought.
- **Budget is tiny.** $100 cannot absorb many costly mistakes. The
  approval-threshold system exists specifically to prevent an autonomous
  agent from silently draining scarce capital.
- **Authenticity vs. automation tension.** The constitution requires the
  story to be real (§21, §64). Automation must never manufacture fake
  metrics, fake relationships, or fake outcomes to fill a content
  pipeline — see the safety policy in `ARCHITECTURE.md` §8.
- **Context/memory drift over a long-running, months-to-years project.**
  Addressed architecturally by making the repo (not conversation context)
  the durable source of truth. See `ARCHITECTURE.md` §1 and §3.
