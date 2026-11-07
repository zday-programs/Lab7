# Architecture A: Data Model and Database Design

**Baseline**: [spec.md](spec.md), [architecture-a.md](architecture-a.md)

## Task Entity and SQLite Table

Task is the only entity. Table `tasks` has no relationships or foreign keys. This is database
design documentation, not an executable migration.

| Column | Domain type | SQLite design | Meaning and constraints |
| --- | --- | --- | --- |
| id | Positive integer | INTEGER PRIMARY KEY AUTOINCREMENT | Generated once, immutable; no reuse of committed deleted ids |
| title | String | TEXT NOT NULL | Nonblank normalized title; duplicates allowed |
| description | String | TEXT NOT NULL, default empty | Empty allowed; preserve entered text |
| due_date | Calendar date | TEXT NOT NULL | Real date in canonical YYYY-MM-DD, no time/time zone |
| completed | Boolean | INTEGER NOT NULL DEFAULT 0 | CHECK value IN (0,1); 0 incomplete, 1 completed |

Include defensive nonempty-title and date-shape checks in the future table definition; true
calendar validation and whitespace normalization remain authoritative service rules. No
application-defined text length limit, unique title constraint, extra timestamps, user table,
soft deletion, or audit history. The primary-key index is sufficient for the lab dataset.
AUTOINCREMENT adds small overhead but prevents a stale deleted id from selecting a new task.

## Validation and Mapping

Service strips surrounding Python whitespace from title and rejects an empty result. Description
is text, possibly empty, otherwise preserved exactly. Reject non-text input rather than coerce.
Dates must have exactly four ASCII digits, hyphen, two digits, hyphen, two digits; parse a valid
calendar date and compare its ISO rendering to the original. Reject impossible dates, compact
ISO forms, week dates, and missing zero padding. Accept past dates and valid leap days.

Targeted actions require a positive integer identity and an existing task. Completion is an
explicit boolean, not arbitrary truthy text. Repository maps dates to ISO text and booleans to
0/1, then reverses that mapping on reads. Invalid stored values/schema cause a storage error,
not silent correction. Display identities to distinguish duplicate titles.

## State Transitions

| Before | Action | After |
| --- | --- | --- |
| Absent | Create valid task | New incomplete task with identity |
| Incomplete | Set completed | Completed; details unchanged |
| Completed | Set incomplete | Incomplete; details unchanged |
| Either status | Set current status | Unchanged status/details |
| Either status | Valid edit | Same id/status; supplied information updated |
| Either status | Delete | Absent permanently |
| Any | Invalid request or failed save | Previously saved state unchanged |
| Absent | Edit/delete/status | Not-found error; other tasks unchanged |

## Operations, Persistence, and Failure

Create inserts one row; list reads all ordered by id; lookup selects by id; edit updates only
title/description/date; set status updates only completed; delete removes by id. Every value
uses SQL parameter binding. Service merges partial edits into a fixed details-update contract;
user-provided field names are never interpolated into SQL.

Use a short-lived connection per operation, closed on every path. For mutation configure
`autocommit=True`, issue SQL BEGIN, write, and SQL COMMIT explicitly. On write/commit error,
attempt SQL ROLLBACK if active and close even when rollback fails. Python connection.commit()
and rollback() are not used in this mode. Success is returned only after SQL COMMIT succeeds.
A zero-match update/delete reports absence rather than success. An existing row assigned its
current status remains a successful idempotent assignment.

Initialization creates the directory/store/table for a fresh path while preserving existing
records. Read failure must differ from empty results. Reopen the same file to validate restart
persistence. Busy/locked or filesystem errors surface as storage failures; no retry or
multi-instance coordination is introduced. Normal recoverable failures must preserve prior
committed records; physical device recovery is outside scope.
