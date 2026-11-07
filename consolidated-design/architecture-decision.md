# Architecture decision: Architecture A with selected B improvements

Date: 2026-11-07. Status: selected consolidated design; documentation only.

## Inputs and precedence

Reviewed the [shared specification](../specs/001-task-management/spec.md); all Architecture A
documents: architecture-a.md, plan.md, data-model.md, research.md, quickstart.md,
testing-strategy.md, contracts/interfaces.md, checklists/requirements.md under
[specs/001-task-management](../specs/001-task-management/); all Architecture B documents:
[README](../architecture-b/README.md), [database design](../architecture-b/database-design.md),
[testing strategy](../architecture-b/testing-strategy.md); the
[comparison report](../Lab7_Architecture_Comparison_Report.md);
[consolidation instructions](../consolidation-instructions.md); and the
[constitution](../.specify/memory/constitution.md).

The report's Step 6 suggests editing in place. The user's Step 7 request and consolidation
instructions explicitly require separate output and preserved originals, so consolidated-design/
contains all new deliverables. Original requirements, Architecture A, Architecture B, and the
comparison report remain unchanged. Historical proposal metadata is retained in the copied
requirements baseline; the appended consolidation addendum establishes the current scope.

## Selection rationale and tradeoffs

Architecture A remains the baseline: TaskCLI -> TaskService -> SQLiteTaskRepository.
TaskService is the single business-rule authority. B's full MVC introduces coordination roles
that are unnecessary for five operations in one local student application.

| Criterion | Selected A-based benefit | Tradeoff relative to B |
| --- | --- | --- |
| Simplicity | Three explicit layers, one process and standard library | Presentation still owns interaction sequencing |
| Modularity | UI, business rules, and storage have strict boundaries | B offers separate View/Controller roles; selected dispatcher supplies only the useful routing seam |
| Readability | Five service methods and one validation authority | Rules remain service-heavy rather than on B's Task factories |
| Maintainability | Changes localized by layer; immutable snapshots | More careful replacement handling; future GUI may warrant further UI separation |
| Testability | Fake repository, I/O-free dispatcher, scripted CLI, real temporary SQLite | Additional seam tests; fakes cannot prove SQLite behavior |
| Basic security | Parameterized SQL, validation, safe errors | No account access control; local file permissions remain an OS responsibility |
| One-student effort | No framework, cloud, ORM, or full MVC module set | Terminal prompts, error handling and transaction tests still need care |

These are qualitative design judgments, not measurements, benchmark results or runtime claims.

## Incorporated B concepts

1. Validated replacement state: service constructs new values for edit/status, never modifies
   a saved Task before commit, and publishes a saved replacement only after persistence returns.
   Task stays passive/immutable; B's entity validation factories are not adopted, avoiding
   competing validation authorities.
2. Separable command dispatch: a small MenuDispatcher within presentation routes collected
   arguments without terminal I/O. TaskCLI retains prompts, rendering and the menu loop.
   This adopts B's testing seam without TaskController/TerminalView/TaskCollection.

## Resolved differences

| Difference | Consolidated choice |
| --- | --- |
| A integer completed vs B text status | completed INTEGER constrained to 0/1; Python bool; UI completed/incomplete words |
| A explicit SQL vs B connection transactions | Python 3.12+ autocommit=True and SQL BEGIN/COMMIT/ROLLBACK consistently |
| B Task owns invariants vs A service validation | TaskService remains authoritative; immutable passive snapshots |
| B controller/view/model modules | Three A layers plus a small presentation dispatcher |
| Database defaults differ | A's data/tasks.db under selected writable local directory |
| Proposal comparison wording vs current deliverable | Originals preserved; only separate consolidated documentation created |

Commit must succeed before mutation success. Empty edit is a checked no-op with "No changes."
Same-status assignment commits idempotently. No-match and storage errors are distinct.

## Change record

| New file | Consolidated contribution |
| --- | --- |
| [updated-requirements.md](updated-requirements.md) | Exact original specification plus scoped consolidation addendum; all thirteen identifiers, stories, scenarios, edge cases, outcomes and assumptions retained |
| [final-architecture.md](final-architecture.md) | Three layers, dependency boundaries, presentation dispatch seam and all five data flows |
| [final-design.md](final-design.md) | Passive replacement values, service/repository/dispatcher contracts, schema, strict validation, safe errors and transaction lifecycle |
| [uml-diagrams.md](uml-diagrams.md) | Consistent class and full-menu activity-style Mermaid source, including error/no-op paths |
| [testing-strategy.md](testing-strategy.md) | Planned isolated unit/integration/acceptance checks and explicit FR-001 through FR-013 matrix |
| [architecture-decision.md](architecture-decision.md) | Selection, adopted improvements, precedence resolution, tradeoffs and limitations |

## Coverage, limitations, and review status

Document review accounts for all 13 functional requirements, five stories, 18 numbered
acceptance scenarios and eight edge cases. Requirements text is copied without alteration;
architectural clarifications are appended separately. Traceability assigns every requirement a
design owner and planned verification. Constitution v1.0.0 is satisfied at the design level:
three responsibilities, Python/SQLite, explicit interfaces, planned isolated tests, input
validation and safe access. No exception or new dependency is introduced.

One active instance is assumed. Concurrency, accounts, remote access, synchronization, undo,
backup/restore and physical-device recovery remain outside scope. AUTOINCREMENT adds small
overhead to avoid reused committed deleted ids. Future implementation must verify Python/SQLite
transaction behavior and controlled failure cases; unrepeatable fault simulations must be marked
pending. Mermaid diagrams have source-level review only; rendering is pending.
No application code, database, migrations, executable tests or runtime performance results
are delivered. No runtime tests are claimed passed.
