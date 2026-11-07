# Consolidated testing strategy and traceability

Date: 2026-11-07. All runtime tests below are planned, not implemented or executed.
Use standard-library unittest/unittest.mock, injected collaborators, scripted terminal I/O,
and temporary SQLite files after implementation is separately authorized.
[Requirements](updated-requirements.md), [architecture](final-architecture.md),
[contracts](final-design.md), and [diagrams](uml-diagrams.md) define expected outcomes.

## Unit test plan

| Test group | Subject and isolation | Planned checks |
| --- | --- | --- |
| U1 | TaskService with compatible fake repository | Valid/duplicate creation, unique ids, incomplete default; list/delete normal behavior |
| U2 | TaskService validation with write-spying fake | Trim title, Unicode whitespace rejection; empty/exact description; no text length cap; strict dates, impossible/missing/compact/week dates, padding, valid leap/past dates; invalid input never writes |
| U3 | TaskService replacement values with fake repository | Individual/combined edits; omitted fields; explicitly clear description; reject unknown/id/status keys; empty edit; old snapshot unchanged before/after failed save; preserve id/status/other tasks |
| U4 | TaskService targeted operations with fake repository | Positive int excluding bool; missing edit/delete/status; explicit bool only; both completion directions and repeated assignment preserving details |
| U5 | Faulting repository fake | PersistenceError to StorageError; read failure distinct from []; no committed result after save error; no-match write becomes TaskNotFoundError |
| U6 | MenuDispatcher with fake service and no terminal | Route each of five parsed commands/arguments exactly once; invalid command no mutation; return values/errors; no business validation in dispatch |
| U7 | TaskCLI with fake dispatcher and scripted I/O | All menu choices, exit, invalid syntax, cancellation, keep/replace including empty description, id selection for duplicate titles, full fields/status/date display, safe errors and no false success |
| U8 | Passive Task snapshots | Existing snapshots never mutated by service operations or repeated listing; no domain validation factories |

Fakes must obey repository absence and commit-return contracts. A fake return alone cannot
prove transaction correctness. Explicitly test completed tasks remain visible and editable.

## Integration test plan

Use a fresh TemporaryDirectory and real SQLite file for each independent case; never the
user's data/tasks.db. Close all handles before cleanup, particularly on Windows.

| Group | Planned real-storage checks |
| --- | --- |
| I1 | Initialize fresh/reopen existing store; exact fields/defaults/checks; integer 0/1 and date mapping; malformed schema/row raises error rather than reset or skip |
| I2 | All five operations; duplicate titles; generated stable ids; no reuse of committed deleted ids; literal quotes and SQL-looking text; parameter binding |
| I3 | Partial/combined edits and explicit empty description preserve id/status/omissions/unrelated rows; status-only writes preserve all details; repeated status is matched |
| I4 | Close/reopen same file after create, edit, delete, and each completion direction; compare every field and deletion absence |
| I5 | Empty vs read failure; missing targets; read-only repeated list; 100 mixed-status tasks with duplicate titles listed exactly once and unchanged after reopen |
| I6 | For every mutation, inject failure before write, after SQL before COMMIT, and at COMMIT through narrow connection wrappers over a real temporary file; assert rollback, closure, no success, identical previously saved rows after reopen |
| I7 | Startup open/schema failure, incompatible store, locked/unwritable store and rollback failure cleanup where reliably reproducible; confirm safe messages, no reset, no false empty/success |

Verify explicit SQL BEGIN/COMMIT/ROLLBACK under Python 3.12+ autocommit=True; do not substitute
connection.commit()/rollback(). Capture original Task snapshots and saved rows around injected
failures. Test failure paths using actual transactions; mocks alone do not establish rollback.
Record unsimulated filesystem/commit faults as pending. Physical media recovery is outside scope.

## Acceptance walkthrough and success criteria

Execute every numbered scenario in all five stories in updated-requirements.md:
Story 1: scenarios 1-4; Story 2: 1-4; Story 3: 1-4; Story 4: 1-4; Story 5: 1-2.
These are 18 scenarios, plus all eight edge cases. Start offline without credentials,
create and list duplicate-title tasks, edit each field/combined values and clear description,
set completed twice then incomplete, delete and retry missing targets. Restart after each
mutation kind and verify exact values. Exercise invalid titles/dates and controlled storage
faults. Include the 100-task collection; it is an acceptance dataset, not a maximum.

SC-001 and SC-005 are document-review targets: both originals share unchanged requirements,
and the decision compares all seven criteria. SC-002 requires a future unassisted walkthrough;
SC-003 requires 100-task listing/restart execution; SC-004 requires invalid/missing/storage-fault
execution. Documentation coverage alone cannot establish those runtime outcomes.

## Requirements traceability

Sx.y means Story x, scenario y. EC means the preserved edge cases.

| Requirement | Design responsibility | Planned unit/integration verification | Acceptance source |
| --- | --- | --- | --- |
| FR-001 | Service default/validation; repository identity | U1, I1-I2 | S1.1, S1.2, S1.4 |
| FR-002 | Service title normalization and exact description | U2-U3, I2-I3 | S1.1-S1.3; S3.1, S3.2, S3.4 |
| FR-003 | Service strict calendar rules; repository ISO mapping; CLI display | U2, U7, I1 | S1.1, S1.3; date EC |
| FR-004 | Repository all rows; CLI full fields/id/status | U1, U7, I2, I5 | S2.1; S1.4 |
| FR-005 | Read-only listing; CLI genuine empty state | U5, U7-U8, I5 | S2.2, S2.4 |
| FR-006 | Service immutable merge; repository details-only update | U3, I3 | S3.1, S3.4 |
| FR-007 | Identity-targeted committed delete | U1, U4, I2, I4 | S5.1 |
| FR-008 | Explicit bool/replacement status; status-only write | U4, I3-I4 | S4.1-S4.3 |
| FR-009 | Service validation before write | U2-U3, I3, I6 | S1.3; S3.2 |
| FR-010 | Service lookup and repository no-match handling | U4-U5, I2, I5 | S3.3; S4.4; S5.2 |
| FR-011 | Local file and commit-before-return | I4-I5 | S2.3 |
| FR-012 | Error classification, explicit rollback, candidate discard, CLI messages | U3, U5, U7, I6-I7 | Read/save failure EC |
| FR-013 | Local startup and all five dispatcher/service operations | U6-U7, offline walkthrough | Stories 1-5 offline without credentials |

## Review record and future evidence

Consolidation review checks requirement text retention, all thirteen matrix rows, component and
method names, integer status schema, parameterized writes, replacement-state ownership,
dispatcher isolation, no-match handling, commit-before-success, link targets and diagram source.
This is static document verification, not application tests or proof of runtime compliance.
Mermaid rendering remains pending in a compatible viewer.

Future test records must include Python/SQLite/platform versions, disposable path, case id,
inputs, expected and actual outcomes, pass/fail, and unsimulated faults. Runtime acceptance
requires all applicable scenarios passing or clearly recorded unresolved issues. No network,
multi-user/load suite or production database is needed. No runtime tests have been run here.
