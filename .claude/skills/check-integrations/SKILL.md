---
name: check-integrations
description: Verify each connected tool in the tool stack still actually works, with a minimal non-destructive call per tool, and record results in TOOL_STACK.md. Use periodically (e.g., weekly) or any time an integration is suspected broken.
---

# Check integrations

Keeps `TOOL_STACK.md`'s status column honest — connected doesn't mean
working, and this project's own rule (Constitution §90) is not to claim
something works without a tool call proving it.

## Steps

For each tool marked 🟢 or 🟡-with-credentials-set in `TOOL_STACK.md`, make
one minimal, non-destructive call:

- **GitHub**: list open issues (cheap, read-only, already relied on
  constantly).
- **Gmail**: `list_labels` — confirms auth without reading message
  content.
- **Google Calendar**: `list_calendars` — same idea. If this errors with
  a re-authorization message, that's a human-only fix (claude.ai
  connector settings) — flag it, don't attempt to work around it.
- **Google Drive**: search for the `ClaudeTheRobot` folder — confirms
  access without creating clutter on repeat runs.
- **Buffer** (once `BUFFER_API_KEY` is set): a read-only query (e.g., list
  connected channels/profiles) — never a test post.
- **Gemini image API** (once `GEMINI_API_KEY` is set): skip routine
  testing here since every call costs money — rely on real usage instead
  of a synthetic check, unless something is actually suspected broken.

## After checking

Update `TOOL_STACK.md`'s status column for each tool with the result and
today's date (e.g., "🟢 verified 2026-08-19"). If anything that was
working now fails, note it in the day's journal entry — a broken
integration is exactly the kind of thing that should be visible, not
silently worked around.

## Guardrails

- Never spend money just to test something works — prefer read-only
  checks, and skip checks that would cost anything unless there's a real
  reason to suspect a problem.
- Never mark something 🟢 based on "it was working last time" — only based
  on this run's actual call.
