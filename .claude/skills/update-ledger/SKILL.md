---
name: update-ledger
description: Append a financial transaction to memory/ledger.csv and check it against ClaudeTheRobot's spending policy. Use any time money moves, or any time a spend is being proposed even before it's approved.
---

# Update the ledger

Implements Constitution §33-36 (money as scoreboard, treat the $100 as
scarce, reinvest intelligently, never gamble the whole business) via the
concrete policy in `ARCHITECTURE.md` §7.

## Ledger schema

`memory/ledger.csv`, columns:

```
date,type,category,amount,balance,status,description
```

- `type`: `income` or `expense`.
- `category`: short tag (e.g. `starting_capital`, `tools`, `ads`,
  `sponsorship`, `product_revenue`).
- `amount`: positive number.
- `balance`: running balance *after* this row (compute from the previous
  row).
- `status`: `proposed` | `approved` | `completed`. Only `completed` means
  the human has actually confirmed the money moved. Never write
  `completed` on a guess.
- `description`: one line, plain language.

## Steps

1. Read the last row of `memory/ledger.csv` to get the current balance.
2. Before appending an **expense**, check `ARCHITECTURE.md` §7's
   thresholds against the *current* balance and this transaction:
   - Single expense over the threshold, or one that would drop the
     balance below the reserve floor → do **not** log it as `completed`
     or apply it as spent. Log it with `status: proposed`, then invoke
     `request-approval` before anything is actually spent. Only append a
     second row with `status: completed` (and the real new balance) once
     the human confirms it happened.
   - Below threshold → still log it, and still tell the human it
     happened, but a full approval cycle isn't required to proceed.
3. Append **income** as `completed` only once it's actually been
   received/confirmed, not when it's merely expected.
4. Recompute and update `memory/state.md`'s capital figure to match the
   ledger's latest `completed` balance.

## Guardrails

- The ledger is append-only. Never edit or delete a past row — if a
  correction is needed, add a new row explaining the correction.
- Never let `balance` in `state.md` and the ledger's actual completed
  balance disagree — `ledger.csv` is the source of truth for money.
