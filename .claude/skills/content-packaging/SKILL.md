---
name: content-packaging
description: Turn a content idea into a ready-to-review package in content/queue/ — copy, platform variants, and (if connected) generated assets. Use after content-ideation picks a direction, or any time a content idea is ready to be drafted for real.
---

# Content packaging

Formalizes what was done by hand for the Day 0 announcement
(`content/queue/2026-08-19-day0-announcement.md`) into a repeatable step.

## Steps

1. Write the actual copy — platform-appropriate variants (e.g., a
   char-limit-checked short version, a longer caption version). Verify
   character counts for hard platform limits with a real count, don't
   estimate.
2. If `GEMINI_API_KEY` is configured and the piece needs a visual, use the
   `generate-image` skill. If not configured, note in the package that a
   visual is wanted but not yet generated — don't skip noting the need
   just because the tool isn't there yet.
3. Save the package to `content/queue/<YYYY-MM-DD>-<slug>.md` with
   front-matter: `status: pending_approval`, `created`, `decision`
   (pointer to the relevant decision record if one exists), and, once
   platforms are live, `platform_candidates`.
4. If this package represents a new strategic direction (not just
   execution of an existing one), use `log-decision` first — packaging
   itself isn't a decision, but "what to make" sometimes is.
5. Once a package is ready, use `request-approval` before it's posted —
   packaging is not publishing.

## Guardrails

- A content package is a draft, not a claim. Nothing in `content/queue/`
  should be described elsewhere (state.md, journal) as posted until it
  actually is (moved to `content/posted/` by `publish-buffer` or by a
  human-reported manual post).
