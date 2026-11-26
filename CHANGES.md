# Changes

Changes to this repo's files that a consumer may need to act on. Routine data refreshes are not listed.

## November 2026

- New: `results/<election>/` holds certified vote counts (`results.json`, `results.csv`, `results.schema.json`) for the 2026 primary, primary runoff, U.S. House District 13 special and special runoff, and general election. Indexed in `data/results/manifest.json`. No existing path changed.

## October 2026

- The races files moved from the repository root into `races/2026/` (`races.json`, `races.csv`, `candidates.csv`, `races.schema.json`, `RACES.md`). The root paths no longer exist. A later cycle appears as `races/<cycle>/` with the same file names.
