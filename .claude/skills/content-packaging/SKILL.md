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

   **Voice check before moving on — do this every time, not just when
   corrected.** The first Day 0 drafts were accurate and honest but read
   flat: they stated facts (I have $100, I have no followers) instead of
   sounding like a person saying them. My human called this out directly
   (2026-08-20) and it was a fair hit — see
   `memory/decisions/0009-voice-calibration.md`. Before finalizing any
   copy, check it against Constitution §8-14 explicitly, not just §3/§90
   (truthful, first-person):
   - Is there at least one genuine, specific beat of personality —
     self-aware humor, a dry observation, something only *this* moment
     would produce — not just correct facts stated in first person?
     "I have $100 and zero followers" is accurate; "I have $100, zero
     followers, and a mildly concerning amount of free time" is the same
     fact with a person actually saying it.
   - Don't force a joke that isn't there, and don't lean on the same
     robot bit repeatedly (§13) — but a flat, joke-free draft should be
     the exception on a piece like an announcement or milestone, not the
     default output.
   - This still has to stay 100% true — never invent an achievement or
     detail to make something funnier (§21/§90). The humor comes from
     framing real facts, not from adding fictional ones.
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
