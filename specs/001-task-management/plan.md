# Implementation Plan: Architecture A - Three-Layer Task Management System

**Branch**: `001-task-management` (setup-reported feature identifier; no Git repository present) | **Date**: 2026-11-06 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/001-task-management/spec.md`

## Summary

Architecture A is the BASE design: one local Python process with a menu-based terminal
interface, a task service, and a SQLite repository. It supports all five task operations
for one offline user. This deliverable ends at Phase 1 design; no application code, database,
migrations, tests, Architecture B, or final architecture comparison is generated.

## Technical Context

**Language/Version**: Python 3.12+ proposed baseline; no installed version assumed.

**Primary Dependencies**: Standard library only: sqlite3, datetime, dataclasses, pathlib,
unittest, unittest.mock, tempfile; terminal input/output for the menu.

**Storage**: SQLite file `data/tasks.db` under a selected writable local directory.

**Testing**: Planned isolated business/UI unit tests, temporary-file database integration
checks, and offline acceptance walkthroughs; no runtime testing performed.

**Target Platform**: Local desktop/laptop terminal; Windows initial review environment.

**Project Type**: Single-process CLI application; documentation-only delivery now.

**Performance Goals**: Correct complete listing and persistence for the specified 100-task
acceptance dataset. No latency requirement or measured performance claim.

**Constraints**: Single user/instance, offline, no login, three layers, bounded student effort,
parameterized SQL, input validation, successful save before acknowledgement.

**Scale/Scope**: One Task entity and five operations; no remote access or collaboration.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Obligation | Initial gate | Post-design evidence |
| --- | --- | --- |
| I. Simple, modular architecture | Pass | UI -> service -> repository; no bypass paths |
| II. Python and SQLite | Pass | Standard library and one local SQLite file |
| III. Readable, maintainable code | Pass | Explicit operations and one validation owner |
| IV. Basic unit testing | Pass | Injected repository fake; isolated integration strategy |
| V. Validation and safe access | Pass | Strict dates/title rules; bound values; atomic writes |
| Evaluation and assignment scope | Pass | FR traceability and seven tradeoff criteria; design only |

Post-design result: PASS for all gates; no exceptions or unresolved decisions.
See [architecture](architecture-a.md), [contracts](contracts/interfaces.md), and
[testing strategy](testing-strategy.md) for evidence.

## Project Structure

### Documentation (this feature)

```text
specs/001-task-management/
  spec.md
  plan.md
  research.md
  architecture-a.md
  data-model.md
  testing-strategy.md
  quickstart.md
  contracts/interfaces.md
  checklists/requirements.md
```

### Source Code (repository root)

Proposed future layout only; these application artifacts are not created:

```text
task_manager/
  __main__.py                 # startup composition
  presentation/cli.py        # TaskCLI
  business/task_service.py   # TaskService
  business/task.py           # passive Task value object
  business/errors.py         # domain/service failures
  data/sqlite_repository.py  # SQLiteTaskRepository and persistence failure
 tests/
  unit/
  integration/
 data/tasks.db               # runtime local store
```

**Structure Decision**: One package and three responsibility groups. Startup only wires
components and lifecycle. Task has no I/O dependencies and can be shared as a value object.
A concrete repository injected into the service allows a compatible test fake without an
abstract framework or a fourth layer.

## Complexity Tracking

No constitution violations. No ORM, web server, dependency injection framework, or extra
infrastructure is required for this baseline.
