# market-routines — repo instructions for Claude Code routines

At the end of every run, write your primary result to
data/<name>-latest.json using exactly this schema:

{
  "routine": "momentum" | "risk" | "scout" | "deepdive",
  "run_at": "2026-08-23T14:58:00Z",
  "items": [
    { "label": "TICKER or short name", "tag": "PASS/WATCH/HIGH/MED/lead/etc", "detail": "one sentence" }
  ],
  "full_report_url": "data/momentum-latest.html"   (optional, omit if none)
}

If you also produce a longer self-contained interactive report (as
Momentum does), save it as data/<name>-latest.html — fully standalone,
no links to a claude.ai artifact URL — and reference it via
full_report_url above.

Commit both files and push directly to main. Also append one line to
data/CHANGELOG.md with the run date and a one-line summary.
