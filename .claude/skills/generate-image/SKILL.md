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

## Verified API shape (2026-08-19, from ai.google.dev)

REST call:

```
curl "https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"instances":[{"prompt":"<the prompt>"}],"parameters":{"sampleCount":1}}'
```

(Or, if using the `google-genai` SDK instead of raw REST:
`client.models.generate_images(model="imagen-3.0-generate-002", prompt=...,
config=types.GenerateImagesConfig(number_of_images=1))`.) Confirm the
current model ID and exact request shape against
`https://ai.google.dev/gemini-api/docs/imagen` the first time this runs
for real — model IDs and endpoints do change — and update this file if
they've moved.

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
