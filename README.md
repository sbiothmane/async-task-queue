# async-task-queue

An async task queue with an operations dashboard — built to demonstrate backend reliability engineering end to end: durable job storage, at-least-once delivery, safe retries, dead-letter replay, idempotency, and live observability.

**Stack:** FastAPI · Redis · Python workers · PostgreSQL · React + TypeScript · Docker Compose · Prometheus

## Architecture

```
Client ──► FastAPI (producer API) ──► PostgreSQL (job store, source of truth)
                    │                        ▲
                    ▼                        │ status updates
                  Redis (broker) ──► Workers ┘
                    ▲                        │
                    └──── replay ──── DLQ ◄──┘ (after max retries)

React/TS ops dashboard ──► FastAPI (query + actions: cancel / replay)
Prometheus scrapes API + workers (/metrics)
```

- **FastAPI producer** — enqueue, status, cancel, and replay endpoints.
- **PostgreSQL job store** — every job is a durable record with a full state history; the database is the source of truth, not the broker.
- **Redis broker** — carries job IDs to workers; supports priorities and delayed jobs.
- **Python workers** — pull jobs, execute, heartbeat, and report status back to Postgres.
- **Ops dashboard** — queue depth, job search/filtering, job detail, worker status, retry/replay actions.

## Job model (contract)

States: `queued → running → succeeded | failed → (retrying) → dead`

API:
- `POST /jobs` — enqueue (accepts an idempotency key)
- `GET /jobs/{id}` — status and attempt history
- `POST /jobs/{id}/cancel` — cancel a queued or running job
- `POST /jobs/{id}/replay` — replay a dead-lettered job

## Deliverables

### Phase 1 — Architecture + repo skeleton
- [ ] Job model and API contract finalized (above)
- [ ] Repo layout: `api/` · `worker/` · `dashboard/` · `infra/`
- [ ] Docker Compose boots Postgres + Redis + API + worker
- [ ] One stub job flows end-to-end: producer → Redis → worker → Postgres status update

### Phase 2 — Core queue
- [ ] Priorities and delayed/scheduled jobs
- [ ] Full job lifecycle with attempt history in Postgres

### Phase 3 — Reliability
- [ ] At-least-once delivery semantics (ack after successful execution)
- [ ] Retries with exponential backoff + jitter, configurable max attempts

### Phase 4 — Dead-letter queue + replay
- [ ] Jobs land in the DLQ after max attempts, with failure context preserved
- [ ] Replay from API and dashboard

### Phase 5 — Idempotency
- [ ] Idempotency keys on enqueue: duplicate submissions return the original job

### Phase 6 — Observability
- [ ] Worker heartbeats + liveness in the dashboard
- [ ] Prometheus metrics: queue depth, job latency, retry/DLQ counts

### Phase 7 — Ops dashboard
- [ ] Queue overview, job list with filters, job detail view
- [ ] Actions: cancel, retry, replay

### Phase 8 — Ship it
- [ ] Load test run; throughput and latency numbers published below
- [ ] Final README pass: setup, architecture diagram, design decisions

## Definition of done

- `docker compose up` runs the whole system from a clean checkout
- Core queue semantics covered by automated tests (delivery, retry, DLQ, idempotency)
- Load-test numbers published in this README (jobs/sec, p95 end-to-end latency)
- Architecture and key design trade-offs documented

## Running locally

> Lands with Phase 1.

```bash
docker compose up
```

## Load test results

> Published at Phase 8.
