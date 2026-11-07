# Architecture A: Phase 0 Research and Decisions

**Date**: 2026-11-06. Decisions apply only to Architecture A. Official sources were reviewed
by the planning research agent. No application experiments or benchmarks were executed.
All design decisions are resolved.

## Local Presentation

**Decision**: Use a menu-driven terminal interface with explicit task identities.
**Rationale**: Meets local single-user needs with no external dependencies and distinguishes
same-title tasks. Presentation delegates business policy to the service.
**Alternatives considered**: A desktop GUI adds UI setup; browser UI adds server concerns.
These are local design considerations, not an independently generated Architecture B.

## Python and SQLite

**Decision**: Propose Python 3.12+ and standard-library sqlite3 with one local file.
**Rationale**: Matches constitution and avoids an ORM or external service.
**Alternatives considered**: Third-party persistence packages add unnecessary setup.

## Transactions and Failures

**Decision**: Each mutation owns a short-lived connection configured with `autocommit=True`,
then issues SQL `BEGIN`, the parameterized mutation, and SQL `COMMIT`. On failure attempt
SQL `ROLLBACK` if a transaction remains active, then close. Do not use connection.commit()
or connection.rollback() for this mode: those methods have no effect with autocommit=True.
**Rationale**: Explicit SQL transaction boundaries avoid implicit-default ambiguity. Success
requires completed commit; read/commit/write errors become storage failures. Close even if
rollback fails. Guarantees cover normal recoverable storage failures, not physical media loss.
**Alternatives considered**: `autocommit=False` with connection.commit()/rollback() is also
valid, but this design consistently selects explicit SQL transaction control.

Sources: [Python transaction control](https://docs.python.org/3/library/sqlite3.html#transaction-control),
[SQLite transactions](https://www.sqlite.org/lang_transaction.html).

## Validation and SQL Binding

**Decision**: Validate title and strict YYYY-MM-DD in the service; bind every SQL value.
Require exact ASCII date shape before calendar parsing and canonical ISO comparison.
**Rationale**: Date parsing alone accepts additional ISO forms. Parameter binding separates
values from SQL. Editable column sets are fixed, not supplied by users.
**Alternatives considered**: Parsing without shape checks is too permissive; interpolated
user input cannot meet safe data access rules.

Sources: [Date parsing](https://docs.python.org/3/library/datetime.html#datetime.date.fromisoformat),
[SQL placeholders](https://docs.python.org/3/library/sqlite3.html#how-to-use-placeholders-to-bind-values-in-sql-queries).

## Test Isolation

**Decision**: Use unittest with an injected service fake, simulated UI, and temporary-file
SQLite integration checks.
**Rationale**: Unit tests isolate rules; a reopened file proves restart persistence. Close
connections before cleanup, especially on Windows.
**Alternatives considered**: In-memory stores cannot alone demonstrate file persistence.

Sources: [unittest](https://docs.python.org/3/library/unittest.html),
[TemporaryDirectory](https://docs.python.org/3/library/tempfile.html#tempfile.TemporaryDirectory).
