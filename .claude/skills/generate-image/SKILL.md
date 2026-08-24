---
name: generate-image
description: Generate an image via the Gemini/Imagen API once GEMINI_API_KEY is configured. Use when a content package needs a real visual asset (thumbnail, quote card, carousel slide).
---

# Generate an image

Implements the Tier 2 media-generation path from `TOOL_STACK.md` — cheap,
pay-as-you-go image generation instead of a media-subscription tool.

## Precondition check (do this first, every time)

Check whether `GEMINI_API_KEY` is set in the environment. If it isn't,
**stop** — say plainly that image generation isn't connected yet, point at
the open approval request, and fall back to a text-only content package
instead of pretending an image was made.

**Also check billing, not just the key.** Verified 2026-08-24: a real key
being present does not mean generation works. The free tier has a **0
quota** for image-generation models specifically (confirmed via a real
`429 RESOURCE_EXHAUSTED`, `limit: 0` on `generate_content_free_tier_requests`).
If billing isn't enabled on the Google Cloud project behind this key, every
call here will 429 before it ever reaches the cost-tracking step below. See
`memory/approvals/pending/2026-08-24-enable-gemini-billing.md` — if that's
still open, don't attempt generation, report the block instead.

## API shape (corrected 2026-08-24, from a real call — see `TOOL_STACK.md`)

The old Imagen 3 `predict` endpoint referenced here previously is gone
(`404` as of 2026-08-24). Current path is a Gemini image-capable model via
`generateContent`:

```
curl "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"contents":[{"parts":[{"text":"<the prompt>"}]}]}'
```

On success, the image comes back as base64 `inlineData` inside
`candidates[0].content.parts`, not a separate `predictions` array — decode
and save it. Re-confirm the current model ID and exact request/response
shape against `https://ai.google.dev/gemini-api/docs/image-generation` the
first time this runs for real after billing is enabled — model IDs and
endpoints do change — and update this file if they've moved.

## Cost discipline

Every call costs real (tiny) money — roughly $0.02-0.04 per image
depending on model/tier. Every call, regardless of size:

1. Log it via the `update-ledger` skill immediately (category:
   `content_tools`), even though it's almost always under the $10
   single-expense approval threshold — small spends still get tracked
   honestly, per Constitution §33-34, they just don't need a full
   approval cycle.
2. If a batch of generations would cross the $10 threshold or the reserve
   floor in one go, stop and run `request-approval` before continuing.

## Steps

1. Precondition check (above).
2. Write a specific, on-brand prompt — not generic. Reflect the actual
   content package's tone and subject.
3. Call the API.
4. On success: save the image (e.g., to the `ClaudeTheRobot` Google Drive
   folder for human review, or attach to the content package), log the
   ledger entry, and reference it from the relevant file in
   `content/queue/`.
5. On failure: don't retry silently more than once or two times; report
   what happened rather than leaving it unresolved.

## Guardrails

- Never claim an image was generated without a successful API response.
- Never let cumulative small charges go untracked — "it's only 2 cents"
  is exactly the reasoning the ledger discipline exists to prevent from
  becoming "where did $30 go."
