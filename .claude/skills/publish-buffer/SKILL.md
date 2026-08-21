---
name: publish-buffer
description: Publish or schedule a post through Buffer's API once BUFFER_API_KEY is configured and a platform account is linked. Use when a content package in content/queue/ is approved and ready to go live.
---

# Publish via Buffer

Implements the Tier 1 publishing path from `TOOL_STACK.md` and
`memory/decisions/0004-tool-stack-strategy.md`.

## Precondition check (do this first, every time)

1. Check whether `BUFFER_API_KEY` is set in the environment. If it isn't,
   **stop** — say plainly that Buffer isn't connected yet, point at the
   open approval request, and do not fabricate a successful publish.
2. Check `memory/state.md` for a recorded Buffer channel/profile ID for
   the target platform. If none is recorded, stop — there's nothing to
   publish to yet, even with a valid key.

## Current API note (endpoint confirmed live 2026-08-21)

Buffer's **legacy REST API** (`/updates/create.json` etc.) is being
retired February 1, 2027 — do not build against it. The **current API is
GraphQL at `https://api.buffer.com/graphql`** (POST, JSON body
`{"query": "..."}`, `Authorization: Bearer $BUFFER_API_KEY`) — confirmed
working 2026-08-21 with a real key: `query { account { id name } }`
returned the real Buffer account (id, name), and schema introspection is
enabled. Root query fields available: `account`, `dailyPostingLimits`,
`channel`, `channels`, `aggregatedPostMetrics`, `post`, `posts`,
`postTemplate`, `postTemplates`, `ideaGroups`, `ideas`. `channels` takes a
required `ChannelsInput!` argument whose shape hasn't been resolved yet —
introspect `ChannelsInput` (`query { __type(name: "ChannelsInput") {
inputFields { name type { name kind } } } }`) before using it, since no
channel is linked yet to test against. Mutation names (for actually
creating/scheduling a post) are still unconfirmed — introspect
`mutationType` the first time this skill runs with a channel to publish
to, then update this note.

Buffer also runs a hosted MCP server at `mcp.buffer.com/mcp` with a
published Claude setup guide — if that MCP server gets connected to this
environment (ask the human to add it as a connector), prefer calling it
directly over hand-rolling GraphQL requests.

## Steps (once the precondition check passes)

1. Confirm the post still matches what was approved (check the linked
   approval record in `memory/approvals/resolved/` or the content
   package's `decision` pointer) — don't publish something that changed
   since approval without a fresh approval.
2. **Even with Buffer connected, this policy still applies**: individual
   posts require `request-approval` before publishing, at least until
   there's a track record of this pipeline working correctly. Connecting
   Buffer authorizes the *mechanism*; it does not, by itself, authorize
   *every future post* — that's a deliberate, conservative reading of
   `ARCHITECTURE.md` §7, recorded in
   `memory/decisions/0005-integration-layer.md`. Revisit this once the
   pipeline has proven itself.
3. Call Buffer's API (GraphQL) or MCP tool to create/schedule the post.
4. On success, move the content package from `content/queue/` to
   `content/posted/`, record the post URL/ID, and update
   `memory/metrics/<platform>.csv` with a baseline (0 views/likes at
   post time) so future snapshots have a starting point.
5. On failure, do not mark anything as posted — log what happened in the
   day's journal entry and leave the package in `content/queue/`.

## Guardrails

- Never claim a post went live without a successful API response
  confirming it (Constitution §90).
- Never publish something that wasn't through `request-approval` first.
- If `BUFFER_API_KEY` is unset, this skill's only job is to say so clearly
  — it is not a fallback trigger for manual posting instructions (that's
  just normal conversation, not this skill).
