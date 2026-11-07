# Architecture A: Interaction and Layer Contracts

These are design contracts, not implementation code or a network API.

## Local User Interface

Numbered menu: Create, View All, Edit, Delete, Set Completion, Exit. No login/network required.
Each list entry shows identity, full title/description, YYYY-MM-DD date, and completed/incomplete.
Base ordering is by identity; the requirements do not mandate ordering.

Create prompts for all information fields. Edit selects a positive numeric identity, then asks
whether to keep or replace each field. Replacing description may supply empty text; keeping
omits the field from changes. Replacing title/date with blank is invalid. Delete selects by
identity with no required confirmation or undo. Completion selects identity and an explicit
completed/incomplete choice, never implicit toggling.

Invalid menu/id/status syntax yields an input message without mutation. Business errors identify
the field. Missing task: "Task not found." Empty collection: "No tasks." Read failure: "Unable
to load tasks." Save failure: "Unable to save changes." Do not print raw SQL/path exceptions.
Success follows committed storage only. Action failure returns to menu; initialization failure
reports an error and exits. Cancelling at a prompt discards unsubmitted input.

## Business Service

| Operation | Inputs | Success | Failure |
| --- | --- | --- | --- |
| create_task | title, description, due_date text | Saved Task, generated id, incomplete | ValidationError, StorageError |
| list_tasks | None | Task list, possibly empty | StorageError |
| edit_task | id, supplied title/description/due_date changes | Saved Task; omitted fields/id/status preserved | ValidationError, TaskNotFoundError, StorageError |
| delete_task | id | Committed deletion acknowledgement | ValidationError, TaskNotFoundError, StorageError |
| set_completed | id, explicit boolean | Saved Task with requested status | ValidationError, TaskNotFoundError, StorageError |

Reject unknown edit keys, including id and completed. Empty changes return the existing task
unchanged after checking existence; UI reports no change. ValidationError contains field and
explanation; TaskNotFoundError identifies the requested task. StorageError indicates failed
persistence, never invalid business input. See [data model](../data-model.md) for exact rules.

## Repository

Service receives a repository satisfying these operations; no abstract framework is necessary.

| Operation | Success/absence contract |
| --- | --- |
| initialize | Valid existing store preserved or new empty store ready |
| insert(title, description, due_date, completed) | Committed Task with generated identity |
| list_all | Task list ordered by identity, possibly empty |
| get_by_id(id) | Task or no match |
| update_details(id, title, description, due_date) | Committed Task or no match |
| delete(id) | True after committed deletion; false if absent |
| set_completed(id, completed) | Committed Task or no match, including unchanged status |

Persistence failures raise PersistenceError; service translates to StorageError. Startup maps
initialization failures for UI display too. No match is an absence result, converted by service
to TaskNotFoundError. Matched existing unchanged rows must be distinguished from missing rows.

Every mutation owns one connection/transaction. All user values are bound parameters; SQL
statements/column sets are fixed. Writes return only after explicit SQL COMMIT. No transaction
spans multiple user actions. Test fakes must obey these contracts but cannot establish real
SQLite transaction correctness.
