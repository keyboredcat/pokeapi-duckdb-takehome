# Data Engineering Take-Home (2-3 hours): PokeAPI -> DuckDB Mart (Emerald Filter)

## Objective

Build a small, rerunnable pipeline that pulls data from a public API, writes an auditable local DuckDB artifact, and produces one core mart table.

Emphasis is on API interaction quality and reproducibility; dimensional modeling is optional.

## Timebox

- Aim for 2--3 hours.

## Submission (important)

- Submit code only (plus `README.md` and any config files needed to run).
- Do not include the generated DuckDB database file in your submission.
- Git-based submissions are highly encouraged:
  - Preferred: a GitHub repository link (public or private).
  - If private, grant access to the reviewers and include the repo URL in your submission.
  - A clear commit history is a plus, but not required.

## Requirements

### 1) One non-interactive run command (required)

- Provide a single command that runs end-to-end (fetch -> persist -> mart).
- This can be a CLI you implement (e.g., `python -m ...`) or an orchestrator-driven command (allowed, highly optional).
- Must run headlessly from the command line (no manual UI steps).
- `README.md` must show the exact command(s) to run.

#### Notebook policy

- Notebooks are allowed.
- Please also provide the command-line run path described above (avoid notebook-only submissions).

### 2) Source API (required)

Use PokeAPI v2:

- `GET https://pokeapi.co/api/v2/pokemon/{id}`

Docs:

- https://pokeapi.co/docs/v2
- https://pokeapi.co/docs/v2#pokemon-section

Inputs:

- `start_id` and `end_id` (inclusive)
- Fetch details for each ID in the window.

Fair use:

- Be considerate of the public API (reasonable request pacing; avoid unnecessary refetches within a single run).

### 3) Local persistence: DuckDB file (required)

- Pipeline writes outputs to a local DuckDB database file (output path configurable).

DuckDB docs:

- https://duckdb.org/docs/
- https://duckdb.org/docs/api/python/overview

## Required output

### Core mart table: Emerald-only + array column (required)

Table: `mart_emerald_pokemon`

Filter requirement:

- Include only Pokemon that "appear in Pokemon Emerald".
- Definition: a Pokemon is "in Emerald" if its detail payload contains at least one `game_indices` entry with `version.name = 'emerald'`.

Required columns (minimum):

- `pokemon_id` INT
- `name` TEXT
- Plus at least one of:
  - `type_names` as an array/list of TEXT values, OR
  - `move_names` as an array/list of TEXT values

Notes:

- The array can be a native DuckDB LIST/ARRAY type or a JSON string representing an array.
- You may add more columns to the mart if you want (keep the required minimum present).

## Optional (encouraged)

You may create additional tables if you want to show more, for example:

- raw landing tables (e.g., persisting the full JSON payloads)
- curated/dimensional models (e.g., `dim_pokemon`, bridge tables for types/moves)
- additional mart tables

## Rerun behavior (required)

- Re-running for the same `start_id/end_id` must not create duplicate records in `mart_emerald_pokemon`.
- Results should be deterministic for a given window.

## Reliability (required, lightweight)

Implement retry OR clear error logging/reporting (either is acceptable).

- If some IDs fail, the run must still complete and report which IDs failed (logs are fine).

## Data checks (optional)

If you have time, add a couple of simple checks on the mart table (as tests or runtime validation). Examples (pick any two):

- `pokemon_id` is unique and non-null in `mart_emerald_pokemon`
- `name` is non-null in `mart_emerald_pokemon`
- the array column you chose (`type_names` or `move_names`) is non-null in `mart_emerald_pokemon`

## Deliverables

- Source code
- `README.md` with:
  - setup steps
  - exact run command for a small window (e.g., 1--10)
  - exact run command for a larger window (e.g., 1--151)
  - how to set DuckDB output path
  - brief description of the required table (`mart_emerald_pokemon`)

## Appendix: Reviewer Workflow (example DuckDB queries)

```sql
select count(*) as mart_rows
from mart_emerald_pokemon;

select pokemon_id, count(*) as c
from mart_emerald_pokemon
group by 1
having count(*) > 1;
```

```sql
-- Adjust to whichever array column you implemented
select count(*) as null_array_rows
from mart_emerald_pokemon
where type_names is null;  -- or move_names is null
```
