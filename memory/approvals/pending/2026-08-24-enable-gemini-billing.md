---
action: human-manual
platform:
content_package:
telegram_message_id:
github_issue: https://github.com/jacksonbgunther/Claudetherobot/issues/8
---

## What

Enable billing on the Google Cloud project behind `GEMINI_API_KEY` (Google
AI Studio → the project's billing settings), so Gemini image generation
actually works.

## Why

Testing issue #7's ask ("real test output, not just theoretical
capability") before committing to the kindness/emotional content
direction: tried a real `generateContent` call against
`gemini-2.5-flash-image` today and got `429 RESOURCE_EXHAUSTED` —
`generate_content_free_tier_requests` quota is **0** for image-generation
models specifically. The key authenticates fine (metadata calls like
`GET /v1beta/models` work), but the free tier doesn't allow any actual
image generation at all — this isn't a rate limit to wait out, it's a
missing billing link. `TOOL_STACK.md`'s prior "verified live" status for
this was wrong; only a free metadata call had been tested before today,
not generation itself. Corrected in `TOOL_STACK.md` and
`generate-image/SKILL.md`.

## Cost

$0 to enable billing itself. Once enabled, real usage is pay-per-image,
~$0.02-0.04 each — same cost discipline as before (logged via
`update-ledger`, well under the $10 single-expense approval threshold
per-image). This request is only about unlocking the capability, not a
spend to approve.

## Expected upside

Unblocks real quality-testing of the kindness/emotional content direction
(decision 0014) and any other visual content, per issue #7's explicit
request for tested output before committing to an emotionally-demanding
niche. Currently I can't even generate one test image to evaluate.

## Potential downside

Once billing is on, a bug or runaway loop could rack up small charges
faster than intended — mitigated by the existing per-call ledger logging
and the $10/reserve-floor approval gate in `generate-image/SKILL.md`,
neither of which changes.

## What happens if we do nothing

Image generation stays completely blocked — not just "not yet tested,"
actually non-functional. The kindness/emotional niche (decision 0014) and
issue #7's requested quality assessment both stay stuck exactly where they
are now: parked on a real capability gap, this time confirmed rather than
assumed.
