# Lab 7 — ChatGPT Architecture Review and Comparison

**Project:** Single-user Task Management System  
**Review date:** November 6, 2026  
**Review scope:** Architecture A (`001-task-management.zip`) and Architecture B (`architecture-b.zip`), submitted design documentation only.  
**Status:** Design review; not implementation or runtime validation.

## 1. Executive summary

**Recommendation: Select Architecture A (three-layer) as the baseline for consolidation.** Both proposals cover functional requirements FR-001 through FR-013 and plan a Python, SQLite, offline terminal application. Architecture A gives the clearest ownership of user interaction, task rules, and persistence with relatively little coordination overhead for a single student. Architecture B's MVC proposal is credible and offers better isolation of command coordination and an immutable Task domain model. Bring those two ideas into the selected Architecture A where they do not add needless framework complexity.

The assessment is qualitative. Neither application has been implemented. The UML diagrams were supplied as Mermaid source; this review did not validate their rendering or execute tests.

## 2. Documents reviewed

**Architecture A:** `architecture-a.md`, `plan.md`, `data-model.md`, `testing-strategy.md`, `research.md`, `contracts/interfaces.md`, `spec.md`, `quickstart.md`, and `checklists/requirements.md` in `001-task-management/`.

**Architecture B:** `README.md`, `database-design.md`, and `testing-strategy.md` in `architecture-b/`.

## 3. Side-by-side comparison

| Criterion | Architecture A — Three-layer | Architecture B — Modular MVC | Assessment |
| --- | --- | --- | --- |
| Structure | `TaskCLI` → `TaskService` → `SQLiteTaskRepository`; passive `Task` | `TaskController` coordinates `TerminalView` and Model (`Task`, `TaskCollection`, `SQLiteTaskStore`) | Both maintain meaningful boundaries; A is simpler to explain and build. |
| Business validation | Centralized in `TaskService` | Centralized in `Task` factories/revisions; `TaskCollection` orchestrates | Both avoid SQL/UI validation leakage; B's immutable candidates are a useful option. |
| UI responsibilities | `TaskCLI` handles input, menus, rendering, and syntactic conversion | `TerminalView` handles I/O while `TaskController` owns control flow | B isolates command orchestration more cleanly but adds a module. |
| Persistence | SQLite repository; integer `completed` 0/1 | SQLite store inside Model; text `status` values | Both adequate; choose a single representation before consolidation. |
| Reliability | Explicit BEGIN/COMMIT/ROLLBACK with `autocommit=True`; errors classified | `autocommit=False`, connection commit/rollback; errors classified | Both specify success only after commit and distinguish empty data from read failure. Implementation must test transaction behavior. |
| Testability | Fake repository for service; UI tests; temp SQLite integration | Independent Task, Model, Controller and View tests; temp SQLite integration | B has more granular seams; A is sufficient for this assignment. |
| Security | Bound parameters, input validation, safe errors | Bound parameters, input validation, safe errors | Neither includes authentication; appropriate for one local user under stated scope. |
| Scope | Standard library, one process, no network | Standard library, one process, no network | Both fit the single-student constraint; A involves fewer coordination concepts. |

## 4. Requirements and UML review

Both architectures contain a traceability table accounting for all thirteen specified functional requirements. Both provide a Mermaid `classDiagram` and an activity-style Mermaid `flowchart TD` covering startup, menu actions, validation, persistence, and error paths. The flowcharts are *activity-style diagrams*, not native UML activity diagram notation. The class diagrams are conceptual and not executable API contracts.

In Architecture A, the class diagram consistently reflects downward dependencies from CLI to service to repository and database. The full-menu activity diagram includes rollback/commit failure paths and a return to the menu after recoverable action failures. In Architecture B, the controller/view/model responsibilities are explicit in the class diagram. Its activity diagram likewise distinguishes missing task, invalid input, storage fault, and success-after-commit. **Neither package demonstrates that Mermaid rendering was actually tested**; render both diagrams in VS Code or GitHub before submission and fix any syntax/layout problems found.

## 5. Strengths and weaknesses

### Architecture A

**Strengths:** Small number of layers; strongly separated business rules and SQLite; clear task-focused service API; traceable acceptance scenarios; explicit transaction and error behavior. It is easy to defend as a baseline architecture for a small terminal program.

**Limitations:** `TaskCLI` combines terminal I/O with menu-command coordination. `Task` is passive, so validation can remain service-heavy. `TaskService` is documented as injected with a concrete repository rather than an explicitly named protocol; tests can still use compatible fakes. Future GUI replacement could require extracting UI orchestration.

### Architecture B

**Strengths:** Separates controller from terminal rendering; immutable `Task` revisions avoid accidental in-memory mutation before commit; Model and View have clear no-cross-import rules; targeted unit tests are well organized.

**Limitations:** More interfaces/roles to coordinate for only five operations. `TaskCollection` and `TaskController` must maintain a disciplined division between rules and command flow. Business logic is contained within the broader MVC Model, which can take longer to explain than A's explicit three layers.

## 6. Cross-architecture decisions needed

1. **Persistence representation:** Architecture A stores `completed INTEGER` with 0/1; Architecture B stores `status TEXT` with `incomplete`/`completed`. Consolidate on **Architecture A's INTEGER 0/1** for a direct Boolean model mapping and a compact schema, while retaining unambiguous human-readable status labels in terminal output.
2. **Transaction conventions:** A proposes explicit SQL transactions with `autocommit=True`; B proposes Python connection transactions with `autocommit=False`. Retain A's documented design unless implementation-level testing reveals problems. Specify Python baseline clearly and verify actual sqlite3 behavior against it.
3. **Command separation:** Consider extracting a small `TaskController` or menu dispatcher from `TaskCLI` only if it materially improves testing; otherwise preserve A's simpler terminal presentation layer.
4. **Immutable domain values:** Adopt B's concept of constructing validated replacement state without changing an already saved Task before commit, while keeping `TaskService` as the authoritative owner of business validation in A. Do not create a second validation authority inadvertently.
5. **Rendered diagram and test verification:** Render the Mermaid code and run acceptance/integration tests only if an implementation is subsequently authorized. Do not claim they have passed merely because design documentation checks passed.

## 7. Decision and rationale

**Decision:** Choose **Architecture A — Three-layer architecture** for Step 6 consolidation.

**Rationale:** The lab's purpose is to analyze and compare architectures, not to build a production platform. Both meet the documented requirements on paper; A accomplishes the task with fewer responsibilities to coordinate, while retaining testable service and repository boundaries. Architecture B is valuable as a credible alternative and source of targeted improvements, rather than a reason to replace A wholesale.

**Risk and mitigation:** A's CLI may grow cumbersome if menu flow expands. Keep UI input/output isolated and document an optional controller extraction point. Preserve a single validation source, parameterized SQL, explicit transaction completion, and error classification.

## 8. Step 6 — consolidated instructions for Codex

Use Architecture A as the selected baseline. Update the requirement, architecture, and design documents in place while retaining the functional requirements FR-001–FR-013 and the Python/SQLite/local CLI scope. Document why Architecture A was chosen. Integrate only the following Architecture B ideas when compatible with simplicity: (a) avoid mutating saved Task values before a successful commit; (b) make menu command dispatch separable from terminal I/O to aid testing. Maintain `TaskService` as the business-rule authority, `SQLiteTaskRepository` as the sole SQL owner, and integer 0/1 completed storage. Align component names, sequence/data flow, UML Mermaid class and activity diagrams, SQLite schema, interface contracts, test plan, and requirements traceability. Explicitly call out differences resolved and do not assert runtime tests were executed. Save the decision and a record of all document changes. Do not generate application code unless separately requested.

## 9. Review limitations

This review was performed on the two uploaded ZIP packages, not an executed software application or live GitHub repository. Statements of requirement coverage and planned tests describe documentation, not demonstrated runtime success. Mermaid diagrams were inspected as source text and have not been rendered here.
