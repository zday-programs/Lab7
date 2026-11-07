# Architecture A: Review and Future Validation Guide

## Current Documentation Review

This is a design-only deliverable, not a runnable Task Management System. Review requires a
Markdown viewer; rendering the diagrams requires Mermaid support. Read [plan](plan.md),
[architecture](architecture-a.md), [data model](data-model.md), [contracts](contracts/interfaces.md),
and [testing strategy](testing-strategy.md). Check the five paths, three boundaries, every Task
field, and all FR identifiers. Architecture B is not generated here.

## Future Setup and Commands

Only after separately authorized implementation: install Python 3.12+ with sqlite3 support,
provide the proposed package/tests in [plan.md](plan.md), and choose a writable local directory.
No package downloads or server are planned. From the future repository root, proposed commands:

```powershell
python --version
python -m task_manager --db-path ./data/tasks.db
python -m unittest discover -s tests -p "test_*.py"
```

These are proposed execution conventions, not working commands for the present repository.
The future test layout must support unittest discovery. Tests use their own temporary paths.
Startup creates a fresh selected store without resetting an existing one. For acceptance,
select a disposable directory/database and reuse the same path after restart.

## Acceptance Walkthrough

1. Start offline with an empty disposable store. View All displays "No tasks."
2. Create "Submit lab", description "Upload report", due "2026-11-20". Expect new identity
   and incomplete status. Create a same-title task with empty description; expect distinct id.
3. View All. Expect both tasks, all fields and identities, and no duplicates.
4. Edit the first task's title, clear its description explicitly, then change its due date.
   Expect unchanged identity/status and unchanged second task.
5. Mark completed, repeat completed, then mark incomplete. Expect requested status each time
   and no information changes.
6. Delete the first task. Only the second remains. Retry edit/delete/status on deleted id;
   expect "Task not found." with no other changes.
7. Restart on the same store after each mutation type. All successful changes must persist.
8. Try blank/whitespace titles, missing date, malformed date, and "2026-02-30". Expect field
   errors and unchanged data. Valid past dates and leap days must be accepted.
9. Under controlled fixtures, inject read/write/commit faults using the testing strategy.
   Expect errors, no false success/empty-list message, and unchanged previously saved rows.
10. With 100 disposable tasks, verify every identity and field in the list and after restart.
    Every menu operation remains available without login or an internet connection.

## Evidence

Record environment, disposable path, inputs, expected/actual results, and pass/fail. Keep design
review separate from executed tests. Runtime verification is pending implementation; SC-001
and SC-005 additionally await the independent Architecture B and comparison deliverables.
