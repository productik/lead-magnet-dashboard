# Lead Magnet Dashboard (trial)

Static snapshot of the **IK Lead Magnet Funnel** dashboard, for dev review. It is not deployed anywhere and has no backend.

## What is here

- `index.html`: the whole dashboard (9 tabs, filters for period, tool, geo, source, device and user). Uses D3 from cdnjs and Google Fonts.
- `data/*.json`: 38 files of pre-computed, validated numbers that `index.html` loads with `fetch`. Aggregates only; no emails or person-level rows (scanned before commit).
- `data_notes.html`: the in-page "Data notes" tab content.
- `docs/table-logic.md`: every table on the dashboard, the BigQuery source it reads, and the exact logic behind each number. BigQuery project: `ik-marketing-data` (SELECT only).

Snapshot date: 2026-10-27.

## Run locally

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

`index.html` fetches `data/<name>.json`.

## Notes for devs

- The JSON files are the output of the BigQuery queries described in `docs/table-logic.md`. The queries themselves are not in this repo yet; the next step is a job that regenerates `data/` on a schedule (or a small API that serves the same shapes).
- Row lookups use a `row_id` key of the form `period|geo|source|device|user|tool` (see `FILES` and `toolRow` in `index.html`).
- `career-tools` was used as a reference only; nothing in it was changed.
