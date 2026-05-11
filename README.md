# ga-races-elections
Georgia election race and candidate data for the 2026 election cycle, published as a static JSON file and updated automatically.

## races.json
352 races across federal, state executive, state legislative, and judicial offices — updated daily via GitHub Actions from VoteGA.org.

### Counts
| Level |	Races |
| :-----: | :-----: |
| U.S. Senate |	1 |
| U.S. House | 14 |
| GA Statewide Executive | 10 |
| GA State Senate |	56 |
| GA House of Representatives	| 180 |
| Superior Court | 84 |
| Georgia Court of Appeals | 5 |
| Supreme Court of Georgia | 2 |
| Total | 352 |

## Top-level structure
{
  "races": [ ... ],
  "updatedAt": "2026-05-11T00:00:00Z"
}

### Race object — legislative / executive
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
Field	Values
level	"federal", "state-executive", "state", "state-judicial"
chamber	See counts table above
district	Integer (legislative/congressional races only; absent for judicial)
activePhase	"primary" or "general"

## Race object — judicial
Judicial races use a flat candidates array (not ballots) since all Georgia judicial elections are non-partisan.
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
### Field	Notes
circuit	Present on Superior Court races only (e.g. "Gwinnett Judicial Circuit")
seat	The seat name, taken from the SOS contest label (e.g. "Mason")
displayTitle	Human-readable title for the race (e.g. "Gwinnett Judicial Circuit — Mason")
_note	Present on open-seat races (e.g. "Open seat — Judge Mason not running.")
Supreme Court and Court of Appeals races follow the same shape but omit circuit.

## Primary phase — open primary (ballots)
Georgia holds an open primary: voters choose a party ballot on election day. Legislative and executive races use a ballots object keyed by party name.
"ballots": {
  "Democrat": [ ...candidates ],
  "Republican": [ ...candidates ]
}
Judicial races are non-partisan and use a flat candidates array instead (see below).

## Candidate object — legislative (GA House / GA Senate)
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
Field	Notes
id	{state}-{chamber}-{district}-{cycle}-{party}-{n}
type	Always "challenger" for legislative races
isIncumbent	true if the candidate currently holds the seat
withdrawn	true if the candidate was disqualified or withdrew
occupation, county, email, website	Present when available from SOS qualifying data

## Candidate object — judicial
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
Field	Notes
type	Always "challenger" for judicial candidates
party	Always "Non-Partisan"
isIncumbent	true if the candidate currently holds the seat
occupation, email, website	Present when available from SOS qualifying data

## Candidate object — federal / executive (incumbent reference)
For races where an incumbent is running, the entry references their record in a separate members dataset rather than duplicating data:
{
  "type": "incumbent",
  "memberId": "O000174",
  "memberSource": "congress"
}
memberSource is "congress" for U.S. members (keyed by bioguide ID) or "ga-members" for state officials.

## ID conventions
Race type	ID pattern	Example
GA House	ga-house-{district}-{cycle}	ga-house-128-2026
GA Senate	ga-senate-{district}-{cycle}	ga-senate-7-2026
U.S. House	house-{district}-{cycle}	house-6-2026
U.S. Senate	senate-{cycle}	senate-2026
GA Executive	ga-{office-slug}-{cycle}	ga-governor-2026
Superior Court	superior-court-{circuit-slug}-{seat-slug}-{cycle}	superior-court-gwinnett-mason-2026
Court of Appeals	court-of-appeals-{seat-slug}-{cycle}	court-of-appeals-gobeil-2026
Supreme Court	supreme-court-{seat-slug}-{cycle}	supreme-court-warren-2026

## Data sources
GA legislative candidates — GA Secretary of State Qualifying Candidate Information. Data collected during the qualifying period (April–May 2026).
GA judicial candidates — GA Secretary of State Qualifying Candidate Information. Covers Superior Court, Court of Appeals, and Supreme Court of Georgia seats on the 2026 ballot.
Federal members — Congress.gov API
GA state members — GA General Assembly
Updates
This file is published automatically from VoteGA.org via GitHub Actions. It updates daily and whenever races.json changes in the source repo.

Data is current through the 2026 qualifying period. General election candidate data will be added after the May 19, 2026 primary.

## Usage
const res = await fetch('https://raw.githubusercontent.com/Votega/ga-races-elections/main/races.json');
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

