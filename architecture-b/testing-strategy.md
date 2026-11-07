# Architecture B: Testing Strategy

**Status**: Proposed tests and review procedures only. No application code or test suites
were created, and no runtime tests or performance measurements were executed.

## Unit Tests by MVC Responsibility

Use standard-library unittest with small injected collaborators. Each test supplies known
input and asserts a business outcome or an observable boundary interaction. Unit tests do not
need terminal access, a real database, or production data. The standard framework supports
fixtures, cases, and assertions: [official unittest documentation](https://docs.python.org/3/library/unittest.html).

| Test subject | Collaborators | Planned checks |
| --- | --- | --- |
| Task | None | Title stripping, blank/whitespace rejection, preserved description, default incomplete, strict dates, valid past dates, leap days, explicit status |
| Task revisions | None | Each field/combined edit, empty description, omitted fields, immutable original, fixed id/status, repeated status assignment |
| TaskCollection | Fake store | Five operations, missing identities, invalid identity/status, unknown edit fields, successful result only after store returns committed outcome |
| TaskCollection failures | Faulting fake store | No publish on failed save; field errors stop writes; persistence errors classified; empty list differs from read failure |
| TaskController | Fake Model and scripted View | Every dispatch, invalid command/id/status syntax, error-to-message selection, no success after failure, exit/cancellation |
| TerminalView | Simulated input/output | Full fields, distinguish same-title ids, completed and incomplete labels, empty message, keep/replace prompts, preserved text |

Task tests include whitespace-only Unicode titles, empty descriptions, SQL-looking text,
2026-02-30, missing date, compact/week-date inputs, zero-padding errors, valid leap days, and
past dates. Test no-op edits and same-status assignments independently of toggling. Repeated
listing must not mutate supplied Tasks. Fakes must follow the documented contracts, including
absence and commit-failure behavior; fakes alone cannot establish SQLite correctness.

## SQLite Integration Tests

Use a fresh temporary directory/database per test. Never use the eventual user's store.
Verify schema fields/defaults/checks, row-to-entity mapping, generated identity, nonreuse after
delete/recreate, duplicate titles, and quotes/SQL-looking content stored literally. Test
create/list/edit/delete/status using actual parameter binding and SQLite connections.

For partial edits assert identity, status, omitted fields, and other rows are unchanged.
For status assignment assert details are unchanged, including setting current status twice.
For every mutation type close and reopen the same file and assert saved results. A newly
created in-memory database alone cannot prove restart persistence. Seed 100 mixed-status tasks
including duplicates and compare every row/field and identity after listing/reopening.

Inject failures before a write, after SQL but before commit, and at commit using a narrow test
connection wrapper backed by a real temporary database. Check rollback, closed connections,
no success result, and identical previously saved rows after reopening. Test genuine empty
collections separately from read errors and malformed stored data. If a specific storage
fault cannot be simulated reliably, record it as pending rather than reporting success.
Close connections before temporary-directory cleanup, particularly on Windows.

## Acceptance Walkthrough

After any future implementation is separately authorized, execute every acceptance scenario
in [the shared specification](../specs/001-task-management/spec.md). At minimum:

1. Offline startup without credentials; list an empty store and see an empty-state message.
2. Create a valid task and a same-title task with an empty description; confirm distinct
   identities and incomplete defaults.
3. List all fields for both tasks, with no omissions or duplicate entries.
4. Edit each information field and combinations; explicitly clear description; verify omitted
   values, identity, completion status, and unrelated tasks remain unchanged.
5. Mark completed, set completed again, and mark incomplete; verify exact requested state.
6. Delete one selected task; confirm only that task disappears.
7. Edit/delete/change status for the deleted identity; expect missing-task errors.
8. Submit invalid titles and dates on create/edit; expect field messages and unchanged data.
9. Restart after each kind of saved change; compare expected values and deleted-record absence.
10. Verify 100-task listing and restart behavior and exercise controlled read/write/commit
    faults without false empty or success messages.

Record environment, disposable database path, input, expected/actual result, and pass/fail.
Runtime results are pending. Architecture-level document review does not count as a passed
application acceptance test or prove satisfaction of the final comparative success criteria.

## Design Review and Exit Criteria

Review [the overview's traceability table](README.md#requirement-and-constitution-review)
against FR-001 through FR-013. Inspect class/activity diagrams for matching component names and
paths. Confirm no View-to-store calls, no SQL in Controller/Task, and one validation owner.
Review the seven advantages/disadvantages criteria using design evidence, without invented
scores. The final Architecture A/B comparison remains a separate deliverable.

Future runtime acceptance requires all specified scenarios passing or explicit unresolved
limitations. No load, web integration, or multi-user tests are required for this scope. Basic
security verification covers validation, SQL binding, safe error messages, and unchanged saved
records after failures. No application launch commands are provided because no executable
application is part of this deliverable.
