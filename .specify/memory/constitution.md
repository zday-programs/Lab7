<!--
Sync Impact Report (temporary review material; remove before committing)
Version change: unratified scaffold -> 1.0.0 (initial adoption)
Modified principles: template placeholders replaced with:
- I. Simple, Modular Architecture
- II. Python and SQLite
- III. Readable and Maintainable Code
- IV. Basic Unit Testing
- V. Input Validation and Safe Data Access
Added sections: Project Scope and Constraints; Architecture Evaluation Workflow;
project-specific Governance.
Removed sections: none; template examples removed.
Follow-up TODOs: none.
-->
# Lab7 Task Management System Constitution

## Core Principles

### I. Simple, Modular Architecture

Every proposed architecture MUST separate the user interface, business logic, and database
responsibilities into identifiable layers or modules. The user interface MUST delegate task
operations to business logic; business logic MUST access storage through the database layer.
Designs MUST remain manageable for one student. Added layers, services, frameworks, or
abstractions MUST have a documented need tied to assignment requirements. This separation
supports comparison and independent reasoning without unnecessary complexity.

### II. Python and SQLite

The backend MUST use Python, and persistent local task data MUST use SQLite. Architecture
proposals MUST identify how Python modules interact with SQLite through the database layer.
Dependencies beyond the Python standard library MUST have a documented benefit and setup
cost. Local setup MUST be reproducible from documented instructions for any executable
prototype. These constraints keep the assignment accessible to a single student.

### III. Readable and Maintainable Code

Designs and any prototype code MUST use descriptive names, cohesive modules, and explicit
interfaces between layers. Business rules MUST have a single authoritative implementation.
Documentation MUST explain module responsibilities, data flow, and consequential design
decisions. Comments MUST explain non-obvious reasoning rather than repeat code. Changes MUST
favor clear solutions that a student can explain and maintain.

### IV. Basic Unit Testing

Each architecture proposal MUST explain how business logic can be tested independently of
the user interface and production database. Any implemented task behavior MUST include basic
unit tests for its normal behavior and relevant invalid input or missing-task cases. Tests
MUST cover creating, listing, editing, deleting, and changing completion status when those
operations are implemented. Database tests MUST use isolated temporary storage. A design-only
submission MUST document the planned tests and MUST NOT claim unexecuted tests passed.

### V. Input Validation and Safe Data Access

Business logic MUST validate task input before persistence, including a non-empty title,
a valid due date, and an explicit completed or incomplete status. Specifications MUST define
accepted date formats and field constraints. Database access MUST use parameterized SQL for
user-provided values. Invalid input and missing task identifiers MUST produce understandable
errors without unintended data changes. Security measures MUST be proportionate to a local
student project.

## Project Scope and Constraints

Lab7 is a university software engineering assignment focused on generating and comparing
software architectures for a simple Task Management System. A complete application is not
required by this constitution. Prototype implementation MUST be limited to work needed for
an explicitly selected assignment deliverable.

All candidate architectures MUST address the same functional scope:

- Create a task with a title, description, and due date.
- View the list of existing tasks.
- Edit task information.
- Delete a task.
- Mark a task as completed or incomplete.

The data model MUST represent task identity, title, description, due date, and completion
status. UI technology is intentionally left to architecture evaluation. New product features
or infrastructure MUST be justified against the assignment and one-student workload before
being included in scope.

## Architecture Evaluation Workflow

Before comparing architectures, document the shared requirements, assumptions, and evaluation
criteria. Each candidate MUST describe its components, layer responsibilities, interfaces,
data model, and the flow of each required task operation.

Compare candidates against the same criteria: simplicity, modularity, readability,
maintainability, testability, basic security, and implementation effort for one student.
Record strengths, weaknesses, and tradeoffs with references to the proposed design. A
recommendation MUST explain how those tradeoffs fit the assignment; unsupported claims of
superiority MUST NOT substitute for comparison.

Specifications, plans, tasks, and architecture reviews MUST check compliance with this
constitution. Executable prototypes MUST include setup and test instructions, and reports
MUST distinguish implemented behavior, executed validation, and proposed behavior. Before
submission, verify coverage of all five task operations and document unresolved assumptions
or limitations.

## Governance

This constitution governs Lab7 specifications, architecture proposals, comparisons, plans,
and any prototypes. Conflicts MUST be resolved by revising the affected artifact or adopting
an explicit constitution amendment before relying on the conflicting design.

Amendments MUST document the proposed change, rationale, affected artifacts, and any migration
or rework needed. The student responsible for the project MUST review and accept amendments
before adoption. Each amendment MUST update the version and last-amended date while preserving
the original ratification date.

Use semantic versioning: MAJOR for incompatible principle redefinitions or removals, MINOR
for new principles or materially expanded guidance, and PATCH for clarifications that do not
change obligations. Version 1.0.0 establishes the initial project-specific constitution.

Compliance MUST be reviewed when preparing specifications and plans, comparing architectures,
and submitting assignment deliverables. Reviews MUST record deviations and their resolution.
Temporary exceptions MUST state their rationale, scope, and resolution point and receive the
project student's acceptance; they MUST NOT silently redefine a principle.

**Version**: 1.0.0 | **Ratified**: 2026-11-06 | **Last Amended**: 2026-11-06
