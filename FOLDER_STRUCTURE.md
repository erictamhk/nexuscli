# Project Folder Structure Analysis

## Overall Architecture Pattern

This project implements **Hexagonal Architecture** (also known as **Ports and Adapters Architecture), with influences from **Clean Architecture** and **Domain-Driven Design (DDD)\*\*.

## Organizational Strategy: Vertical Slice by Features

> **NOTE**: This project uses **vertical slice (feature-based) folder structure** for better scalability and maintainability in large projects. If your project is small or medium-sized, a layer-based structure might be more appropriate. Choose the organizational strategy based on project size and complexity.

## Organizational Strategy: Vertical Slice by Features

For better scalability and maintainability in large projects, this project uses **vertical slice (feature-based) folder structure**. Each business feature is self-contained with its own domain, use cases, and adapters, while maintaining hexagonal architecture principles.

```
src/main/java/.../tasks/
│
├── project/                          # Feature: Project management
│   ├── domain/
│   │   ├── Project.java              # Entity
│   │   ├── ProjectName.java          # Value Object
│   │   └── ReadOnlyProject.java     # Read-Only interface
│   ├── usecase/
│   │   ├── port/in/
│   │   │   └── add/
│   │   │       ├── AddProjectUseCase.java      # Use Case interface
│   │   │       └── AddProjectInput.java       # Input class
│   │   ├── service/
│   │   │   └── AddProjectService.java         # Use Case implementation
│   │   └── port/
│   │       ├── ProjectDto.java                 # DTO
│   │       ├── ProjectPo.java                 # PO (Persistent Object)
│   │       └── ProjectMapper.java             # Mapper
│   └── adapter/
│       ├── in/
│       │   └── controller/console/
│       │       └── AddProjectConsoleController.java
│       └── out/
│           └── repository/
│               └── ProjectRepository.java     # Feature-specific repository interface
│
├── task/                             # Feature: Task management
│   ├── domain/
│   │   ├── Task.java                 # Entity
│   │   ├── TaskId.java               # Value Object
│   │   └── ReadOnlyTask.java        # Read-Only interface
│   ├── usecase/
│   │   ├── port/in/
│   │   │   ├── add/
│   │   │   │   ├── AddTaskUseCase.java
│   │   │   │   └── AddTaskInput.java
│   │   │   ├── setdone/
│   │   │   │   ├── SetDoneUseCase.java
│   │   │   │   └── SetDoneInput.java
│   │   │   ├── deadline/
│   │   │   │   ├── DeadlineUseCase.java
│   │   │   │   └── DeadlineInput.java
│   │   │   ├── delete/
│   │   │   │   ├── DeleteTaskUseCase.java
│   │   │   │   └── DeleteTaskInput.java
│   │   │   ├── view/
│   │   │   │   ├── ViewTaskUseCase.java
│   │   │   │   ├── ViewTaskInput.java
│   │   │   │   └── ViewTaskOutput.java
│   │   │   └── today/
│   │   │       ├── TodayUseCase.java
│   │   │       ├── TodayInput.java
│   │   │       └── TodayOutput.java
│   │   ├── service/
│   │   │   ├── AddTaskService.java
│   │   │   ├── SetDoneService.java
│   │   │   ├── DeadlineService.java
│   │   │   ├── DeleteTaskService.java
│   │   │   ├── ViewTaskService.java
│   │   │   └── TodayService.java
│   │   └── port/
│   │       ├── TaskDto.java
│   │       ├── TaskPo.java
│   │       └── TaskMapper.java
│   └── adapter/
│       ├── in/
│       │   └── controller/console/
│       │       ├── AddTaskConsoleController.java
│       │       ├── CheckConsoleController.java
│       │       ├── UncheckConsoleController.java
│       │       ├── DeleteConsoleController.java
│       │       ├── DeadlineConsoleController.java
│       │       ├── TodayConsoleController.java
│       │       └── ViewConsoleController.java
│       └── out/
│           └── repository/
│               └── TaskRepository.java
│
├── todolist/                          # Feature: ToDoList aggregate root
│   ├── domain/
│   │   ├── ToDoList.java             # Entity (Aggregate Root)
│   │   └── ToDoListId.java          # Value Object
│   ├── usecase/
│   │   ├── port/in/
│   │   │   ├── show/
│   │   │   │   ├── ShowUseCase.java
│   │   │   │   ├── ShowInput.java
│   │   │   │   └── ShowOutput.java
│   │   │   ├── help/
│   │   │   │   ├── HelpUseCase.java
│   │   │   │   └── HelpOutput.java
│   │   │   └── error/
│   │   │       ├── ErrorUseCase.java
│   │   │       └── ErrorInput.java
│   │   ├── service/
│   │   │   ├── ShowService.java
│   │   │   ├── HelpService.java
│   │   │   └── ErrorService.java
│   │   └── port/
│   │       ├── ToDoListDto.java
│   │       ├── ToDoListPo.java
│   │       └── ToDoListMapper.java
│   └── adapter/
│       ├── in/
│       │   └── controller/console/
│       │       ├── ShowConsoleController.java
│       │       ├── HelpConsoleController.java
│       │       ├── ErrorConsoleController.java
│       │       ├── ConsoleControllerExecutor.java    # Command dispatcher
│       │       ├── ConsoleController.java           # Common interface
│       │       ├── Request.java                    # Request model
│       │       └── Response.java                   # Response model
│       └── out/
│           ├── presenter/
│           │   ├── ShowConsolePresenter.java
│           │   ├── HelpConsolePresenter.java
│           │   └── HelpWebPresenter.java
│           └── repository/
│               └── ToDoListRepository.java
│
├── shared/                            # Cross-cutting concerns (shared across features)
│   ├── adapter/
│   │   └── out/
│   │       ├── repository/
│   │       │   ├── ToDoListCrudRepository.java      # CRUD implementation
│   │       │   ├── ToDoListCrudRepositoryPeer.java
│   │       │   ├── ToDoListInMemoryRepository.java # In-memory implementation
│   │       │   └── ToDoListInMemoryRepositoryPeer.java
│   └── io/
│       ├── springboot/
│       │   ├── ToDoListSpringBootApp.java
│       │   └── config/
│       │       ├── RepositoryInjection.java         # Repository bean configuration
│       │       ├── UseCaseInjection.java            # Use case bean configuration
│       │       ├── ToDoListDataSourceConfiguration.java
│       │       └── WebSecurityConfig.java
│       └── standard/
│           └── ToDoListApp.java
│
└── resources/
    └── application.properties
```

## Vertical Slice vs Layer-Based Structure

### Vertical Slice (Feature-Based) Structure

**Pros:**

- All related code for a feature in one place
- Easy to locate and modify feature-specific code
- Better code organization for large projects
- Clear feature boundaries
- Easier to add/remove features
- Better cognitive load when working on specific features

**Cons:**

- More folders initially
- May have some code duplication between features
- Requires discipline to keep features independent

### Layer-Based Structure

**Pros:**

- Clear separation of architectural layers
- Easier to see all entities, all use cases, etc.
- Less folder depth initially

**Cons:**

- Related code scattered across the project
- Harder to see feature boundaries
- More difficult to remove features
- Higher cognitive load for feature changes

## Design Patterns in Vertical Slice Structure

### 1. **Hexagonal Architecture (Ports and Adapters)**

Each feature maintains hexagonal architecture:

- **Domain Layer**: Business entities and value objects
- **Use Case Layer**: Application business logic (ports and services)
- **Adapter Layer**: External system integration (controllers, presenters, repositories)

### 2. **Feature Isolation**

Each feature is self-contained:

- `project/` - All project-related code
- `task/` - All task-related code
- `todolist/` - All todolist-related code
- `shared/` - Cross-cutting concerns

### 3. **Repository Pattern**

Feature-specific repository interfaces in each feature:

- `project/adapter/out/repository/ProjectRepository.java`
- `task/adapter/out/repository/TaskRepository.java`
- `todolist/adapter/out/repository/ToDoListRepository.java`
- Shared implementations in `shared/adapter/out/repository/`

### 4. **Use Case Pattern**

Each use case encapsulated within its feature:

- `project/usecase/port/in/add/AddProjectUseCase.java`
- `task/usecase/port/in/add/AddTaskUseCase.java`
- `task/usecase/port/in/setdone/SetDoneUseCase.java`

### 5. **DTO/PO/Mapper Pattern**

Located within each feature's `usecase/port/`:

- DTOs: `ProjectDto.java`, `TaskDto.java`, `ToDoListDto.java`
- POs: `ProjectPo.java`, `TaskPo.java`, `ToDoListPo.java`
- Mappers: `ProjectMapper.java`, `TaskMapper.java`, `ToDoListMapper.java`

### 6. **Interface Segregation**

Each feature has focused, small interfaces:

- `AddProjectUseCase` - Only for adding projects
- `SetDoneUseCase` - Only for marking tasks as done
- `ViewTaskUseCase` - Only for viewing tasks

### 7. **Dependency Inversion Principle**

Features depend on abstractions:

- Use cases depend on repository interfaces
- Controllers depend on use case interfaces
- Implementations are injected in `shared/io/springboot/config/`

### 8. **Aggregate Root Pattern**

`todolist/` feature serves as aggregate root:

- Manages consistency boundaries
- Controls access to `Project` and `Task` entities
- Single entry point for ToDoList operations

### 9. **Read-Only Interface Pattern**

Located in each feature's `domain/`:

- `ReadOnlyProject.java`, `ReadOnlyTask.java`
- Prevents unauthorized modifications

### 10. **Command Pattern (Console Controllers)**

Each console command is a controller in its feature:

- `AddProjectConsoleController.java`
- `AddTaskConsoleController.java`
- `ConsoleControllerExecutor.java` in `todolist/` for dispatching

## Architecture Benefits of Vertical Slice

1. **Feature Independence**: Each feature can be developed, tested, and deployed independently
2. **Easier Navigation**: All code for a feature is in one directory
3. **Clear Boundaries**: Feature boundaries are explicit and enforced
4. **Better Scalability**: Adding new features doesn't clutter existing code
5. **Team Collaboration**: Different teams can work on different features with minimal conflicts
6. **Faster Development**: Context switching between related code is minimized
7. **Easier Refactoring**: Impact of changes is contained within feature boundaries

## Quick Reference: Where to Put Code

| Code Type                 | Location                                 | Example                            |
| ------------------------- | ---------------------------------------- | ---------------------------------- |
| New Entity (Project)      | `project/domain/`                        | `Project.java`                     |
| New Value Object          | `project/domain/`                        | `ProjectName.java`                 |
| Read-Only Interface       | `project/domain/`                        | `ReadOnlyProject.java`             |
| Use Case Interface        | `project/usecase/port/in/add/`           | `AddProjectUseCase.java`           |
| Input Class               | `project/usecase/port/in/add/`           | `AddProjectInput.java`             |
| Use Case Service          | `project/usecase/service/`               | `AddProjectService.java`           |
| Output Class              | `project/usecase/port/in/add/`           | `AddProjectOutput.java`            |
| DTO                       | `project/usecase/port/`                  | `ProjectDto.java`                  |
| PO                        | `project/usecase/port/`                  | `ProjectPo.java`                   |
| Mapper                    | `project/usecase/port/`                  | `ProjectMapper.java`               |
| Controller (Console)      | `project/adapter/in/controller/console/` | `AddProjectConsoleController.java` |
| Controller (Web)          | `project/adapter/in/controller/web/`     | `AddProjectWebController.java`     |
| Presenter Interface       | `project/usecase/port/out/`              | `ShowProjectPresenter.java`        |
| Presenter Implementation  | `project/adapter/out/presenter/`         | `ShowProjectConsolePresenter.java` |
| Repository Interface      | `project/adapter/out/repository/`        | `ProjectRepository.java`           |
| Repository Implementation | `shared/adapter/out/repository/`         | `ToDoListCrudRepository.java`      |
| Configuration Class       | `shared/io/springboot/config/`           | `UseCaseInjection.java`            |
| Main Application          | `shared/io/standard/`                    | `ToDoListApp.java`                 |

## Dependency Rules in Vertical Slice Structure

### Intra-Feature Dependencies (Within a Feature)

```
domain/ (no dependencies)
    ↑
usecase/service/ → depends on → domain/
usecase/port/ (DTOs, POs, Mappers)
adapter/in/ → depends on → usecase/port/in/
adapter/out/ → depends on → usecase/port/out/ + domain/
```

### Inter-Feature Dependencies (Between Features)

```
todolist/ (aggregate root)
    ↓ manages
project/ and task/

task/ and project/
    → depend only on shared/ for infrastructure
    → NO direct dependencies between features

shared/
    → used by all features
    → contains infrastructure implementations
```

### Dependency Rule - STRICT

- **NEVER** create direct dependencies between features (e.g., task/ → project/)
- **ALWAYS** communicate through aggregate root (todolist/)
- **NEVER** skip architectural layers within a feature
- **ALWAYS** use repository interfaces for persistence
- **NEVER** depend on concrete implementations outside your feature

## File Placement Rules for Vertical Slices

### Adding Code to Existing Feature

```
NEW ENTITY (Project) → project/domain/
NEW VALUE OBJECT (ProjectName) → project/domain/
NEW USE CASE (AddProject) → project/usecase/port/in/add/
NEW SERVICE (AddProjectService) → project/usecase/service/
NEW CONTROLLER → project/adapter/in/controller/console/
NEW DTO → project/usecase/port/
NEW PO → project/usecase/port/
NEW MAPPER → project/usecase/port/
```

### Adding a New Feature

```
Create new folder structure under src/main/java/.../tasks/
├── feature-name/
│   ├── domain/           # Entities and value objects
│   ├── usecase/
│   │   ├── port/in/     # Input ports (use case interfaces)
│   │   ├── service/     # Use case implementations
│   │   └── port/       # DTOs, POs, Mappers
│   └── adapter/
│       ├── in/          # Controllers (inbound adapters)
│       └── out/        # Presenters, repositories (outbound adapters)
```

### Adding Cross-Cutting Infrastructure

```
NEW REPOSITORY IMPLEMENTATION → shared/adapter/out/repository/
NEW CONFIGURATION CLASS → shared/io/springboot/config/
NEW FRAMEWORK SETUP → shared/io/springboot/ or shared/io/standard/
```

## Key Characteristics of Vertical Slice Structure

- **Feature-First Organization**: Code organized by business feature, not technical layer
- **Self-Contained Features**: Each feature has all its domain, use cases, and adapters
- **Aggregate Root Pattern**: One feature (todolist/) serves as entry point to manage others
- **Shared Infrastructure**: Common implementations in `shared/` to avoid duplication
- **Clear Feature Boundaries**: Dependencies between features are minimal and explicit
- **Hexagonal Architecture Maintained**: Each feature internally follows hexagonal principles
- **Dependency Direction**: Always inward within a feature, through aggregate root between features

## Migration from Layer-Based to Vertical Slice

When migrating from layer-based to vertical slice:

1. **Identify Features**: Group related entities, use cases, and controllers
2. **Create Feature Directories**: Create feature folders (project/, task/, todolist/)
3. **Move Domain Code**: Move entities and value objects to feature/domain/
4. **Move Use Cases**: Move use case interfaces and services to feature/usecase/
5. **Move Adapters**: Move controllers and presenters to feature/adapter/
6. **Create Shared Folder**: Move repository implementations to shared/adapter/out/repository/
7. **Update Imports**: Fix import statements after restructuring
8. **Update Configuration**: Update dependency injection in config classes
9. **Test Thoroughly**: Ensure all functionality still works after migration
