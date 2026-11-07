# Consolidated UML diagrams

Date: 2026-11-07. Conceptual design only; method contracts are defined in
[final design](final-design.md). Mermaid source is provided; rendered output is not verified.
The activity diagram uses Mermaid flowchart notation for UML-style actions and decisions.

## Class diagram

TaskList means a list of immutable Task snapshots. OptionalTask means Task or None.
Dispatcher and CLI are both in the presentation layer; startup composition is omitted.

```mermaid
classDiagram
    class TaskCLI {
        +run()
        +collect_arguments(command)
        +display_tasks(tasks)
        +display_error(message)
    }
    class MenuDispatcher {
        +dispatch(command, arguments) Outcome
    }
    class TaskService {
        +create_task(title, description, due_date) Task
        +list_tasks() TaskList
        +edit_task(id, changes) Task
        +delete_task(id) Acknowledgement
        +set_completed(id, completed) Task
        -validate_details(title, description, due_date)
    }
    class Task {
        <<immutable value>>
        +int id
        +str title
        +str description
        +date due_date
        +bool completed
    }
    class SQLiteTaskRepository {
        +initialize()
        +insert(title, description, due_date, completed) Task
        +list_all() TaskList
        +get_by_id(id) OptionalTask
        +update_details(id, title, description, due_date) OptionalTask
        +delete(id) bool
        +set_completed(id, completed) OptionalTask
    }
    class SQLiteDatabase {
        +tasks
    }
    TaskCLI --> MenuDispatcher : parsed commands and arguments
    MenuDispatcher --> TaskService : delegates five operations
    TaskService --> SQLiteTaskRepository : injected dependency
    TaskService ..> Task : validates replacement values
    SQLiteTaskRepository ..> Task : maps saved rows
    SQLiteTaskRepository --> SQLiteDatabase : bound SQL and transactions
```

Only SQLiteTaskRepository owns SQL. Task has no validation/persistence methods.
A creation candidate has no saved identity and is not the saved Task depicted here.
Failure outcomes are specified in the contracts rather than modeled as extra layers.

## Activity diagram

```mermaid
flowchart TD
    Start([Start]) --> Init[Startup initializes local SQLite store]
    Init --> Ready{Initialization successful?}
    Ready -->|No| Fatal[CLI shows startup storage error]
    Fatal --> End([End])
    Ready -->|Yes| Menu[CLI shows menu and collects choice]
    Menu --> Choice{Choice?}
    Choice -->|Exit| End
    Choice -->|Invalid| InputError[CLI shows input error]
    InputError --> Menu
    Choice -->|View All| List[Dispatcher calls service list_tasks]
    List --> Read[Repository reads and maps all rows]
    Read --> ReadResult{Read outcome?}
    ReadResult -->|Failure| LoadError[CLI shows load failure]
    LoadError --> Menu
    ReadResult -->|Empty| Empty[CLI shows No tasks]
    ReadResult -->|Tasks| Render[CLI shows every task and all fields]
    Empty --> Menu
    Render --> Menu
    Choice -->|Create or Edit or Delete or Set Completion| Input[CLI collects arguments]
    Input --> Cancel{Cancelled?}
    Cancel -->|Yes| Menu
    Cancel -->|No| Syntax{Valid command syntax?}
    Syntax -->|No| InputError
    Syntax -->|Yes| Dispatch[Dispatcher calls selected service operation]
    Dispatch --> Args{Valid argument types and edit keys?}
    Args -->|No| FieldError[CLI shows field error]
    FieldError --> Menu
    Args -->|Yes| Needs{Existing task required?}
    Needs -->|No create| Candidate[Service validates new candidate values]
    Needs -->|Yes| Fetch[Repository fetches selected id]
    Fetch --> Found{Lookup outcome?}
    Found -->|Failure| LoadError
    Found -->|Absent| Missing[CLI shows Task not found]
    Missing --> Menu
    Found -->|Found| Kind{Operation?}
    Kind -->|Delete| Begin[Repository opens transaction with SQL BEGIN]
    Kind -->|Edit| NoEdit{No supplied changes?}
    NoEdit -->|Yes| Unchanged[CLI shows No changes]
    Unchanged --> Menu
    NoEdit -->|No| Candidate
    Kind -->|Set Completion| Candidate
    Candidate --> Valid{Candidate valid?}
    Valid -->|No| FieldError
    Valid -->|Yes| Begin
    Begin --> BeginOK{Transaction opened?}
    BeginOK -->|No| SaveError[CLI shows save failure]
    BeginOK -->|Yes| Write[Fixed parameterized mutation and result snapshot]
    Write --> Written{Write outcome?}
    Written -->|Failure| Rollback[Rollback if active and discard candidate]
    Written -->|No match| Absent[End transaction and close connection]
    Absent --> Missing
    Written -->|Matched or inserted| Commit[SQL COMMIT]
    Commit --> Committed{Commit successful?}
    Committed -->|No| Rollback
    Committed -->|Yes| Publish[Close connection and return committed result]
    Publish --> Success[CLI displays success or resulting status]
    Success --> Menu
    Rollback --> Close[Close even if rollback fails]
    Close --> SaveError
    SaveError --> Menu
```

All reads close connections on success and failure. Beginning a transaction unsuccessfully
also closes the connection; no-match cleanup failure follows the storage-error path.
Same-status assignment follows the matched write/commit path. Delete requires no replacement
details; candidate validation remains in TaskService for create/edit/status.
