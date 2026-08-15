# queuebaga

A **background job queue** for Baga — apps-roadmap **№5**. Worker pool over
`chan` of job ids, payloads on disk (persistence + the only shared store
workers can reach — `go`/`chan` carry only `i64`).

This repository is the package. The compiler and `std` stay in the baga
language monorepo. Check this tree out as `app-product/queuebaga` there
(git submodule) so path deps and `-I app-product` keep working.

## Checkout

Inside a baga language clone:

```bash
git submodule update --init app-product/queuebaga
# or, first time from a fresh baga tree without the submodule recorded:
git clone git@github.com:bagalang/queuebaga.git app-product/queuebaga
```

`sandak.toml` keeps `std = { path = "../../std" }`.
`tests/queue_test.baga` stays in baga.

## Design

```
  enqueue(payload) → id
        │ write <prefix>.<id>.{job,status,…}
        ▼
  chan_send(work, id) ──► workers (go_bg)
                              │ reverse payload (demo)
                              ▼
                         status done|failed + result
```

Demo job body: **byte-reverse** the payload. Payload prefix `fail:` fails
until `max_attempts` (retry with backoff).

## API

```baga
fn q_open(prefix, max_attempts) -> Queue !IO !Par
fn q_start_workers(q, n) -> Queue !IO !Par
fn q_enqueue(q, payload) -> i64 !IO !Par     // job id
fn q_status(q, id) -> str !IO                // pending|running|done|failed
fn q_result(q, id) -> str !IO
fn q_wait(q, id, timeout_ms) -> str !IO !Par !Time
fn q_process_fn(prefix, id, max, handler) -> i64 !IO  // L5 in-process
fn q_close(q) -> i64 !Par                    // close work chan
```

Paths: `<prefix>.<id>.job|.status|.result|.attempts` (flat — no mkdir).

## Run

```bash
cd app-product/queuebaga
BAGA=../../baga sandak build
../../baga -I ../.. -I .. demo.baga

./baga -I . -I app-product tests/queue_test.baga
```

## Honest limits

See [`gaps.md`](gaps.md): no `setenv` (active prefix file), `write_file`
truncate races (q_wait polls through `""`), chan/i64-only workers, no
idempotency key yet.
