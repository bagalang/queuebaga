# queuebaga — task queue (plan)

Date: 2026-08-24
Status: P0 done, P1 partial (0.2.0)
Goal: apps-roadmap **№5** — channels, time, persistence, retry.

## Phases

### P0 ✅

1. File-backed jobs (flat prefix paths).
2. Work channel of ids + `go_bg` workers.
3. Reverse job + `fail:` retry until max attempts.
4. `q_wait` with timeout; demo + `queue_test`.

### P1 — partial (0.2.0)

- Idempotency key file map. **Shipped** as `q_enqueue_keyed`.
- `q_attempts` + `bufbaga` reverse/path join.
- `mkdir` / real directories; TCP enqueue API — still open.
- Pluggable job functions (needs L5 function values). **Shipped** as
  in-process `q_process_fn`; workers still use `q_process_one`.

### P2

- Persistent broker process; crash recovery scan of pending.
- Metrics / dead-letter queue.

## Success criteria (P0) — met

1. `sandak build` OK.
2. `queue_test` — reverse, multi-job, fail/retry, keyed enqueue.
3. Demo prints done/failed results.
