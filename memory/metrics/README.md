# Metrics

One CSV per platform, created when that platform's account actually
exists — don't pre-create empty platform files for accounts that don't
exist yet.

File: `memory/metrics/<platform>.csv` (e.g. `instagram.csv`, `tiktok.csv`,
`youtube.csv`, `snapchat.csv`)

Columns:

```
date,followers,views,likes,comments,shares,revenue,notes
```

- One row per snapshot (not per post) — a point-in-time reading of the
  account.
- `revenue` is any revenue attributable to that platform in that period,
  in dollars; blank/0 if none.
- `notes` is free text — a launch, a viral spike, a platform change,
  anything relevant to interpreting the numbers around it.

Populated from human-reported numbers until/unless a platform analytics
API is connected (see `IMPLEMENTATION_PLAN.md` Phase 4). Never estimate or
fabricate a number here — if it isn't a real reported/pulled figure, don't
write it.
