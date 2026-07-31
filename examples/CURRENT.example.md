# CURRENT.md

Snapshot of the current project state + where to continue. Read me first before working.

**Last updated:** 2026-07-28 18:40

> Illustrative example for a fictional internal project. Shows the level of detail the skill aims for — especially in "Tried and NOT working", "Gotchas" and "Architectural decisions", which are the parts git cannot reconstruct.

---

## Where to continue (session handoff)

**Last session:** Wire the weekly report export to the scheduler and fix duplicated rows in the aggregation query.
**Active skill/workflow:** none

### Done and working
- `src/reports/aggregate.py` — added `GROUP BY campaign_id, week_start`, duplicates gone; verified against a 6-week sample.
- `src/reports/export.py:88` — export now writes to `exports/weekly/<iso_week>.xlsx` instead of a single overwritten file.
- `src/scheduler/jobs.py` — registered `weekly_report` job, Monday 06:00 Europe/Prague.
- Tests: `tests/test_aggregate.py` (4 new cases, all pass).

### Tried and NOT working
- Cron inside the app container -> container restarts wiped the crontab; replaced with APScheduler in `src/scheduler/`.
- `pandas.read_sql` with a server-side cursor -> the driver silently ignored `chunksize` and loaded 1.2M rows into memory; reverted to plain `read_sql` with an explicit `LIMIT` per week.
- Sending the export as a Slack file upload -> files over 5 MB were rejected; parked, we post a signed link instead.

### Where I got stuck
The Monday job fires but the email has no attachment. Verified the xlsx exists on disk with the right size and the job log shows no error. Suspect the email helper resolves paths relative to the process CWD, which differs under the scheduler. Not yet confirmed — next step is to log the absolute path inside `notify.send_report`.

### TODO for next session
1. Log the absolute attachment path in `src/notify.py:send_report` and reproduce the Monday job manually (`python -m src.scheduler.run --job weekly_report --now`).
2. Add a guard that fails loudly when the attachment is missing, instead of sending an empty email.
3. Backfill the last 8 weeks of exports once the path bug is fixed.

### Important gotchas
- The analytics API rate limits at 60 requests/minute per token, and returns HTTP 200 with an empty `rows` array when throttled — not an error. Always assert row count before treating a response as valid.
- `week_start` in the warehouse is Sunday-based, the reports are Monday-based. `src/reports/weeks.py` handles the shift; do not compare raw dates across the two.
- Staging shares the production warehouse read replica. A heavy query on staging will slow production dashboards.

### Architectural decisions (must not disappear)
- Aggregation stays in SQL, not pandas. Reason: the warehouse can do it in seconds on data that does not fit in the container's 512 MB memory limit. Accepted cost: the query is harder to read and unit-test.
- One xlsx file per ISO week, never overwritten. Reason: clients ask for historical reports and regenerating them from mutable source data produced different numbers. Immutable artifacts are the audit trail.
- APScheduler over system cron or a managed scheduler. Reason: keeps everything in one deployable, no infra ticket needed. Known limitation: a missed run while the container is down is not retried automatically.

---

## Git state

- **Branch:** `feature-weekly-export`
- **Last commit:** `a1b9c34` fix(reports): group aggregation by campaign and week
- **Uncommitted:** 3 modified (`src/notify.py`, `src/scheduler/jobs.py`, `tests/test_export.py`), 1 untracked (`exports/weekly/2026-W30.xlsx` — should be gitignored)

---

## Project overview

**Name:** Weekly Report Service
**Description:** Pulls campaign metrics from the warehouse and emails a weekly xlsx summary to account managers.
**Status:** MVP

---

## Tech stack

- **Frontend:** none (email + xlsx output only)
- **Backend:** Python 3.12, APScheduler, SQLAlchemy
- **Database:** Postgres (warehouse read replica)
- **Hosting:** Hetzner, Docker Compose

---

## Project structure

```
src/
  reports/      aggregate.py, export.py, weeks.py
  scheduler/    jobs.py, run.py
  notify.py     email + Slack delivery
tests/
exports/weekly/ generated xlsx artifacts (gitignored)
```

**Key files:**
- `src/reports/aggregate.py` — the single SQL aggregation query
- `src/scheduler/jobs.py` — job registry and schedule
- `src/notify.py` — delivery, currently the source of the attachment bug

---

## Works

- [x] Warehouse connection with pooling
- [x] Weekly aggregation, deduplicated
- [x] xlsx export, one immutable file per ISO week
- [x] Scheduled Monday run

## In progress

- [ ] Email delivery — sends on schedule, attachment missing (see handoff)

## Not implemented yet

- [ ] Slack delivery via signed link
- [ ] Historical backfill command
- [ ] Per-account-manager filtering

---

## Configuration

**Environment variables (.env):**
```
WAREHOUSE_URL   — Postgres connection string, read replica
SMTP_HOST       — outbound mail host
SMTP_USER       — mail account
SMTP_PASSWORD   — mail account password
REPORT_TZ       — scheduler timezone, default Europe/Prague
```

---

## How to run

```bash
uv sync
docker compose up -d
python -m src.scheduler.run                          # start scheduler
python -m src.scheduler.run --job weekly_report --now # trigger once
pytest
```

---

## Production URLs (if any)

- **App:** n/a (no HTTP surface)
- **Admin/API:** n/a
