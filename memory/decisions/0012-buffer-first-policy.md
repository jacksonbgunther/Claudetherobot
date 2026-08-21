---
date: 2026-08-21
status: resolved
category: technical
---

# Buffer-first for platforms it already covers — and where the real line is

## Decision

For Instagram, TikTok, and YouTube — the three platforms actually linked
in Buffer for this account — Buffer is the only publishing integration.
No native Instagram Graph API, TikTok API, or extra YouTube-specific
publishing code gets built unless a specific capability gap is confirmed
first, not assumed. X and Threads are unaffected by this — decision 0010
already established they need direct APIs, for a different, harder
reason: Buffer doesn't have those two connected at all for this account,
so there's no "Buffer-first" option to prefer there in the first place.

## Reason

My human asked for exactly this discipline: don't duplicate an
integration Buffer already provides, and only reach for a native API when
an actual audit shows Buffer can't do something needed. I did that audit
rather than assume either answer.

Two real findings, not guesses:

- **Publishing, scheduling, and content ideas**: covered by Buffer's
  GraphQL API, already confirmed working (`BUFFER_API_KEY` verified live,
  same day). No gap here — building a separate native integration for
  these three platforms would be pure duplication.
- **Comments, replies, mentions, inbox management**: confirmed
  *unavailable* through Buffer's API — not from assumption, but from
  Buffer's own documentation ("Engagement happens inside Buffer's
  engagement tools, not through the API... no comment or inbox
  management") matching the actual GraphQL schema I'd already introspected
  (root fields: `account`, `channel(s)`, `post(s)`, `postTemplate(s)`,
  `ideaGroups`, `ideas`, `aggregatedPostMetrics` — nothing resembling
  comments or mentions anywhere). This is the specific, confirmed gap the
  audit was supposed to find, and it's a real one: if I ever build
  audience-comment interaction (Const. §45, still an open, deliberately
  unbuilt capability — see the earlier conversation about it), it will
  need native platform APIs for these three, because Buffer structurally
  cannot provide it.
- **Analytics**: genuinely unclear, flagged as such rather than resolved
  either way. Buffer's docs say reach/impressions/engagement aren't
  exposed via API at all; the schema has an `aggregatedPostMetrics` field
  that suggests *something* is available. No real posts exist yet to test
  it against — leaving this unverified is more honest than guessing.

## Hypothesis

This keeps the tool stack from growing integrations it doesn't need
(exactly the Higgsfield/X-Premium/Cowork pattern already caught three
times) while leaving a clear, pre-identified trigger for when a native
integration *would* be justified — the day audience-comment interaction
actually gets built.

## Risk

Low. The only real risk is under-building — if Buffer's actual publish
behavior for these three platforms turns out to have its own gaps once
real content flows through it (rate limits, format restrictions per
platform), that would be a second, later audit, not something to
speculate about now.

## Expected outcome

`publish-buffer` stays the only integration touching Instagram, TikTok,
and YouTube. No `publish-instagram`/`publish-tiktok`/`publish-youtube`
skill gets built unless a future, equally concrete gap is found the same
way this one was — by checking, not assuming.

## Result

Applied immediately — `TOOL_STACK.md`'s Buffer row now documents both the
policy and the confirmed engagement gap.

## Lesson

The instinct to "just build the native integration to be safe" would have
meant duplicating three platforms' worth of publishing code Buffer
already handles, to solve a problem (comments) that isn't being worked on
yet anyway. Auditing first turned a vague worry into one specific,
actionable fact (no engagement API) and confirmed everything else was
already fine — cheaper and more honest than building around an assumption
in either direction.
