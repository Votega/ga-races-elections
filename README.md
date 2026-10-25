# ga-races-elections
Georgia election data for the 2026 cycle, maintained by [VoteGA.org](https://votega.org): **races and candidates** (updated daily) and **early-voting turnout by county** (refreshed by hand when the Secretary of State posts a new file; see [Early voting](#early-voting)).

## Files

Pick the format that fits how you work — the CSV and Markdown files are flattened views of the nested JSON.

| File | Format | Best for |
|------|--------|----------|
| [`races/2026/races.json`](races/2026/races.json) | JSON | Developers — full nested data (every candidate & phase) |
| [`races/2026/races.csv`](races/2026/races.csv) | CSV | **Spreadsheets** — one row per race |
| [`races/2026/candidates.csv`](races/2026/candidates.csv) | CSV | **Analysis** — one row per (race, phase, candidate); the deeply-nested candidate data, flattened, with incumbent names resolved. `candidateId` is blank for an incumbent listed by reference; `memberId` / `memberSource` point at their member record, and `candidateKey` is `candidateId` if there is one, else `memberId`, so every row has one join key |
| [`races/2026/races.schema.json`](races/2026/races.schema.json) | JSON Schema | Validating / typing `races.json` |
| [`races/2026/RACES.md`](races/2026/RACES.md) | Markdown | **Reading** — counts by cycle, level, and active phase |

> In `races.json`, partisan races store candidates under party-keyed `phases.<phase>.ballots`, while nonpartisan (judicial) races use a flat `phases.<phase>.candidates` array. `candidates.csv` unifies both into one table.

### Layout

```
ga-races-elections/
├── README.md, LICENSE, NOTICE.md
├── races/
│   └── 2026/                 one folder per election cycle (a cycle spans its primary, runoff and general)
│       ├── races.json, races.csv, candidates.csv, races.schema.json, RACES.md
└── early-vote/
    └── 2026-general/         one folder per election
```

> **Moved in October 2026.** The races files used to sit at the repository root (`/races.json` and so on). They now live in `races/2026/`; the old paths no longer exist. A later cycle appears as `races/2028/` with the same file names.

### Early voting (`early-vote/2026-general/`)

| File | Format | Best for |
|------|--------|----------|
| [`county-totals.csv`](early-vote/2026-general/county-totals.csv) / [`.json`](early-vote/2026-general/county-totals.json) | CSV / JSON | **Spreadsheets and analysis** — one row per county: ballots, in person, mail, turnout, and comparisons with 2022 and 2024 at the same point |
| [`county-by-day.csv`](early-vote/2026-general/county-by-day.csv) / [`.json`](early-vote/2026-general/county-by-day.json) | CSV / JSON | **Pace charts** — accepted ballots per county per day before election day (2026 and 2024) |
| `*.schema.json` | JSON Schema | Every column described, for both tables |
| [`source-data-dictionary.md`](early-vote/2026-general/source-data-dictionary.md) | Markdown | **Reading** — what each field in the Secretary of State's source files means |

## races.json
363 races across federal, state executive, state legislative, judicial, and district attorney offices — updated daily via GitHub Actions from [VoteGA.org](https://votega.org). Live counts by level and phase are in [`races/2026/RACES.md`](races/2026/RACES.md).

### Counts
| Level | Races |
| :-----: | :-----: |
| U.S. Senate | 1 |
| U.S. House | 15 |
| GA Statewide Executive (incl. Public Service Commission) | 10 |
| GA State Senate | 56 |
| GA House of Representatives | 180 |
| Superior Court | 84 |
| District Attorney | 10 |
| Georgia Court of Appeals | 5 |
| Supreme Court of Georgia | 2 |
| Total | 363 |

## Top-level structure
```json
{
  "races": [ ... ],
  "updatedAt": "2026-05-11T00:00:00Z"
}
```

### Race object — legislative / executive
```json
{
  "id": "ga-house-128-2026",
  "level": "state",
  "chamber": "Georgia House of Representatives",
  "district": 128,
  "cycle": 2026,
  "activePhase": "primary",
  "phases": {
    "primary": {
      "electionDate": "2026-05-19",
      "ballots": { ... }
    },
    "general": {
      "electionDate": "2026-11-03",
      "candidates": []
    }
  }
}
```
| Field |	Values |
| :-----: | :-----: |
| level | "federal", "state-executive", "state", "state-judicial", "local" (district attorney) |
| chamber |	See counts table above |
| district | Integer (legislative/congressional races only; absent for judicial) |
| activePhase | "primary", "runoff" or "general" |
| status | `"certified"` once the Secretary of State has certified the race; absent until then |
| winner | Present with `status: "certified"`: the winner's `name`, `candidateId` (or `memberId` when the winner is an incumbent listed by member reference), `party` and `phase` |

## Race object — judicial
Judicial races use a flat candidates array (not ballots) since all Georgia judicial elections are non-partisan.
```json
{
  "id": "superior-court-gwinnett-mason-2026",
  "level": "state-judicial",
  "chamber": "Superior Court",
  "circuit": "Gwinnett Judicial Circuit",
  "seat": "Mason",
  "displayTitle": "Gwinnett Judicial Circuit — Mason",
  "cycle": 2026,
  "activePhase": "primary",
  "phases": {
    "primary": {
      "electionDate": "2026-05-19",
      "candidates": [ ... ]
    },
    "general": {
      "electionDate": "2026-11-03",
      "candidates": []
    }
  }
}
```
| Field |	Notes |
| :-----: | :-----: |
| circuit	| Present on Superior Court races only (e.g. "Gwinnett Judicial Circuit") |
| seat | The seat name, taken from the SOS contest label (e.g. "Mason") |
| displayTitle | Human-readable title for the race (e.g. "Gwinnett Judicial Circuit — Mason") |
| _note | Present on open-seat races (e.g. "Open seat — Judge Mason not running.") |

Supreme Court and Court of Appeals races follow the same shape but omit circuit.

## Primary phase — open primary (ballots)
Georgia holds an open primary: voters choose a party ballot on election day. Legislative and executive races use a ballots object keyed by party name.
```json
"ballots": {
  "Democrat": [ ...candidates ],
  "Republican": [ ...candidates ]
}
```
Judicial races are non-partisan and use a flat candidates array instead (see below).

## Candidate object — legislative (GA House / GA Senate)
```json
{
  "id": "ga-house-128-2026-d-1",
  "type": "challenger",
  "name": "Jane Smith",
  "party": "Democrat",
  "occupation": "Attorney",
  "county": "Gwinnett",
  "isIncumbent": true,
  "website": "https://janeforgeorgia.com",
  "email": "jane@janeforgeorgia.com"
}
```
| Field |	Notes |
| :-----: | :-----: |
| id | {state}-{chamber}-{district}-{cycle}-{party}-{n} |
| type | Always "challenger" for legislative races |
| isIncumbent | true if the candidate currently holds the seat |
| withdrawn |	true if the candidate was disqualified or withdrew |
| occupation, county, email, website | Present when available from SOS qualifying data |

## Candidate object — judicial
```json
{
  "id": "jud-sup-gwinnett-mason",
  "type": "challenger",
  "name": "Tracey Diane Mason",
  "party": "Non-Partisan",
  "isIncumbent": true,
  "occupation": "Superior Court Judge",
  "email": "reelecttraceymason@gmail.com",
  "website": "https://reelectraceymason.com"
}
```
| Field | Notes |
| :-----: | :-----: |
| type | Always "challenger" for judicial candidates |
| party |	Always "Non-Partisan" |
| isIncumbent |	true if the candidate currently holds the seat |
| occupation, email, website | Present when available from SOS qualifying data |

## Candidate object — federal / executive (incumbent reference)
For races where an incumbent is running, the entry references their record in a separate members dataset rather than duplicating data:
```json
{
  "type": "incumbent",
  "memberId": "O000174",
  "memberSource": "congress"
}
```
memberSource is "congress" for U.S. members (keyed by bioguide ID) or "ga-members" for state officials. In `candidates.csv` these entries have a blank `candidateId`; they are identified by `memberId` (also given as `candidateKey`).

## ID conventions
| Race type |	ID pattern | Example |
| :-----: | :-----: | :-----:|
| GA House | ga-house-{district}-{cycle} | ga-house-128-2026 |
| GA Senate | ga-senate-{district}-{cycle} | ga-senate-7-2026 |
| U.S. House | ga-{district}-{cycle} | ga-06-2026 |
| U.S. House (special election) | ga-{district}-special-{cycle} | ga-13-special-2026 |
| U.S. Senate | senate-{cycle} |	senate-2026 |
| GA Executive | ga-{office-slug}-{cycle} | ga-governor-2026 |
| District Attorney | da-{circuit-slug}-{cycle} | da-atlantic-2026 |
| Superior Court | superior-court-{circuit-slug}-{seat-slug}-{cycle} | superior-court-gwinnett-mason-2026 |
| Court of Appeals | court-of-appeals-{seat-slug}-{cycle} |	court-of-appeals-gobeil-2026 |
| Supreme Court |	supreme-court-{seat-slug}-{cycle} |	supreme-court-warren-2026 |

## Early voting

County-level counts of early in-person and mail ballots for the **2026 general election**, in [`early-vote/2026-general/`](early-vote/2026-general/). It powers the [Early Voting Turnout page](https://www.votega.org/ga-early-vote/) on VoteGA.org. Each file says what it runs through (`as_of` / `metadata.asOf`), and the folder has its own [README](early-vote/2026-general/README.md).

### county-totals

One row per county (159). Main columns; the schema describes all of them.

| Column | Meaning |
| :-----: | :-----: |
| `county_fips`, `county` | 5-digit FIPS code and county name |
| `active_voters` | Registered voters with active status (the turnout denominator) |
| `ballots`, `in_person`, `mail` | Accepted ballots so far, and how they were cast |
| `turnout_pct_of_active` | `ballots` as a percentage of `active_voters` |
| `citizen_adults_est`, `ballots_pct_of_citizen_adults_est` | **Estimated** citizen adults (residents 18+ minus non-citizen residents of every age; runs slightly low) and ballots as a share of them |
| `mail_outstanding`, `mail_rejected`, `mail_cancelled_or_spoiled`, `mail_other` | Where mail and electronic ballots stand. A blank in the last three means 1 to 9, withheld, not zero |
| `ballots_<year>_same_point`, `ballots_2026_at_<year>_point` | A past election's ballots at the same number of days before election day, and this year's at that point |
| `early_share_<year>_pct`, `carried_<year>`, `carried_<year>_pct` | A past election's early/mail share of its final vote, and which party carried the county (R, D, O, or blank if no one cleared half) |
| `repeat_<year>_pct` | Share of this year's ballots cast by someone who also voted early or by mail in that year (a county aggregate) |

### county-by-day

One row per county, election year and day before election day, zero-filled: `election_year`, `county_fips`, `county`, `date`, `days_before`, `in_person`, `mail`, `ballots`. `days_before` 45 pools everything 45 or more days out and has no date. Only elections whose Secretary of State file is complete are included.

### Read this first

- **Elections are compared at the same number of days before election day**, not on the same date. The comparison days are in `metadata.comparisons`.
- **2022 is compared at 17 days out and has no day-by-day file.** The Secretary of State's 2022 absentee file is missing most Fulton and Gwinnett ballots after 2022-10-22, and there is no other download for it.
- **Georgia voters do not register by party**, so this data cannot say which party's voters voted early. `carried_<year>` is how the county voted in that year's top-of-ticket race.
- **The Secretary of State's file runs through the previous day**, and this dataset is refreshed by hand, so it can lag the Secretary of State's dashboard by a day or more.

### Provenance

Everything in `early-vote/` is a county-level aggregate derived from the Secretary of State's absentee-ballot and voter-history downloads, its county active/inactive voter counts and its certified results. It contains no elector-level data (no names, addresses or voter IDs), nothing below the county, and it does not use the Secretary of State's voter list products (the Voter List and Statewide Voter File ordered under O.C.G.A. § 21-2-225). County Census figures (population, citizenship, education) are from the U.S. Census Bureau's American Community Survey, compiled by [Georgia Civic Data](https://georgiacivicdata.org) ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)).

## Data sources
 - GA legislative candidates — GA Secretary of State Qualifying Candidate Information. Data collected during the qualifying period (April–May 2026).
 - GA judicial candidates — GA Secretary of State Qualifying Candidate Information. Covers Superior Court, Court of Appeals, and Supreme Court of Georgia seats on the 2026 ballot.
 - Federal members — Congress.gov API
 - GA state members — GA General Assembly
 - Early voting — GA Secretary of State absentee-ballot and voter-history downloads, active/inactive voter counts, and certified results; county Census figures from the U.S. Census Bureau American Community Survey via Georgia Civic Data (CC BY 4.0)

## Updates
`races.json` and the files built from it are published automatically from VoteGA.org via GitHub Actions. They update daily and whenever `races.json` changes in the source repo.

`early-vote/` is published by hand-triggered workflow after the Secretary of State posts a new absentee file, so its update times vary; check `as_of`. Files in this repository are overwritten on each publish, so please don't edit them here.

## Usage
```js
const res = await fetch('https://raw.githubusercontent.com/Votega/ga-races-elections/main/races/2026/races.json');
const { races } = await res.json();

// All Gwinnett House districts
const gwinnettHouse = races.filter(r =>
  r.chamber === 'Georgia House of Representatives' &&
  [96, 97, 98, 99, 100].includes(r.district)
);

// All contested judicial races
const contestedJudicial = races.filter(r =>
  r.level === 'state-judicial' &&
  r.phases[r.activePhase].candidates.length > 1
);

// All Superior Court races in a given circuit
const gwinnettCourts = races.filter(r =>
  r.circuit === 'Gwinnett Judicial Circuit'
);
```

### Early voting
```js
const base = 'https://raw.githubusercontent.com/Votega/ga-races-elections/main/early-vote/2026-general/';
const { metadata, counties } = await (await fetch(base + 'county-totals.json')).json();

// Counties furthest ahead of (or behind) 2022 at the same point before election day
const vs2022 = counties
  .map(c => ({ county: c.county, change: c.ballots_2026_at_2022_point / c.ballots_2022_same_point - 1 }))
  .sort((a, b) => b.change - a.change);
console.log(`As of ${metadata.asOf}:`, vs2022.slice(0, 5));

// Fulton County's 2024 pace, day by day (county-by-day.csv has the same rows)
const { rows } = await (await fetch(base + 'county-by-day.json')).json();
const fulton2024 = rows.filter(r => r.county_fips === '13121' && r.election_year === 2024);
```

## License
[CC-BY-SA 4.0](https://github.com/Votega/ga-races-elections?tab=CC-BY-SA-4.0-1-ov-file). See [`NOTICE.md`](NOTICE.md) for how to credit this data, including the Georgia Civic Data (CC BY 4.0) credit for the county Census figures in `early-vote/`.
