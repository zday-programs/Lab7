# Architecture A: Testing Strategy

**Status**: Planned verification only. No application code or tests implemented or executed.

## Basic Unit Tests

Use standard-library unittest and unittest.mock. Construct TaskService with a compatible fake
repository: no terminal, database, real files, or network. Simulate UI inputs and capture outputs
with a fake service for presentation tests. Assert outcomes and meaningful boundary calls.

| Area | Cases | Expected evidence |
| --- | --- | --- |
| Create | Valid fields, empty description, duplicate title, default status | Normalized values, separate identities, incomplete default |
| Validation | Blank/Unicode whitespace title, missing/bad date, 2026-02-30, leap dates, compact/week ISO forms, past date | Invalid fields rejected before writes; valid dates accepted |
| Edit | Each field, combined change, clear description, omit field, empty changes | Identity/status/other tasks preserved |
| Completion | Both directions, repeated status, wrong input type | Explicit idempotent assignment; details unchanged |
| Delete/missing id | Existing, absent, invalid id for all targeted operations | Correct deletion or classified error; no unrelated writes |
| List | Empty, mixed statuses, repeated reads | Complete list without mutation |
| Errors | Repository read/write failures | StorageError, no false success |
| UI | All menu actions, keep/replace, duplicate titles selected by id, cancellation | Correct service calls and clear messages |

## SQLite Integration Tests

Each test uses a fresh temporary directory and real SQLite file. Close connections before
cleanup, especially on Windows. Never target `data/tasks.db` or student data. Verify row
mapping, uniqueness including delete/recreate, defaults, details-only edits, status-only writes,
selected deletion, and constraints. Quotes and SQL-looking text must remain stored data.

Reopen the same temporary file after every mutation type and check saved state. Seed 100
mixed-status tasks including duplicate titles, then compare every identity and field before
and after reopening. Compare snapshots to establish unrelated records remain unchanged.
Test schema/connection/read failures separately from genuine empty collections.

Inject failures before writing, after writing but before commit, and during commit using a
narrow test connection wrapper around a real temporary store. Verify rollback, closed
connections, error reporting, and unchanged previously committed rows after reopening. A mock
that raises without exercising a transaction is insufficient evidence for rollback. If a
filesystem/commit failure cannot be reliably simulated, record that limitation and leave its
check pending. Do not claim coverage that was not executed.

## Acceptance and Architecture Review

Execute all [specification scenarios](spec.md) and the [validation guide](quickstart.md) after
future implementation. Check five operations offline without credentials, restart persistence,
complete 100-task listing, readable messages, and unchanged data for failed requests. Use the
[traceability table](architecture-a.md#requirement-traceability) for FR-001 through FR-013.
SC-001/SC-005 also depend on future Architecture B and comparison work.

Design review checks all five flows, layer boundaries, diagrams, schema, and contract consistency.
Future runtime acceptance requires all applicable expected outcomes and a record of environment,
inputs, results, and limitations. No measured performance or user satisfaction claim is made.

## Limits

Unit tests verify rules; integration tests verify SQLite; acceptance checks verify user journeys.
No load suite, network integration, multi-user checks, or extensive penetration testing is needed.
Basic safety checks cover validation, bound SQL, transaction failures, and safe error messages.
