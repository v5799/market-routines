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

Commit both files and push directly to main. Also append one line to
data/CHANGELOG.md with the run date and a one-line summary.
