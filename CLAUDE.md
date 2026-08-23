# market-routines — repo instructions for Claude Code routines

This repository is a landing zone for automated routine output only. No app code lives here.

At the end of every routine run in this repo:

1. Write your primary structured result to data/<routine-name>-latest.json
   (momentum, risk, scout, or deepdive-<TICKER>). Keep the same schema
   between runs so a dashboard can render it without changes.

2. If you also produce a longer interactive report beyond the short JSON
   summary (tables, full analysis, sourcing) — as the Momentum screener
   does — save it as a fully self-contained HTML file at
   data/<routine-name>-latest.html. Do not link out to a claude.ai
   artifact URL; the HTML must stand on its own since it will be viewed
   outside any Claude session.

3. Add a "full_report_url" field to the JSON pointing at that file's
   relative path, e.g. "data/momentum-latest.html".

4. Commit both files and push directly to main — this repo has no
   protected-branch workflow, direct pushes are expected.

5. Append one line to data/CHANGELOG.md noting the run date and a
   one-line summary, so past runs stay auditable even though the
   -latest files get overwritten each time.
