# Changes

Changes to this repo's files that a consumer may need to act on. Routine data refreshes are not listed.

## November 2026

- `results.csv` gains four columns at the end (`electionDay`, `advanceVoting`, `absenteeByMail`, `provisional`), and `results.json` the same fields on each candidate plus `metadata.countyResults`. Schema version 1.1.0; no column moved and no field was removed.
- New: `results/<election>/county-results.csv` (results by county and vote method) for the five certified 2026 elections.
- New: `results/<election>/` holds certified vote counts (`results.json`, `results.csv`, `results.schema.json`) for the 2026 primary, primary runoff, U.S. House District 13 special and special runoff, and general election. Indexed in `data/results/manifest.json`. No existing path changed.

## October 2026

- The races files moved from the repository root into `races/2026/` (`races.json`, `races.csv`, `candidates.csv`, `races.schema.json`, `RACES.md`). The root paths no longer exist. A later cycle appears as `races/<cycle>/` with the same file names.
