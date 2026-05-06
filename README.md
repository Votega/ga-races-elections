# ga-races-elections

Georgia election race and candidate data for the 2026 election cycle, published as a static JSON file and updated automatically.

## races.json

**261 races** across federal, state executive, and state legislative offices — updated daily via GitHub Actions from [VoteGA.org](https://www.votega.org).

### Counts

| Level | Races |
|---|---|
| U.S. Senate | 1 |
| U.S. House | 14 |
| GA Statewide Executive | 10 |
| GA State Senate | 56 |
| GA House of Representatives | 180 |
| **Total** | **261** |

### Top-level structure

```json
{
  "races": [ ... ],
  "updatedAt": "2026-05-06T02:28:42Z"
}
```

### Race object

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

| Field | Values |
|---|---|
| `level` | `"federal"`, `"state-executive"`, `"state"` |
| `chamber` | See table above |
| `district` | Integer (legislative/congressional races only) |
| `activePhase` | `"primary"` or `"general"` |

### Primary phase — open primary (`ballots`)

Georgia holds an open primary: voters choose a party ballot on election day. Legislative and executive races use a `ballots` object keyed by party name.

```json
"ballots": {
  "Democrat": [ ...candidates ],
  "Republican": [ ...candidates ]
}
```

### Candidate object — legislative (GA House / GA Senate)

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

| Field | Notes |
|---|---|
| `id` | `{state}-{chamber}-{district}-{cycle}-{party}-{n}` |
| `type` | Always `"challenger"` for legislative races |
| `isIncumbent` | `true` if the candidate currently holds the seat |
| `withdrawn` | `true` if the candidate was disqualified or withdrew |
| `occupation`, `county`, `email`, `website` | Present when available from SOS qualifying data |

### Candidate object — federal / executive (incumbent reference)

For races where an incumbent is running, the entry references their record in a separate members dataset rather than duplicating data:

```json
{
  "type": "incumbent",
  "memberId": "O000174",
  "memberSource": "congress"
}
```

`memberSource` is `"congress"` for U.S. members (keyed by bioguide ID) or `"ga-members"` for state officials.

### ID conventions

| Race type | ID pattern | Example |
|---|---|---|
| GA House | `ga-house-{district}-{cycle}` | `ga-house-128-2026` |
| GA Senate | `ga-senate-{district}-{cycle}` | `ga-senate-7-2026` |
| U.S. House | `house-{district}-{cycle}` | `house-6-2026` |
| U.S. Senate | `senate-{cycle}` | `senate-2026` |
| GA Executive | `ga-{office-slug}-{cycle}` | `ga-governor-2026` |

## Data sources

- **GA legislative candidates** — [GA Secretary of State Qualifying Candidate Information](https://mvp.sos.ga.gov/s/qualifying-candidate-information). Data collected during the qualifying period (April–May 2026).
- **Federal members** — [Congress.gov API](https://api.congress.gov)
- **GA state members** — [GA General Assembly](https://www.legis.ga.gov)

## Updates

This file is published automatically from [VoteGA.org](https://github.com/Votega/votega.org) via GitHub Actions. It updates daily and whenever `races.json` changes in the source repo.

Data is current through the 2026 qualifying period. General election candidate data will be added after the May 19, 2026 primary.

## Usage

```js
const res = await fetch('https://raw.githubusercontent.com/Votega/ga-races-elections/main/races.json');
const { races } = await res.json();

const gwinnettHouse = races.filter(r =>
  r.chamber === 'Georgia House of Representatives' &&
  [96, 97, 98, 99, 100].includes(r.district)
);
```
