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
