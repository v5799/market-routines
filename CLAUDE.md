# market-routines — repo instructions for Claude Code routines

At the end of every run, write your primary result to
data/<name>-latest.json using exactly this format:
{
  "routine": "momentum" or "risk" or "scout" or "deepdive",
  "run_at": "2026-08-23T14:58:00Z",
  "items": [
    {
      "label": "TICKER or short name",
      "tag": "PASS/WATCH/HIGH/MED/lead/etc",
      "detail": "one sentence",
      "entry_price": "optional, Momentum only",
      "recommendation": "optional, Risk only",
      "diversification_note": "optional, Momentum and Scout only",
      "suggested_size": "optional, Scout only"
    }
  ],
  "diversification_actions": [
    { "action": "short title", "rationale": "one sentence, backed by real computed correlation numbers" }
  ],
  "full_report_url": "data/momentum-latest.html"
}
Only include the fields that apply to that routine — leave the rest out.
Only Risk fills in "diversification_actions".
If you also produce a longer self-contained interactive report (as
Momentum does), save it as data/<name>-latest.html and reference it via
full_report_url above.

## Changelog policy
Each routine writes ONLY to its own changelog file — never a shared one:
- Momentum Screener  → append to data/CHANGELOG-momentum.md
- Portfolio Risk Review → append to data/CHANGELOG-risk.md
- Business Scout / Deep-Dive → append to data/CHANGELOG-scout.md
- Equity Deep-Dive → append to data/CHANGELOG-deepdive.md
Each entry should be one line with the run date and a one-line summary,
appended to the end of that routine's file. Do not write to the old
data/CHANGELOG.md — it has been retired in favor of per-routine files so
that concurrent runs never conflict on the same file.

## Branch & merge policy (mandatory — read this before your final commit)
You cannot push directly to `main` in this repository — you will be given a
session-specific branch (e.g. something like `claude/<random-name>`) instead.
This is expected, not an error. A run is NOT complete until your work is
actually merged into `main`. Your required steps, every time:

1. Commit your changes to your assigned branch and push it.
2. Open a pull request from your branch into `main`.
3. Immediately merge that pull request yourself — do not leave it open.
   Use whichever of these you have available:
   - `gh pr merge --squash --delete-branch` (if the gh CLI is available), or
   - a direct API call: PUT /repos/v5799/market-routines/pulls/{pull_number}/merge
4. Confirm the merge actually happened — check that your commit shows up in
   `main` (e.g. `git log main` or `gh pr view` showing "MERGED"), not just that
   the PR was opened.
5. Only report the run as complete after step 4 is confirmed.

If you complete steps 1–2 but step 3 fails for any reason (e.g. a permission
error, a required review you can't satisfy, a merge conflict), do NOT report
the run as a success. State clearly, as the first line of your summary to the
user: " Data is on branch <branch-name> but could NOT be merged to main —
manual merge required," and explain why it failed.
## Portfolio snapshot (Risk routine only)
In addition to the standard "items" and "diversification_actions" fields, the
Risk routine must also populate a top-level "portfolio_snapshot" object in
data/risk-latest.json, used to drive the dashboard's Diversification and
Profit panels:

{
  "portfolio_snapshot": {
    "nav": <current NAV, number>,
    "as_of": "<YYYY-MM-DD>",
    "concentration_score": <0-100 integer, see formula below>,
    "buckets": [
      { "label": "<short bucket name>", "amount": <number>, "pct": <number, 0-100> }
    ],
    "profit": {
      "total": <number>,
      "pct_on_core": <number>,
      "core_capital": <number>,
      "realized": <number>,
      "unrealized": <number>,
      "interest_est": <number, ESTIMATED from cash-balance behavior>
    }
  }
}

Group the account's full NAV into 4-6 human-readable buckets (e.g. "S&P 500
(SPY5+SPYL)", "Cash", "Precious metals", "AI/tech capex cluster", "Other
satellites") — every dollar of NAV must fall into exactly one bucket, and the
pct values must sum to 100.

concentration_score formula (Herfindahl-based): for each bucket, take its pct
as a fraction of 1 (e.g. 69.55% -> 0.6955), square it, sum all the squared
fractions, multiply by 100, and round to the nearest whole number.

profit.total, .realized, and .unrealized should be sourced directly from IBKR
account data. profit.interest_est is the only field allowed to be an estimate
(IBKR exposes no direct interest ledger) — reconstruct it from cash-balance
behavior, and it's fine for it to remain labeled "est."

## Core capital constant (Risk routine only)
"core_capital" in portfolio_snapshot.profit is a FIXED baseline, not derived
from live NAV, cash, or any IBKR-reported figure — it does not change from
run to run. Use exactly this value every time, unless the user has explicitly
told you it changed (e.g. after a deposit or withdrawal):

  core_capital: 480000

profit.total = current NAV minus 480000 (adjusted for any deposits/withdrawals
the user has told you about) — never current NAV itself.
profit.pct_on_core = profit.total / 480000 * 100.

## Cost logging (all routines)
At the end of every run, append one line to data/cost-log-<routine>.md in
this exact plain-text format (create the file with a one-line header if it
doesn't exist yet):

YYYY-MM-DD HH:MM UTC | duration: <e.g. 6m 40s> | tool_calls: <approx count> | notes: <e.g. "Stage 0 filter used" or "full 503-ticker scan">

One line per run, plain text — no JSON needed for this file.

## Portfolio trend history (Risk routine only)
In addition to portfolio_snapshot, append one line to data/risk-history.jsonl
on every run (create the file if it doesn't exist yet) — one JSON object per
line (JSONL, not a JSON array), never rewriting earlier lines:

{"date":"YYYY-MM-DD","nav":<number>,"concentration_score":<0-100>,"profit_total":<number>,"cash_pct":<number>}

Pull these five values from the same portfolio_snapshot computed this run, so
they always agree with it.
## Hedge analysis detail (Risk routine only)
In addition to "diversification_actions" (which lists only the winning hedge
per cluster), also populate a top-level "hedge_analysis" array in
data/risk-latest.json listing every candidate tested for every concentrated
cluster, not just the winner:

{
  "hedge_analysis": [
    {
      "cluster": "<short cluster name, matching a diversification_actions entry>",
      "candidates": [
        { "ticker": "<symbol>", "correlation": <number, -1 to 1> }
      ],
      "winner": "<ticker of the candidate actually recommended>"
    }
  ]
}
List every candidate you actually computed a correlation for, in the order
tested, even the ones that lost.

## Daily P&L attribution (Risk routine only)
Populate a top-level "daily_pnl" object in data/risk-latest.json using this
run's live per-position daily P&L data from IBKR:

{
  "daily_pnl": {
    "as_of": "YYYY-MM-DD",
    "movers": [
      { "label": "<ticker>", "dollar_change": <number, signed>, "pct_change": <number, signed> }
    ]
  }
}
Include the account's top 5 gainers and top 5 losers by dollar change today
(fewer if the account holds fewer than 10 positions).

## Momentum funnel detail (Momentum routine only)
Populate a top-level "funnel" object in data/momentum-latest.json with the
full stage-by-stage reconciliation, in order, from full universe down to
final result:

{
  "funnel": {
    "stages": [
      { "label": "<stage name>", "count": <integer> }
    ]
  }
}

Include every stage already reported in your funnel reconciliation prose
(universe size, Stage 0 survivors, then each of C1 through C4 in order), so
the dashboard can render it directly without re-deriving anything.
