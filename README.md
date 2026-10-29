# JSONL Kit

A dependency-free command-line toolkit for JSON Lines (`.jsonl` / NDJSON) files. It validates records, filters nested fields, projects selected fields, and computes streaming field statistics without loading the full dataset into memory.

## Use cases

- Validate event streams before ingestion.
- Inspect large model-evaluation or telemetry exports.
- Extract a subset of records from logs without writing a custom script.
- Detect missing fields, nulls, and type drift across line-oriented data.

## Install

Requires Python 3.10+; no runtime dependencies.

```bash
python -m pip install -e .
jsonl-kit validate events.jsonl --require-object
jsonl-kit query events.jsonl --where 'status="failed"' --select id,service,error.code --limit 100
jsonl-kit stats events.jsonl --field service --field status --field user.id --format json
```

Filter literals are parsed as JSON when possible. Use quoted strings for text (`--where 'status="failed"'`), numeric literals for numbers (`--where 'latency_ms>=500'`), `true`/`false` for booleans, and `null` for null. Multiple `--where` clauses are combined with AND. Missing fields do not match a filter, including `!=`.

Nested paths use dot notation and numeric list indexes, for example `request.user.id` and `items[0].sku`. `--select` emits an object whose keys are the requested selectors; missing selected fields are emitted as `null`.

## Commands

### `validate`

Checks each nonblank line independently and returns a bounded list of errors. By default, any valid JSON value is accepted. Add `--require-object` for event records expected to be JSON objects. Exit codes: `0` valid file, `1` invalid record(s), `2` operational or argument error.

### `query`

Streams input records and emits matching JSONL records to standard output. Use `--limit` to stop after a maximum number of matches; memory use does not grow with the number of records apart from the current line/record.

### `stats`

Profiles one or more nested fields. Distinct values are tracked up to `--max-distinct` per field. A `distinct_count_is_lower_bound` flag distinguishes capped counts from exact values.

## Design and limitations

- Input is read as UTF-8, accepting a UTF-8 BOM; blank lines are ignored.
- Each line is parsed independently. JSON arrays spanning multiple physical lines are not JSONL and are not supported.
- Query processing stops with a line-specific error on malformed JSON rather than silently skipping corrupt records.
- `stats` uses bounded distinct tracking but still stores each distinct signature up to the configured cap for each requested field.
- This tool does not mutate input files or make network requests.

## Development

```bash
python -m pip install -e .
python -m compileall -q src tests
python -m unittest discover -s tests -v
```
## Project status

This is an AI-assisted initial implementation released as an alpha. Review the streaming semantics and field-path behavior before integrating it into production pipelines, and report edge cases with reproducible fixtures.
