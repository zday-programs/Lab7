# Final design and interface contracts

Date: 2026-11-07. Documentation only. See [architecture](final-architecture.md),
[requirements](updated-requirements.md), [diagrams](uml-diagrams.md), and [tests](testing-strategy.md).

## Domain and schema

Task is a passive immutable saved snapshot. The service derives replacement values rather than
mutating it; Task has no validation factories. A creation candidate carries validated details
and completed=False, without a saved identity. Only insertion assigns the saved identity.

| Field | Python/domain type | SQLite declaration/constraint | Meaning |
| --- | --- | --- | --- |
| id | Positive int; bool excluded | INTEGER PRIMARY KEY AUTOINCREMENT | Stable and immutable; committed deleted ids not reused |
| title | str | TEXT NOT NULL; CHECK(length(title) > 0) | Trim surrounding whitespace; duplicates allowed |
| description | str | TEXT NOT NULL DEFAULT '' | Empty permitted, preserve entered text exactly |
| due_date | datetime.date | TEXT NOT NULL; length 10 and ASCII digit/hyphen YYYY-MM-DD shape check | Canonical calendar day, no time or timezone |
| completed | bool | INTEGER NOT NULL DEFAULT 0; CHECK(completed IN (0,1)) | 0 incomplete; 1 completed |

This is a schema specification, not an executed migration. The single tasks table has no
foreign keys, user table, timestamps, history, soft deletion, or title uniqueness. There is no
application-defined text length limit. Only the primary-key index is needed for this scope.
Database title/shape checks are defensive; they do not replace the service's Unicode whitespace
and real-calendar rules. On reading, repository decoding verifies stored types and canonical
values, including normalized nonblank titles and valid date/status, and raises PersistenceError
on malformed rows/schema instead of correcting or skipping them. This is integrity checking,
not a second business-rule authority. Decode 0/1 strictly before producing a boolean.

## Authoritative validation

TaskService owns one shared details-validation path for create and edit:
- Require title and description to be strings; reject coercion. Strip Python surrounding
  whitespace from title and reject empty/Unicode-whitespace-only results.
- Preserve description character-for-character, including surrounding whitespace and empty text.
- Require date text with exactly four ASCII digits, hyphen, two digits, hyphen, two digits.
  Parse a real calendar day and require canonical ISO rendering to equal input.
  Reject missing dates, 2026-02-30, week/compact forms, and missing padding.
  Accept past, present, future, and valid leap days.
- Require a positive int identity, explicitly excluding bool, for edit/delete/status.
- Require an actual boolean for completion, converted from an explicit presentation choice.
- Accept only title, description, due_date edit keys. Reject id, completed, and unknown keys.
  Omitted keys retain values. Explicit empty description clears it; blank title/date fails.

For targeted actions, validate argument shape/types, fetch existing Task, then merge/validate
replacement details as applicable. Missing tasks stop without writes. Delete validates only
identity/existence. Invalid create/edit cannot call a repository mutation.

## Presentation contracts

TaskCLI offers Create, View All, Edit, Delete, Set Completion, Exit.
Each task shows its id, full title, full description, YYYY-MM-DD date, and completed/incomplete
label. Same-title tasks are selected by id. List ordering by id is a repeatability choice,
not a new requirement. Editing asks keep/replace independently for each field, making an empty
description replacement distinct from omission. Delete is permanent; confirmation is not required.
Cancellation discards unsubmitted input without a service mutation; exit closes the loop.

MenuDispatcher.dispatch(command, arguments) has no terminal I/O. Supported parsed commands
create/list/edit/delete/status route to the corresponding service method and return its result.
TaskCLI handles exit before dispatch. Invalid dispatch commands/argument syntax yield InputError
without service mutation. No dispatcher rule trims fields or parses calendar dates.
TaskCLI renders results and catches InputError and service errors; dispatcher propagates
service errors without converting them to success.

## Service contracts

| Operation | Inputs | Successful result | Declared failures |
| --- | --- | --- | --- |
| create_task | title, description, due_date text | Committed Task with new id and incomplete status | ValidationError, StorageError |
| list_tasks | None | List of Tasks, possibly empty | StorageError |
| edit_task | id, changes mapping | Committed Task preserving id/status/omissions; existing Task for empty changes | ValidationError, TaskNotFoundError, StorageError |
| delete_task | id | Committed deletion acknowledgement | ValidationError, TaskNotFoundError, StorageError |
| set_completed | id, completed bool | Committed Task with exactly requested status | ValidationError, TaskNotFoundError, StorageError |

Empty edit returns unchanged existing values with "No changes." and makes no save claim.
All other mutation success results require repository commit. TaskNotFoundError identifies
the requested id. ValidationError carries field and explanation. StorageError carries an
operation category suitable for safe display, not raw SQL/path information.

## Repository contracts

| Operation | Inputs | Success/absence |
| --- | --- | --- |
| initialize | Configured database path | Fresh compatible table ready or existing records preserved |
| insert | Valid title, description, date, completed=False | Committed Task with generated id |
| list_all | None | All Tasks ordered by id, including completed; [] only on successful empty read |
| get_by_id | id | Task or None |
| update_details | id, title, description, date | Committed Task or None if absent; only details changed |
| delete | id | True after committed deletion; False if absent |
| set_completed | id, bool | Committed Task or None if absent; only status changed |

Every storage/connection/schema/mapping failure raises PersistenceError. Service translates it
to StorageError; startup handles initialization errors for safe display. None/False absence
becomes TaskNotFoundError. Existing same-status rows count as matched, not absent.
No-match writes roll back/end their transaction and close, returning absence without success.

All SQL statements and column sets are fixed. Every user value, including identity, uses
sqlite3 placeholders and bound parameters. Do not interpolate values or user-selected column
names. Convert date to canonical text and bool to integer 0/1 at the repository boundary.
Return complete saved Tasks, never cursors or raw rows.

## Atomicity and candidate publication

Use Python 3.12+ with an explicit autocommit=True connection per mutation and SQL
BEGIN / COMMIT / ROLLBACK, as selected in Architecture A. Read/construct the return snapshot
within the transaction before COMMIT where needed; publish it only after COMMIT succeeds.
This avoids an extra fallible read being needed after a successful save. Do not use Python
connection.commit()/rollback() in this mode. On failed write/commit, roll back if active and
close on every path, even if rollback itself fails. Never publish an uncommitted candidate.
Read operations close connections too. Startup creates directories/store only where appropriate;
it must not reset or overwrite incompatible existing data.

## Error handling and state transitions

| Condition | Observable behavior | Saved state |
| --- | --- | --- |
| Invalid command/id/status syntax | Understandable input error; menu again | Unchanged |
| Invalid business input | Field-specific error; menu again | Unchanged |
| Absent edit/delete/status target | "Task not found."; menu again | Unchanged |
| Successful empty read | "No tasks." | Unchanged |
| Read failure or corrupt row | "Unable to load tasks."; never false empty | Unchanged by read |
| Recoverable write/commit failure | "Unable to save changes."; no success; discard candidate | Prior committed rows unchanged |
| Startup failure | "Unable to initialize task storage."; exit | No reset |
| Committed create | Show success, identity and incomplete status | New independent row |
| Committed details edit | Show saved replacement | Same id/status; only supplied details changed |
| Committed status assignment | Show requested status; never toggle | Same details/id, including repeated assignment |
| Committed delete | Show deletion success | Selected row absent; others unchanged |

Raw SQL and filesystem exceptions are not printed. No transaction spans commands, retries,
multi-instance locking scheme, backup/restore, or physical-device recovery is introduced.
