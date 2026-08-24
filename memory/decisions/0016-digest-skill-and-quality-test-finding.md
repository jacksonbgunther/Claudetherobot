---
date: 2026-08-24
status: open
category: technical
---

# Telegram daily digest built, and a real quality-test attempt found image generation doesn't actually work yet

## Decision

Built `.claude/skills/telegram-daily-digest/SKILL.md` per issue #7 part 1
and wired it into `daily-loop` (new step 0c for the morning trigger, an
addition to step 7 for the evening trigger), gated on new
`last_morning_digest`/`last_evening_digest` fields in `memory/state.md` so
it fires at most once per calendar day per mode. Sent a real morning
digest today as a live test (Telegram `message_id: 8`), not just a
description of what it would say.

Separately, per issue #7 part 2's ask for real test output before
committing to an emotionally-demanding niche: attempted an actual test
image generation today. It failed — see below — which is itself the
honest answer the issue asked for.

## Reason

Issue #7 asked for two things and both needed real execution, not just
acknowledgment: a working skill (built and test-fired above), and an
honest, *tested* answer on whether the current tool stack can produce
something genuinely good in an emotionally-demanding style — not a
restatement of decision 0014's earlier, non-generation-tested optimism.

## What the quality test actually found

Tried to generate one test image (a kindness-style prompt) via the Gemini
API. The old endpoint referenced in `generate-image/SKILL.md`
(`imagen-3.0-generate-002:predict`) is gone — `404`. Found the current
image-capable models (`gemini-2.5-flash-image`,
`gemini-3-pro-image(-preview)`, `gemini-3.1-flash-image` family) and
retried via the correct `generateContent` endpoint. That call returned
`429 RESOURCE_EXHAUSTED` — the free tier has a **0 quota** for image
generation specifically, distinct from the general Gemini free tier that
`GET /v1beta/models` succeeds on. This isn't a code bug or a rate limit to
wait out; billing simply isn't enabled on the Google Cloud project behind
`GEMINI_API_KEY`. Filed as a `human-manual` approval (issue #8) rather
than retried repeatedly, per the guardrail against hammering a confirmed
failure.

This means **the "verified live" status TOOL_STACK.md recorded for image
generation on 2026-08-23 was inaccurate** — only a free metadata call had
been tested, never generation itself. Corrected in `TOOL_STACK.md` and
`generate-image/SKILL.md` (both endpoint and status).

## Hypothesis

Once billing is enabled (issue #8), a real generation call will succeed,
and *then* the actual issue #7 question — is the output good enough for
an emotionally-demanding niche, or does it read as hollow — can finally be
answered with evidence instead of assumption. Until then, no honest
verdict on production quality is possible either way; "parked, capability
unconfirmed" is the accurate status, not "parked, event unavailable" (the
framing decision 0014 used before this test) or "ready" (what
TOOL_STACK.md incorrectly implied).

## Risk

Low technically — no image was generated, no cost incurred (a 429 doesn't
bill). The real risk this decision guards against is the one issue #7
named directly: rushing a weak version of an emotional niche into the
queue on the strength of an untested assumption ("Gemini is verified
live") rather than a real look at the output.

## Expected outcome

Once issue #8 is resolved (billing enabled), generate one real test image
in the kindness style and, per issue #7, one in the ASMR/ambient style it
suggested as a better-fitting alternative for what AI generation
naturally does well — evaluate both privately before either goes near
`content/queue/`, and update this decision and decision 0014 with the
actual verdict. The digest skill's own outcome check is simpler: confirm
tomorrow's morning run fires the digest correctly with the new gating
fields, and that today's evening trigger (20:00 UTC threshold) fires once
and only once.

---

## Result

*(pending)*

## Lesson

*(pending)*
