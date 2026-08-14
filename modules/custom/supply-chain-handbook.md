---
name: supply-chain-handbook
---

# Supply-chain operations handbook

Team memory for supply-chain agents. This is what the team already paid to
learn — do not relearn it the hard way.

## Weekly ops report

- Headline first, always in this order: orders shipped, revenue, open
  breaches. The audience scans; the table comes after.
- Revenue is **net of refunds**. Totals come from the `ops-metrics` tool,
  never hand-summed from raw rows.
- Post to the team channel before Monday 09:00 US Central, with the
  breakdown table attached.

## orders-api

- Paginate at 100; the API silently truncates larger pages.
- `status=shipped` excludes returns-in-transit — use `status=closed` for
  revenue math.

## Escalation

- Any metric moving more than 10% week-over-week gets escalated to the
  forge-dev channel with the raw query attached — not a screenshot.
- Never file tracker issues for data questions; ask in-channel first.
