LAB 7 — ARCHITECTURE CONSOLIDATION INSTRUCTIONS

Selected Architecture: Architecture A (Three-Layer)

Review the existing requirements specification, Architecture A, Architecture B, and the ChatGPT Architecture Comparison Report.

OBJECTIVE:
Consolidate the two proposed architectures into one final design, using Architecture A as the baseline.

1. PRESERVE ARCHITECTURE A
- TaskCLI handles terminal input and output.
- TaskService handles business logic and validation.
- SQLiteTaskRepository manages SQLite persistence.
- Maintain the separation between presentation, business logic, and data access.

2. INCORPORATE SELECTED IMPROVEMENTS
- Use validated replacement task values rather than modifying saved task objects before database commits.
- Keep menu command dispatch separable from terminal I/O for improved testability.
- Avoid introducing unnecessary MVC components.

3. STANDARDIZE DATA STORAGE
- Use SQLite.
- Store completion status as INTEGER (0 or 1).
- Use parameterized SQL.
- Ensure transaction success is confirmed before reporting successful updates.

4. UPDATE REQUIREMENTS
- Preserve all 13 existing functional requirements.
- Maintain requirement identifiers and acceptance criteria.
- Update only where needed for architectural consistency.

5. UPDATE ARCHITECTURE AND DESIGN
- Architecture overview
- Component responsibilities
- Data flow
- Database schema
- Mermaid UML class diagram
- Mermaid UML activity diagram
- Component interface contracts
- Testing strategy
- Requirements traceability

6. DOCUMENT THE DECISION
Explain why Architecture A was selected over Architecture B.
Record which Architecture B concepts were incorporated.
Identify any remaining limitations.

7. CONSTRAINTS
- Use Python and SQLite.
- Keep the application local and single-user.
- Do not introduce cloud services or unnecessary dependencies.
- Do not implement application code.
- Do not claim runtime tests have passed.

Save the consolidated documentation separately from the original Architecture A and Architecture B files so both original designs remain available for comparison.