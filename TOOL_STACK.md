# TOOL_STACK.md — ClaudeTheRobot's tool stack

Living reference, not a one-time decision. Updated as tools get evaluated,
authorized, connected, or dropped. See
`memory/decisions/0004-tool-stack-strategy.md` for the reasoning behind
the tiering; this file is the concrete table.

Status legend: 🟢 connected & authorized · 🟡 free, recommended, awaiting
human setup · 🔴 evaluated, not recommended yet (cost/access) · ⚪
evaluated, rejected for now (better alternative exists)

Last updated: 2026-08-19 (Day 0, third loop of the day — autonomous
integration build).

---

## TIER 1 — Required immediately (all $0 to start)

| Tool | What it allows | MCP/API | Autonomous once set up? | Credentials needed | Cost | Free tier? | Necessary? | Cheaper/better alt? | Status |
|---|---|---|---|---|---|---|---|---|---|
| **Buffer** | Publish + schedule posts across 11 channels (IG, FB, LinkedIn, TikTok, X, Threads, Bluesky, Pinterest, YouTube, Google Business, Mastodon); basic analytics | Yes — REST API on every plan including free, *and* a hosted MCP server at `mcp.buffer.com/mcp` with a published Claude setup guide | Yes, once account exists and platform accounts are OAuth-linked to it (one-time human step) | Buffer account (free signup) + OAuth link per platform + `BUFFER_API_KEY` set as an environment variable | $0 on free tier | Yes — 3,000 API requests/30 days, 1 API key, 8 lifetime channel connections | **Yes.** This is the single biggest bottleneck-remover: turns "human copy-pastes my draft" into "I can actually call an API/MCP to publish." | Postiz (self-hosted, fully free, more powerful) — but requires hosting a server, which reintroduces the custom-infrastructure cost `ARCHITECTURE.md` §1 deliberately avoided. Keep as the Tier 3 fallback if Buffer's limits bind. | 🟡 skill built (`.claude/skills/publish-buffer/`) — `BUFFER_API_KEY` and network allowlist (`api.buffer.com`) set by my human 2026-08-20 (Configure cloud environments). Not yet verified live — env changes only apply to new sessions, so first real test is the next Routine fire or any fresh session. Still inert for actual publishing either way until issue #1 gives it a platform account to link. |
| **First platform account** (X and/or Threads) | Somewhere to actually post | N/A (human-created) | N/A | Human identity/phone, per platform's signup | $0 | N/A | Yes — nothing else works without it | N/A | 🟡 blocked on issue #1 |
| **YouTube Data API v3** | Upload/manage video, read analytics | Yes, official, free API key via Google Cloud Console, no card required | Yes | A Google Cloud project + API key (human creates once) | $0 — no per-call billing | Yes, effectively — 10,000 units/day, uploads now cost ~100 units (~100 uploads/day possible) | Yes, once there's video content — genuinely the easiest official platform API of the bunch | N/A — already the cheap option | 🟡 not yet set up (no key provided) |
| **WebSearch / WebFetch** (this environment) | Trend research: read TikTok Creative Center, Google Trends pages, Reddit, competitor content, news | Built into this session already | Yes | None — already available | $0 | N/A | Yes, and already covers most trend-research needs without a dedicated API | A dedicated trends API (see Tier 4) is a marginal upgrade, not a requirement | 🟢 in active use (used for all research in this file) |
| **Gmail** | Individual, low-volume outreach (Const. §59-61 policy) | Yes, connected | Yes, within the no-mass-outreach policy | Already connected | $0 | — | Yes | — | 🟢 verified 2026-08-19 (`list_labels` call succeeded, real inbox data returned) |
| **Google Calendar** | Content calendar / posting schedule visibility | Yes, connected | Yes | Already connected | $0 | — | Useful, not blocking | — | 🔴 **broken** — `list_calendars` failed 2026-08-19 (OAuth token expired) and again 2026-08-20 from a scheduled run, now with a different signature: upstream connection timeout. Two consecutive failures, two error modes; treating it as reliably down, not flaky. Human-only fix (claude.ai connector settings); Claude cannot run an OAuth flow from inside a session. Not blocking — nothing in the loop depends on it yet. |
| **Google Drive** | Asset handoff/storage between me and my human | Yes, connected | Yes | Already connected | $0 | — | Useful, not blocking | — | 🟢 verified 2026-08-19 — created the `ClaudeTheRobot` asset folder: https://drive.google.com/drive/folders/177e_eevOEvLTvN-G-9HLdx8EuPfyJQ1n |
| **GitHub** | Approval workflow, decision audit trail, repo | Yes, connected | Yes | Already connected | $0 | — | Yes | — | 🟢 verified — issues #1 and #2 created successfully |
| **Claude Code Skills + Routines** | Memory, skill creation, scheduling | Native to this runtime | Yes | None | $0 | — | Yes — this *is* the runtime (`ARCHITECTURE.md` §1) | — | 🟢 in use — 10 skills now live: daily-loop, log-decision, log-journal, update-ledger, request-approval, content-ideation, content-packaging, check-integrations, publish-buffer, generate-image |

## TIER 2 — Required for autonomous content creation (cheap, usage-based, fits a $100 total budget)

Deliberately built around **pay-as-you-go APIs, not monthly subscriptions.**
A $15-39/mo subscription can burn a meaningful slice of total starting
capital before a single dollar of revenue exists; a per-unit API lets
spend track actual output and stop anytime.

| Tool | What it allows | MCP/API | Autonomous? | Credentials | Cost | Free tier? | Necessary? | Cheaper alt? | Status |
|---|---|---|---|---|---|---|---|---|---|
| **Image generation** (Google Imagen 4 Fast via Gemini API, or a Flux provider) | Thumbnails, quote cards, carousel visuals — the cheapest real path to visual content | Yes, standard REST API | Yes | A Gemini/Google AI Studio API key (free to create; billing is pay-per-image) | ~$0.02/image (Imagen 4 Fast) | Free tier exists for the Gemini API generally; image generation itself bills per image | Yes, for any visual content, and it's the leanest option available | Flux 2 Pro (~$0.02/unit) is comparable; Higgsfield is *not* cheaper — see below | 🟡 network allowlist (`generativelanguage.googleapis.com`) pre-added 2026-08-20 while my human was in the environment settings anyway — still no `GEMINI_API_KEY` |
| **Shotstack** | JSON-driven video assembly — captions + images + audio into a short video, without needing full generative video | Yes, REST API, sandbox for testing | Yes | Free signup, no card required for sandbox | $0.30/min pay-as-you-go (or $39/mo plan at $0.20/min — skip the plan, PAYG fits better at this scale) | 10 free credits/30 days | Yes, if pursuing narrated/edited short-form video — this plus cheap images plus ElevenLabs covers a real video pipeline for a few dollars, not a few hundred | Full generative video (Higgsfield/Runway) — see why that's Tier 3, not this tier | 🔴 no account yet |
| **ElevenLabs (free tier)** | Voiceover for narrated content | Yes, REST API | Yes | Free signup | $0 to start | Yes — 10,000 credits/mo ≈ 10 min TTS | Only if narrated video is the chosen format — not necessary for text/image-first content | — | 🔴 not needed yet |

**Explicitly not recommended for Tier 2, despite being discussed earlier:**

- **Higgsfield** — $15/mo (Starter, 200 credits) up to $129/mo (Ultra); no
  clean direct API (third-party wrappers add another $29/mo on top). A
  single month at even the cheapest paid tier is 15% of the *entire*
  starting budget, for a subscription that keeps billing whether or not
  it's used. Doesn't fail on capability — it's a real product — it fails
  the budget-fit and reversibility tests `ARCHITECTURE.md` §7 and Const.
  §34-36 care about. Revisit once there's revenue or a specific proven
  format that needs cinematic quality Shotstack-assembly can't match.
- **Runway** — API access is gated to Enterprise contracts; not available
  to an indie project at any price right now. Not evaluated further.

## TIER 3 — Required for scaling (revisit once there's revenue or clear signal)

| Tool | What it unlocks | Why it waits |
|---|---|---|
| **Higgsfield or comparable generative video** | Cinematic-quality video, if a format proves it's worth it | Only worth the subscription once Tier 2's cheap image+voice+Shotstack pipeline has been tested and something specifically needs full generative video |
| **Postiz (self-hosted)** | Same publishing capability as Buffer, no request/channel caps, fully free | Requires hosting a small always-on service — a real infrastructure decision, not a signup. Only worth it once Buffer's free-tier limits (3,000 req/30 days, 8 channels) actually bind |
| **TikTok Content Posting API (direct)** | Native TikTok publishing without going through Buffer | Manual app audit, 2-6 weeks, and Buffer already covers TikTok in the meantime — pursue direct access only if Buffer's TikTok support proves insufficient |
| **Instagram Graph API "Advanced Access" (direct)** | Native IG publishing beyond Buffer | Requires Meta App Review + Business Verification, 2-4 weeks — same logic as TikTok: Buffer covers this today |
| **ElevenLabs paid tier / commercial license** | Real monetized use of narrated audio | Free tier has no commercial license — only matters once actually monetizing narrated content |

## TIER 4 — Nice-to-have (low priority, marginal benefit over what's free)

| Tool | Why it's here, not higher |
|---|---|
| **LunarCrush** (found in the MCP registry, not installed) | Social listening/trend data, but leans crypto/finance in its core product; uncertain fit for this content. Revisit only if WebSearch-based trend research proves insufficient. |
| **Dedicated trends API** (e.g., a 30+-source aggregator, ~100 free requests/mo) | WebSearch/WebFetch already cover most of this need at $0; a dedicated API is a marginal convenience upgrade, not a new capability. |
| **Snapchat Marketing API** | Actually the most *open* of the paid-social APIs (no business verification gate, available since 2018) — genuinely easy if Snapchat becomes a target platform. Lower priority only because Snapchat isn't the current platform focus (Decision 0002), not because it's hard to get. |
| **Motion Creative Analytics** (found in the MCP registry) | Competitor/ad creative intelligence — relevant once running paid distribution, not before there's any revenue. |
| **Phyllo** (creator analytics aggregator) | Evaluated and rejected for now — no free tier, custom enterprise pricing (reportedly $20k/yr at scale). Native platform analytics + Buffer's built-in analytics cover current needs at $0. |

---

## What this changes about the plan

- Publishing no longer has to mean "human manually uploads forever." Buffer
  (free) plus a linked platform account is a real path to me actually
  calling a publish API — this pulls part of `IMPLEMENTATION_PLAN.md`
  Phase 4 forward, once my human authorizes and links it.
- Media generation doesn't have to mean Higgsfield. A cheap image-gen API
  plus Shotstack (and ElevenLabs if narrated) covers real content
  production for single-digit dollars instead of a monthly subscription
  that outpaces the entire starting budget.
- Nothing above is connected or authorized yet. This is the recommendation
  from research — see the approval request for what's actually being asked
  of my human right now.
