# Final architecture: consolidated three-layer design

Date: 2026-11-07. Status: documentation only; no implementation or runtime results.
Baseline: [updated requirements](updated-requirements.md). Decision: [architecture decision](architecture-decision.md).
Contracts and storage: [final design](final-design.md). Diagrams: [UML](uml-diagrams.md).

## Scope and composition

Use Python 3.12+ with standard-library sqlite3, datetime, dataclasses, pathlib, unittest,
unittest.mock, and tempfile as appropriate. One synchronous local process serves one user and
one active instance. All five operations work offline without registration or login. No ORM,
cloud service, server, background worker, third-party framework, or network dependency is needed.
The default database is data/tasks.db relative to a selected writable local directory.
Startup resolves that path, initializes storage, constructs repository, service and presentation,
and starts the CLI. Startup is composition, not a fourth layer.

## Layers, responsibilities, and permitted dependencies

| Layer/component | Owns | Must not own |
| --- | --- | --- |
| Presentation: TaskCLI | Menu loop, prompts, parsing menu/id/status syntax, rendering full fields and safe messages | Business validation, repository access, SQL |
| Presentation: MenuDispatcher | Map a parsed command and collected arguments to exactly one TaskService operation; return outcome or propagate service error | Terminal I/O, normalization, persistence |
| Business: TaskService | Authoritative validation, normalization, incomplete default, partial edit merge, replacement candidates, existence checks, error translation | SQL, connections, printing |
| Shared domain value: Task | Passive immutable snapshot of id, title, description, calendar due date, completed boolean | Validation factories, I/O, saving |
| Data access: SQLiteTaskRepository | Schema initialization, parameter binding, row mapping, identities, connections, transactions, no-match results | Menu behavior or authoritative business rules |
| SQLite file | Committed task rows and defensive constraints | Application orchestration |

TaskCLI calls MenuDispatcher, which calls TaskService, which calls SQLiteTaskRepository.
Only the repository accesses sqlite3. Both service and repository use the passive Task value,
which imports neither. Service receives a concrete repository through constructor injection;
compatible fakes satisfy the same documented contract without an abstract framework.
MenuDispatcher is a small presentation collaborator, not an MVC controller or additional layer.
TaskCLI retains the terminal loop and I/O. There is no TerminalView or TaskCollection.

## Data flows

| Action | Presentation to service | Service and repository | Returned/displayed result |
| --- | --- | --- | --- |
| Create | create_task(title, description, due_date text) | Validate; construct incomplete candidate values; insert | Committed Task with generated id; then success |
| View All | list_tasks() | list_all reads ordered by id; map all valid rows | Every task once with id and all fields, or genuine empty message |
| Edit | edit_task(id, changes) | Validate id/keys; fetch; merge omitted fields; validate a new candidate; update_details | Committed replacement; same id/status and unrelated rows |
| Delete | delete_task(id) | Validate id; fetch/check; delete only selected id | Committed acknowledgement |
| Set Completion | set_completed(id, explicit boolean) | Validate; fetch; derive replacement; set_completed updates only status | Committed requested status; same details/id |

A fetched Task is never modified. Valid replacement values exist only as uncommitted candidates
until the repository returns after commit. Failed candidates are discarded, leaving old snapshots
and saved rows unchanged. No persistent mutable cache exists. Empty edit checks existence,
returns the existing Task without a write, and displays "No changes." Same-status assignment
uses an idempotent status write and still confirms commit before reporting success.
All targeting uses identity, never title. Completed tasks remain visible and editable.

## Connection, transaction, and failure flow

Initialization preserves an existing compatible store and creates a fresh one when absent.
An incompatible schema or unreadable store produces a safe startup error and stops startup;
there is no automatic reset or destructive migration.

Retain Architecture A's explicit transaction convention: each mutation owns a short-lived
connection configured with autocommit=True, issues SQL BEGIN, its fixed parameterized write,
and SQL COMMIT. Do not mix in connection.commit()/rollback() for this mode. On write or
commit failure, attempt SQL ROLLBACK if active, discard candidates, and close even if rollback
fails. Read connections also close on all paths. Initialization schema creation uses the same
explicit commit discipline. No transaction spans menu actions.

Repository absence is distinct from failure. Targeted writes check matches, including an
existing row already holding the requested status. Service maps absence to TaskNotFoundError
and PersistenceError to StorageError. Action errors render safe messages and return to the
menu. Failed reads never produce "No tasks."; mutation success never precedes commit.
Fetch-then-write assumes one active application instance; no concurrent-editor guarantee is added.
Normal recoverable save failures preserve committed rows; physical device loss is outside scope.

## Requirement coverage

All FR-001 through FR-013 are retained verbatim in updated-requirements.md. The operation,
validation, failure, persistence, and offline responsibilities above are mapped individually
to planned checks in [testing strategy](testing-strategy.md). No runtime behavior is claimed.
