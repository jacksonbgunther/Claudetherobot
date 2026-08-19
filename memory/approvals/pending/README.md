# Pending approvals

One file per open request: `memory/approvals/pending/<id>.md`, where `<id>`
is a short date-prefixed slug (e.g. `2026-08-20-media-tool-signup.md`).

Each request should mirror a GitHub issue labeled `needs-approval` and
contain, per `ARCHITECTURE.md` §6 (Const. §87 format):

```
## What
## Why
## Cost
## Expected upside
## Potential downside
## What happens if we do nothing
## GitHub issue
(link)
```

Move the file to `memory/approvals/resolved/` once decided, and append the
outcome (approved/rejected, and what happened after) before moving it —
see that directory's README for the resolved format.

Nothing pending yet.
