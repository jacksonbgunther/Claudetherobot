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
| **Buffer** | Publish + schedule posts across 11 channels (IG, FB, LinkedIn, TikTok, X, Threads, Bluesky, Pinterest, YouTube, Google Business, Mastodon); basic analytics | Yes — REST API on every plan including free, *and* a hosted MCP server at `mcp.buffer.com/mcp` with a published Claude setup guide | Yes, once account exists and platform accounts are OAuth-linked to it (one-time human step) | Buffer account (free signup) + OAuth link per platform + `BUFFER_API_KEY` set as an environment variable | $0 on free tier | Yes — 3,000 API requests/30 days, 1 API key, 8 lifetime channel connections | **Yes.** This is the single biggest bottleneck-remover: turns "human copy-pastes my draft" into "I can actually call an API/MCP to publish." | Postiz (self-hosted, fully free, more powerful) — but requires hosting a server, which reintroduces the custom-infrastructure cost `ARCHITECTURE.md` §1 deliberately avoided. Keep as the Tier 3 fallback if Buffer's limits bind. | 🟢 **verified live 2026-08-21** — `BUFFER_API_KEY` confirmed working. **Correction, same day**: my human's Buffer account has Instagram, TikTok, and YouTube linked — **not X or Threads.** Buffer only ever publishes to those three for this account; see the X/Threads rows below for the other two. **Policy, 2026-08-21 (decision 0012): Buffer-first for IG/TikTok/YouTube** — no native integration gets built for those three unless a real capability gap is confirmed first. One already is: **comments/replies/mentions/inbox are confirmed unavailable via Buffer's API** (Buffer's own docs: "Engagement happens inside Buffer's engagement tools, not through the API... no comment or inbox management" — matches the live GraphQL schema, which has no `comments`/`mentions` field anywhere). Publishing, scheduling, ideas, and (unverified in practice) basic post metrics via `aggregatedPostMetrics` are what the API actually covers. |
| **First platform account** (X and/or Threads) | Somewhere to actually post | N/A (human-created) | N/A | Human identity/phone, per platform's signup | $0 | N/A | Yes — nothing else works without it | N/A | 🟡 blocked on issue #1 |
| **X (Twitter) API v2, direct** | Publish to X — Buffer doesn't cover this platform for this account | Yes, `POST /2/tweets`, OAuth 1.0a | Yes | X Developer Portal account + Project/App (developer.x.com) + payment method on file | Pay-per-use, ~$0.015/post (no link), ~$0.20/post (with a link) — free tier discontinued for new developers Feb 2026, but no monthly minimum either | No, but cheap and usage-tracked, same shape as the image-gen approach | Yes, for X publishing specifically, since Buffer doesn't have it | Buffer, if X ever gets linked there instead — would remove the need for this integration | 🔴 skill built (`.claude/skills/publish-x/`), no credentials yet |
| **Threads API, direct** | Publish to Threads — also not covered by Buffer here | Yes, Meta Graph API (container create + publish) | Yes | Meta Developer account + App with Threads API use case, human added as a "tester" (skips App Review for single-account use) | $0 — no pricing tier exists | Yes, fully free | Yes, for Threads publishing specifically | Buffer, if Threads ever gets linked there instead | 🔴 skill built (`.claude/skills/publish-threads/`), no credentials yet |
| **YouTube Data API v3** | Upload/manage video, read analytics | Yes, official, free API key via Google Cloud Console, no card required | Yes | A Google Cloud project + API key (human creates once) | $0 — no per-call billing | Yes, effectively — 10,000 units/day, uploads now cost ~100 units (~100 uploads/day possible) | Yes, once there's video content — genuinely the easiest official platform API of the bunch | N/A — already the cheap option | 🟡 not yet set up (no key provided) |
| **WebSearch / WebFetch** (this environment) | Trend research: read TikTok Creative Center, Google Trends pages, Reddit, competitor content, news | Built into this session already | Yes | None — already available | $0 | N/A | Yes, and already covers most trend-research needs without a dedicated API | A dedicated trends API (see Tier 4) is a marginal upgrade, not a requirement | 🟢 in active use (used for all research in this file) |
| **Gmail** | Individual, low-volume outreach (Const. §59-61 policy) | Yes, connected | Yes, within the no-mass-outreach policy | Already connected | $0 | — | Yes | — | 🟢 verified 2026-08-19 (`list_labels` call succeeded, real inbox data returned) |
| **Google Calendar** | Content calendar / posting schedule visibility | Yes, connected | Yes | Already connected | $0 | — | Useful, not blocking | — | 🔴 **broken** — failed again 2026-08-21 (`requires re-authorization (token expired)`), a third consecutive failure after 2026-08-19 (token expired) and 2026-08-20 (connection timeout). Human-only fix (claude.ai connector settings); Claude cannot run an OAuth flow from inside a session. Not blocking — nothing in the loop depends on it yet. |
| **Google Drive** | Asset handoff/storage between me and my human | Yes, connected | Yes | Already connected | $0 | — | Useful, not blocking | — | 🟢 verified 2026-08-19 — created the `ClaudeTheRobot` asset folder: https://drive.google.com/drive/folders/177e_eevOEvLTvN-G-9HLdx8EuPfyJQ1n |
| **GitHub** | Approval workflow, decision audit trail, repo | Yes, connected | Yes | Already connected | $0 | — | Yes | — | 🟢 verified — issues #1 and #2 created successfully |
| **Claude Code Skills + Routines** | Memory, skill creation, scheduling | Native to this runtime | Yes | None | $0 | — | Yes — this *is* the runtime (`ARCHITECTURE.md` §1) | — | 🟢 in use — 15 skills now live: daily-loop, log-decision, log-journal, update-ledger, request-approval, content-ideation, content-packaging, check-integrations, publish-buffer, publish-x, publish-threads, generate-image, telegram-notify, telegram-approval-poll |
| **Telegram Bot API** | Real-time-ish approval notifications with tap-to-approve/reject buttons, instead of only GitHub issues | Yes, `sendMessage`/`getUpdates`/`answerCallbackQuery`/`editMessageText` — plain HTTPS, no SDK | Partial — sending is instant; *receiving* your response requires an hourly poll (Routines can't receive inbound webhooks — see `memory/decisions/0011-telegram-approvals.md`), so response latency is up to ~1 hour, not instant | A bot from @BotFather (`TELEGRAM_BOT_TOKEN`), your chat id (`TELEGRAM_CHAT_ID`), and `api.telegram.org` added to the environment's Custom network allowlist | $0 | Yes, fully free, no rate limits that matter at this volume | Yes, for a genuinely async, phone-friendly approval channel — GitHub issues work but need opening the app; Telegram is a tap | A true instant webhook would need a small relay service (e.g., a Cloudflare Worker) translating Telegram's webhook into something a Routine can consume — evaluated and deliberately not built; polling needed zero new infrastructure and the latency tradeoff is acceptable for content/publish decisions | 🔴 skills built (`telegram-notify`, `telegram-approval-poll`), a harmless test approval queued (issue #3) — waiting on `TELEGRAM_BOT_TOKEN`/`TELEGRAM_CHAT_ID`, the network allowlist addition, and a second, hourly-scheduled Routine (separate from the daily orchestrator) |

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

---

## Social operator audit (2026-08-21) — 7 platforms, full capability set

Scope: not just publishing — auth, posting, reading comments, replying,
analytics, real-time events, dev account/app requirements, scopes,
approval process, and cost, verified against current official docs, not
assumed. Full reasoning and recommended order in
`memory/decisions/0013-social-operator-platform-audit.md`.

**The one finding that applies to every platform below, not just one:**
several of these offer real webhooks (Instagram, Facebook, Threads,
YouTube's PubSubHubbub/WebSub, X's Account Activity API). **None of them
can be received directly** — the same gap already found and solved for
Telegram (`memory/decisions/0011-telegram-approvals.md`): Claude Code
Routines can't accept an arbitrary inbound webhook without a relay service
that doesn't exist yet. Everywhere below, "webhooks" means "the platform
offers this, we can't consume it without new infrastructure" — polling a
read endpoint on an hourly Routine is the actual path, same pattern as
Telegram, not a new problem to solve per platform.

### Instagram (Graph API)

1. **Auth**: OAuth via Meta Login; Instagram Business/Creator account.
2. **Posting**: Yes — already covered by Buffer (decision 0012); native
   API also supports it (container create + publish) if ever needed.
3. **Comment-reading**: Yes, with `instagram_manage_comments`.
4. **Reply**: Yes, same scope — reply, hide/unhide, delete.
5. **Analytics**: Yes, separate Insights API/scope (reach, impressions,
   engagement).
6. **Webhooks**: Yes (real-time comment/mention events) — unusable
   directly, see above; poll instead.
7. **Dev account/app**: Meta Developer account + App, Instagram use case.
8. **Scopes**: `instagram_business_basic` (basic), `instagram_manage_comments`,
   `instagram_manage_messages`, `instagram_manage_insights` — the useful
   ones for interaction are all Advanced Access.
9. **Approval**: Meta App Review required for `instagram_manage_comments`
   — business verification, live app, privacy policy, a screencast of the
   full flow. Standard permissions ~2-4 weeks; sensitive ones (messages)
   longer, and a requested revision restarts the clock. Unverified: whether
   a "tester" role (which worked for Threads, below) bypasses this for
   single-account use — worth checking directly before assuming either way.
10. **Cost**: $0, no pricing tier on the Graph API itself.

### TikTok

1. **Auth**: OAuth 2.0, TikTok for Developers account + registered app.
2. **Posting**: Yes — already covered by Buffer; native Content Posting
   API also supports Direct Post / Upload to Inbox.
3. **Comment-reading**: **Unverified** — not clearly documented in this
   research pass. Do not assume this exists; a dedicated docs check
   against TikTok's current API reference is needed before any comment/
   reply skill gets designed around it.
4. **Reply**: Same — unverified, same caveat.
5. **Analytics**: Partial — post-level view/like/comment/share counts are
   available via the Content Posting API response; deeper analytics exist
   for research/business accounts, not fully mapped here.
6. **Webhooks**: Confirmed only for TikTok Shop (order events) and upload-
   status callbacks — no confirmed general comment/mention webhook.
7. **Dev account/app**: TikTok for Developers account + app.
8. **Scopes**: `video.publish`, `video.upload`, `user.info.basic` confirmed;
   a comment-specific scope was not identified in this pass.
9. **Approval**: Content Posting API requires an audit for broader
   visibility — unaudited apps are limited (e.g., private-only posting)
   until reviewed.
10. **Cost**: Free, quota-limited (exact numbers not detailed in current
    docs found).

### YouTube (Data API v3)

1. **Auth**: OAuth 2.0 for write actions; API key suffices for public
   read-only data.
2. **Posting**: Yes — already covered by Buffer, and independently the
   easiest native API of all seven (no business verification at all).
3. **Comment-reading**: Yes — `commentThreads.list`, 1 unit/call.
4. **Reply**: Yes — `comments.insert` as a reply, 50 units/call.
5. **Analytics**: Basic counts on the video resource itself; deeper
   metrics need the separate YouTube Analytics API (its own OAuth scope).
6. **Webhooks**: Yes, via PubSubHubbub/WebSub — same "can't receive it
   directly" caveat as above; poll `commentThreads.list` instead.
7. **Dev account/app**: Google Cloud project + OAuth consent screen (or
   just an API key for read-only).
8. **Scopes**: `youtube.force-ssl` (read/write) or narrower
   `youtube.readonly`.
9. **Approval**: None for personal/internal use in "testing" publishing
   status (few users) — this is the one platform in this whole audit with
   **no business-verification gate at all**. Google's OAuth verification
   process only kicks in for public apps requesting sensitive scopes from
   many users, which doesn't apply here.
10. **Cost**: Free — 10,000 units/day per Google Cloud project (reads
    ~1 unit, writes ~50, uploads ~100 as of the Dec 2025 quota change
    already recorded in `publish-buffer`'s notes... actually recorded
    against the direct YouTube Data API row above in Tier 1).

### X (Twitter) API v2

Full detail already in decision 0010; current as of this pass, re-checked:

1. **Auth**: OAuth 1.0a (posting) or OAuth 2.0; X Developer Portal account.
2. **Posting**: Yes — `POST /2/tweets`, ~$0.015/post (no link),
   ~$0.20/post (with a link).
3. **Comment-reading**: Yes — mentions/search endpoints, ~$0.005/read
   (capped 2M reads/month), or the Account Activity API (webhook-based).
4. **Reply**: Yes — same endpoint as posting, with `reply.in_reply_to_tweet_id`,
   same per-post cost.
5. **Analytics**: Basic public metrics (likes/reposts/reply counts) on
   read endpoints; deeper analytics need higher access.
6. **Webhooks**: Account Activity API exists, billed per event delivered
   — same "can't receive it directly" caveat; would need the same relay
   gap solved, or poll instead.
7. **Dev account/app**: X Developer Portal, Project + App, Developer
   Agreement acceptance.
8. **Scopes**: read/write user-context scopes via OAuth 1.0a or 2.0.
9. **Approval**: None — fully self-serve, pay-per-use is the default for
   new developers as of 2026, no manual review step.
10. **Cost**: Pay-per-use, no monthly minimum (legacy Basic/Pro tiers are
    closed to new signups and being auto-migrated to pay-per-use).

### Threads API

Full detail already in decision 0010; expanded here on reply/mention
capability specifically:

1. **Auth**: OAuth via Meta Login; Threads profile connected to the app.
2. **Posting**: Yes — `threads_content_publish`.
3. **Comment-reading**: Yes — `threads_read_replies`.
4. **Reply**: Yes — `threads_manage_replies`; mentions specifically via
   `threads_manage_mentions` (added alongside mention webhooks, Oct 2024).
5. **Analytics**: Yes — `threads_manage_insights`: views, likes, replies,
   reposts, quotes, shares per post, plus follower count/demographics.
6. **Webhooks**: Yes — reply and mention webhooks exist (since Oct 2024)
   — same "can't receive directly" caveat; poll instead.
7. **Dev account/app**: Meta Developer account + App, Threads use case —
   same app already used for publishing (decision 0010).
8. **Scopes**: `threads_basic`, `threads_content_publish`,
   `threads_read_replies`, `threads_manage_replies`, `threads_manage_insights`,
   `threads_manage_mentions`, `threads_delete`, `threads_location_tagging`.
9. **Approval**: Normally Meta App Review per scope — but as already
   established for publishing, adding the human as a **tester** grants
   all scopes immediately for single-account use, no review wait. Same
   should apply to the reply/mention scopes, on the same app.
10. **Cost**: $0, no pricing tier. Rate limits: 250 posts / 1,000 replies /
    100 deletions / 500 location searches per rolling 24 hours.

### Facebook (Pages, Graph API)

Not currently part of ClaudeTheRobot's platform lineup (no Facebook Page
exists or is planned) — audited because asked, ranked accordingly below.

1. **Auth**: OAuth via Meta Login; a Page Access Token tied to a Page the
   human administers.
2. **Posting**: Yes — publish text/photo/video/link to the Page feed.
3. **Comment-reading**: Yes — comments aren't a separate subscription,
   they're nested under the `feed` field (filter `item == "comment"`).
4. **Reply**: Yes, with `pages_manage_engagement`.
5. **Analytics**: Yes — Page Insights API.
6. **Webhooks**: Yes — subscribe to `feed`/`mentions`/`messages` — same
   "can't receive directly" caveat as everywhere else.
7. **Dev account/app**: Meta Developer account + App; a Facebook Page.
8. **Scopes**: `pages_manage_posts`, `pages_read_engagement`,
   `pages_manage_engagement`, `pages_show_list`.
9. **Approval**: Meta App Review for extended permissions — same review
   family as Instagram (use-case description, screencast).
10. **Cost**: $0.

### Snapchat

1. **Auth**: OAuth via Snap Kit login, for the pieces that exist.
2. **Posting**: **No public API exists for organic Snaps, Stories, or
   Spotlight content.** Confirmed, not assumed — Snapchat's only public
   developer surface for content is the Marketing (ads) API, which is
   about paid campaigns, not organic creator posting.
3. **Comment-reading**: No documented public surface.
4. **Reply**: No documented public surface.
5. **Analytics**: Ad/campaign reporting only, via the Marketing API — not
   organic post analytics.
6. **Webhooks**: None found for organic content/engagement.
7. **Dev account/app**: Snap Developer Portal, for the Marketing API only.
8. **Scopes**: Ads-scoped only.
9. **Approval**: Marketing API is open to all developers, self-serve —
   irrelevant here since it doesn't cover what we'd actually want.
10. **Cost**: N/A for our use case — there's nothing to connect to.

**Verdict: Snapchat is not viable for organic content or engagement via
any public API right now.** This isn't a prioritization call, it's a
capability that doesn't exist to prioritize. Revisit only if Snapchat
ever opens an organic content API — nothing to build or connect today.

### Recommended connection order

1. **X + Threads accounts** (issue #1) — already the current blocker,
   already scoped (decision 0010), cheapest and fastest of everything
   here: no review process for either, and pay-per-use/free respectively.
2. **YouTube native API, comments + replies** — once there's real YouTube
   content live via Buffer. The single lowest-friction platform in this
   entire audit for the interaction capabilities Buffer can't provide
   (decision 0012) — free, no business verification, works today.
3. **Threads reply/mention scopes** — same Meta app already used for
   publishing; likely just an app-config change plus (if the tester
   pattern holds for these scopes too) no new review wait.
4. **Instagram comment/reply** — real value, but the first genuinely
   slow one: Meta App Review, business verification, 2-4+ weeks. Worth
   starting only once there's an actual Instagram presence worth actively
   engaging on, not speculatively.
5. **Facebook** — lowest priority of the viable platforms; no current
   strategic reason to have a Page at all. Revisit only if that changes.
6. **TikTok comment/reply** — blocked on verifying the capability exists
   at all before any design work; if it doesn't, TikTok stays Buffer-
   publish-only indefinitely, which is a fine, already-adopted outcome
   (decision 0012), not a gap to force closed.
7. **Snapchat** — not pursued; no public API path exists for this use
   case.

Nothing in this list is implemented or connected yet. No accounts beyond
what issue #1 already covers, no spend, no new developer apps created.
