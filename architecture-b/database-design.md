# Architecture B: Database and Model Design

See [MVC overview](README.md). This document specifies a proposed schema and contracts;
it creates no database, migrations, or application implementation.

## Entity and Table

Task represents one independent item of work. The Model contains validated Task values and a
SQLiteTaskStore that maps them to one `tasks` table. There are no users, ownership relationships,
subtasks, dependencies, or foreign keys.

| Attribute / column | Task representation | SQLite design | Meaning |
| --- | --- | --- | --- |
| id | Positive integer for saved Task; absent for draft | INTEGER PRIMARY KEY AUTOINCREMENT | Stable identity, never edited; deleted committed identities not reused |
| title | Normalized string | TEXT NOT NULL | At least one non-whitespace character; duplicate titles allowed |
| description | String | TEXT NOT NULL, default empty string | Empty allowed, otherwise preserved exactly |
| due_date | Calendar date | TEXT NOT NULL | Canonical YYYY-MM-DD, no time of day/time zone |
| status | Task exposes completed boolean | TEXT NOT NULL DEFAULT 'incomplete' | CHECK status IN ('incomplete', 'completed') |

Store maps completed false/true to incomplete/completed text and reverses the mapping. Using
named status values keeps manual inspection understandable; only these two states exist.
A defensive nonempty-title check belongs in the schema, but full whitespace and calendar
validation belongs to Task. No application-defined text length limit is introduced. There is
no unique constraint on title. Only the primary-key index is needed for the lab's 100-task
dataset; that dataset is not a maximum task count. AUTOINCREMENT trades small storage/write
overhead for avoiding reused deleted identities.

## Validation Ownership

Task factory and revision operations use one shared validation path:

- Title must be text; remove surrounding whitespace and reject an empty result.
- Description must be text; permit empty text, preserving all other characters and whitespace.
- Due-date input must have exactly four ASCII digits, hyphen, two digits, hyphen, two digits.
  Parse a real calendar date and require its canonical ISO form to equal the input. This
  rejects impossible dates and other ISO forms. Past/present/future dates are accepted.
- Completion assignment requires a boolean, mapped from an explicit Controller choice.
- Saved identity is immutable. TaskCollection checks positive integer identities (excluding
  boolean arguments) and existence before targeted operations.

Task produces revised immutable candidates. A failed validation creates no persisted change.
Partial edit omission retains a value; explicitly empty description clears it. Title/date may
not be cleared. Unknown edit fields are rejected. Identity/status are not information edits.

Python's date parser accepts ISO representations beyond the required presentation format,
so shape checking precedes parsing. See [official date parsing documentation](https://docs.python.org/3/library/datetime.html#datetime.date.fromisoformat).

## Persistence Contracts

SQLiteTaskStore owns all connection and SQL behavior. Public Model operations return Tasks,
not cursors or rows. Its internal storage contract is:

| Operation | Result |
| --- | --- |
| prepare | Fresh table ready or existing data preserved; failure is explicit |
| read_all | All saved Tasks ordered by id, or storage error |
| find(id) | Task, absence, or storage error |
| add(draft) | Committed saved Task including generated identity |
| replace_details(candidate) | Committed Task with details updated; missing-row result if absent |
| write_status(id, completed) | Committed Task with only status updated; missing-row result if absent |
| remove(id) | Committed deletion acknowledgement; missing-row result if absent |

Every write binds user values through SQL placeholders; fixed operation-specific statements
prevent user-controlled column names. Invalid stored status/date/title or malformed schema is
a storage error rather than a corrected or omitted row. Read failures never mean an empty list.
SQL binding is documented in the [Python sqlite3 reference](https://docs.python.org/3/library/sqlite3.html#how-to-use-placeholders-to-bind-values-in-sql-queries).

## Transaction Lifecycle

Propose Python 3.12+ sqlite3 connections explicitly configured with `autocommit=False`. Each
store mutation owns a fresh connection and uses connection.commit() after its fixed writes.
On write or commit failure it attempts connection.rollback(), then closes the connection even
if rollback fails. Success must not escape before commit. This transaction mode is explicitly
chosen rather than relying on evolving implicit defaults. See [Python transaction control](https://docs.python.org/3/library/sqlite3.html#transaction-control).

A store operation closes connections for reads too. A read snapshot needs no commit; closing
ends its transaction. Matched-row/existence checks distinguish missing rows from unchanged
values, so repeated status assignment is successful. A zero-match targeted write becomes
TaskMissing, never a success. There is no transaction spanning several user menu actions.

TaskCollection fetches, derives a candidate, then asks the store to persist it. One active
application instance is assumed; no concurrent editor contract is added. Because no long-lived
mutable Task cache exists, rollback does not require restoring a separately mutated cached
entity. Previously returned values remain snapshots, never claims of newly committed state.

Store path is selected at startup; a proposed default is a `task-data/tasks.sqlite3` file under
the local working directory. Initialization creates a new store if necessary and never deletes
or resets existing tasks. Existing incompatible data produces an initialization error, not an
automatic destructive repair. Locked/unwritable/corrupt storage is reported to Controller.
Normal recoverable save failures preserve previously committed rows; physical device recovery,
backup/restore, and multiple simultaneous instances are outside the specification.

## State Transitions

| State before | Action | State after |
| --- | --- | --- |
| No task | Valid creation | Saved incomplete Task with new identity |
| Incomplete | Assign completed | Completed, same details/id |
| Completed | Assign incomplete | Incomplete, same details/id |
| Either | Assign current status | Same state, successful idempotent assignment |
| Either | Valid information revision | Updated specified details; same status/id |
| Either | Delete | Permanently absent |
| Any | Invalid input or failed save | Previously saved state unchanged |
| Absent | Edit/delete/status | Missing-task error; other tasks unchanged |

Completed tasks remain visible and editable. Listing is read-only and returns each record once.
