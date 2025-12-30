# Nexus AI Agent Development Guidelines

## Purpose

These are **MANDATORY** guidelines for AI agents working on the Nexus project. AI agents MUST strictly follow ALL rules defined in these guidelines before writing any code.

## Mandatory Pre-Work

### 1. Read Both Documents Before Coding

- Read **docs/architecture/ARCHITECTURE.md** completely
- Read **docs/engineering/METHODOLOGIES.md** completely
- Read **docs/engineering/patterns/PATTERNS.md** completely
- Understand Hexagonal Architecture pattern
- Identify all applicable design patterns for task
- Reference specific rules when making implementation decisions

### 2. Verify Understanding

- Identify which architectural layer changes are needed
- Confirm which patterns apply to task
- List all conventions that must be followed
- Acknowledge dependency rules and constraints

## Architecture Adherence Rules

### Dependency Rule - STRICT

- **NEVER** violate inward dependency direction
- Adapters depend on Ports (interfaces), NOT implementations
- Use cases depend on Ports, NOT concrete adapters
- Domain layer has NO dependencies on outer layers
- Check every import statement against this rule

### Layer Boundaries - STRICT

- **NEVER** skip layers (e.g., Controller directly calling Repository)
- **ALWAYS** communicate through interfaces (Ports)
- **NEVER** create dependencies between adapters
- **NEVER** use concrete classes from other layers

### Vertical Slice Structure - MANDATORY

**For Existing Features:**

```typescript
NEW ENTITY           → feature-name/domain/
NEW VALUE OBJECT     → feature-name/domain/
NEW USE CASE INTERFACE → feature-name/usecase/port/in/
NEW USE CASE SERVICE   → feature-name/usecase/service/
NEW CONTROLLER         → feature-name/adapter/in/controller/console/ or web/
NEW PRESENTER INTERFACE → feature-name/usecase/port/out/
NEW PRESENTER IMPLEMENTATION → feature-name/adapter/out/presenter/
NEW REPOSITORY INTERFACE → feature-name/adapter/out/repository/
NEW DTO               → feature-name/usecase/port/
NEW PO                → feature-name/usecase/port/
NEW MAPPER           → feature-name/usecase/port/
```

**For New Features:**

Create new feature folder with complete structure:

```
feature-name/
├── domain/              # Entities and value objects
├── use-cases/          # Input ports (use case interfaces)
│   ├── port/in/        # Input ports (use case interfaces)
│   ├── service/        # Use case implementations
│   └── port/          # DTOs, POs, Mappers
└── adapter/
    ├── in/             # Controllers (inbound adapters)
    └── out/            # Presenters, repositories (outbound adapters)
```

**For Cross-Cutting Infrastructure:**

```typescript
NEW REPOSITORY IMPLEMENTATION → shared/adapter/out/repository/
NEW CONFIGURATION CLASS     → shared/io/springboot/config/
NEW FRAMEWORK SETUP        → shared/io/springboot/ or shared/io/standard/
```

## Pattern Implementation Checklist

### When Creating a New Feature

#### Step 1: Domain Layer (if new entities needed)

- [ ] Entity has identity field (implements Entity<Id>)
- [ ] Value Object is immutable (use `readonly`)
- [ ] Value Object has static factory `of()` method
- [ ] Validation in value object constructor
- [ ] Domain logic encapsulated in entities
- [ ] No framework dependencies in domain layer

#### Step 2: Input Port (Use Case Interface)

- [ ] Extends `Command<INPUT, OUTPUT>` for write operations
- [ ] Extends `Query<INPUT, OUTPUT>` for read operations
- [ ] Located in `usecase/port/in/`
- [ ] Interface name ends with `UseCase`
- [ ] Verb + noun naming
- [ ] Single responsibility
- [ ] Public fields only (no behavior)
- [ ] No behavior, just data

#### Step 3: Input Class (if custom input needed)

- [ ] Located in same package as interface
- [ ] Ends with `Input`
- [ ] Implements `Input` interface
- [ ] Public fields only
- [ ] No behavior, just data

#### Step 4: Output Class (if custom output needed)

- [ ] Located in same package as interface
- [ ] Ends with `Output` or `CqrsOutput<OUTPUT>`
- [ ] Public fields for data
- [ ] No behavior, just data

#### Step 5: Use Case Service

- [ ] Located in `usecase/service/`
- [ ] Implements corresponding `UseCase` interface
- [ ] Class name ends with `Service`
- [ ] Constructor injection of dependencies
- [ ] Implements `execute()` method
- [ ] Returns `CqrsOutput` or custom `Output`
- [ ] Validates inputs before business logic
- [ ] Uses repository for persistence
- [ ] Handles errors and converts to output

#### Step 6: DTO (if data transfer needed)

- [ ] Located in `usecase/port/`
- [ ] Ends with `Dto`
- [ ] Public fields only
- [ ] No getters/setters
- [ ] No behavior, only data

#### Step 7: PO (if persistence needed)

- [ ] Located in `usecase/port/`
- [ ] Ends with `Po`
- [ ] Database annotations (`@Entity`, `@Table`, `@Id`, `@Column`)
- [ ] Default constructor
- [ ] Getters and setters
- [ ] No business logic

#### Step 8: Mapper (if conversion needed)

- [ ] Located in `usecase/port/`
- [ ] Ends with `Mapper`
- [ ] Static methods only
- [ ] Implements `toDto()`, `toDomain()`, `toPo()`
- [ ] Supports single and collection: `toDto(List<T>)`

#### Step 9: Output Port (Presenter Interface)

- [ ] Located in `usecase/port/out/`
- [ ] Ends with `Presenter`
- [ ] Single method: `present(Dto data)`
- [ ] No implementation details

#### Step 10: Presenter Implementation

- [ ] Located in `adapter/out/presenter/`
- [ ] Ends with `XxxConsolePresenter` or `XxxWebPresenter`
- [ ] Implements Presenter interface
- [ ] Constructor injection of output medium
- [ ] Formats and outputs data
- [ ] No business logic

#### Step 11: Controller

- [ ] Located in `adapter/in/controller/console/` or `web/`
- [ ] Ends with `XxxConsoleController` or `XxxWebController`
- [ ] Implements appropriate controller interface
- [ ] Constructor injection of use cases
- [ ] Implements `execute()` method
- [ ] Creates input objects from request
- [ ] Calls use case
- [ ] Returns response object
- [ ] No business logic

#### Step 12: Dependency Configuration

- [ ] Add bean method in appropriate config class
- [ ] `RepositoryInjection` for repositories
- [ ] `UseCaseInjection` for use cases
- [ ] Constructor injection pattern
- [ ] Inject interfaces, not implementations

#### Step 13: Registration (if console command)

- [ ] Register controller in `ConsoleControllerExecutor`
- [ ] Map command string to controller
- [ ] Test command execution

## Naming Conventions - STRICT ENFORCEMENT

### Type Naming

```typescript
Entity:              Project, Task, ToDoList
Value Object:         ProjectName, TaskId, ToDoListId
Read-Only:            ReadOnlyProject, ReadOnlyTask
Use Case Interface:   AddTaskUseCase, ShowUseCase, SetDoneUseCase
Use Case Service:     AddTaskService, ShowService, SetDoneService
Controller:           AddConsoleController, ShowConsoleController
Presenter Interface:    ShowPresenter, HelpPresenter
Presenter Impl:       ShowConsolePresenter, HelpWebPresenter
DTO:                  TaskDto, ProjectDto, ToDoListDto
PO:                    TaskPo, ProjectPo, ToDoListPo
Mapper:                TaskMapper, ProjectMapper, ToDoListMapper
Input:                AddTaskInput, ShowInput, SetDoneInput
Output:               ShowOutput, TodayOutput, HelpOutput
```

### Method Naming

```typescript
Getters:              getName(), getId(), getDescription()
Boolean Getters:       isDone(), isActive()
Setters:              setName(), setDone() (entities only)
Actions:              addProject(), setDone(), deleteTask()
Queries:              getProjects(), containTask(), findXxx()
Factory:              of()
Mappers:              toDto(), toDomain(), toPo()
```

### Field Naming

```typescript
Private:              private String name;
Final:                private final TaskId id;
Descriptive:          private List<Project> projects;
```

## Code Style Rules - MANDATORY

### Immutability

- Value objects MUST use `readonly` keyword
- Value objects MUST be immutable
- Entity identity fields MUST be final
- Return defensive copies or unmodifiable collections

### Visibility

- Fields: private by default
- Methods: public for API, private for internal
- Classes: public for API, package-private for internal

### Constructor Injection

```typescript
private readonly dependency: Dependency;

constructor(dependency: Dependency) {
  this.dependency = dependency;
}
```

### Factory Methods

```typescript
public static ProjectName of(name: string): ProjectName {
  return new ProjectName(name);
}
```

### Stream API Usage

```typescript
// Preferred
projects.stream()
  .filter(p => p.getName().equals(name))
  .map(ProjectMapper::toDto)
  .toList();

// Avoid old-style loops
```

### Optional Usage

```typescript
public getTask(id: TaskId): Optional<Task> {
  return tasks.stream()
    .filter(t => t.getId().equals(id))
    .findFirst();
}
```

### Error Handling

```typescript
// Use Result<T> or Output<T> for consistent response
return Output.create()
  .fail()
  .setMessage(`Could not find project '${name}'`);

// Domain throws exceptions
if (invalid) {
  throw new RuntimeException("Invalid state");
}
```

## Testing Requirements - MANDATORY

### Test Location

```typescript
src/test/.../tasks/feature-name/domain/XxxTest.ts
src/test/.../tasks/feature-name/usecase/XxxUseCaseTest.ts
```

### Test Structure

```typescript
@Test
public descriptive_method_name_with_underscores() {
  // Arrange
  const repository = new InMemoryRepository(...);

  // Act
  const result = useCase.execute(input);

  // Assert
  assertEquals(ExitCode.SUCCESS, result.getExitCode());
}
```

### Test Naming

- Describe behavior, not implementation
- Use underscores for readability
- One test per scenario
- Test case description in name

## Validation Rules - MANDATORY

### Value Objects

- Validate in constructor/compact constructor
- Throw exceptions for invalid values

### Use Cases

- Validate inputs before business logic
- Check for nulls
- Use contract libraries: `require()`, `reject()`
- Return appropriate error messages

### Domain Entities

- Enforce invariants in methods
- Throw exceptions for invalid state
- Validate state changes

## Prohibited Actions - NEVER DO THESE

1. **NEVER** skip layers (Controller → Repository directly)
2. **NEVER** depend on concrete classes outside your layer
3. **NEVER** use mutable value objects
4. **NEVER** expose entity internals via getters
5. **NEVER** put business logic in controllers
6. **NEVER** put business logic in presenters
7. **NEVER** use `null` returns (use `Optional`)
8. **NEVER** use getters on entities to modify state
9. **NEVER** create generic exceptions (use specific types)
10. **NEVER** use hardcoded values (use configuration)
11. **NEVER** create direct dependencies between features
12. **NEVER** import domain classes from another feature
13. **NEVER** share use case implementations between features
14. **NEVER** skip creating feature folders for new business capabilities
15. **NEVER** place feature-specific code in `shared/` folder

## Feature Isolation - MANDATORY

- **NEVER** create direct dependencies between features (e.g., `task/ → project/`)
- **ALWAYS** communicate through aggregate root (e.g., `todolist/` for task/project)
- **NEVER** import domain classes from another feature
- Use shared repository interfaces in `shared/` for cross-feature communication

## Decision Tree for AI Agents

### Question 0: Which feature does this belong to?

- **Project Management** → `project/` feature folder
- **Task Management** → `task/` feature folder
- **ToDoList Operations** → `todolist/` feature folder
- **New Business Capability** → Create new feature folder
- **Infrastructure/Configuration** → `shared/` folder

### Question 1: What type of code am I creating?

- **New Entity/Value Object** → Feature domain/
- **New Business Operation** → Feature usecase/port/in/ + service/
- **New User Interface** → Feature adapter/in/controller/ or web/
- **New Data Transfer** → Feature usecase/port/ (DTO)
- **New Persistence** → Feature adapter/out/repository/ or shared/
- **New Output Format** → Feature adapter/out/presenter/

### Question 2: Does this modify state?

- **Yes** → Extend `Command<INPUT, OUTPUT>`
- **No** → Extend `Query<INPUT, OUTPUT>`

### Question 3: Which layer depends on which?

- **Domain Layer** → No dependencies
- **Use Case Layer** → Depends on Domain Layer + Ports
- **Adapter Layer** → Depends on Ports (interfaces only)
- **Framework Layer** → Depends on Adapters

### Question 4: What naming convention applies?

- **Entity** → Singular noun
- **Value Object** → Identity name + `record`
- **Use Case** → Verb + noun + `UseCase`
- **Service** → Noun + `Service`
- **Controller** → Verb + `ConsoleController`/`WebController`
- **DTO** → Noun + `Dto`
- **PO** → Noun + `Po`
- **Mapper** → Noun + `Mapper`

## Feature Communication Rules

### CORRECT: Feature Dependencies

- ✓ CORRECT: `task/` uses `todolist/` repository interface
- ✓ CORRECT: `todolist/` manages task and project entities

### WRONG: Direct Feature Dependencies

- ✗ WRONG: `task/` directly calls `project/` repository implementation
- ✗ WRONG: `task/` imports Project entity from `project/` feature
- ✗ WRONG: Use case in `task/` depends on ProjectRepository from `project/`
- ✗ WRONG: Direct feature-to-feature method calls

### Shared Infrastructure

- Repository implementations go in `shared/adapter/out/repository/`
- Configuration classes go in `shared/io/springboot/config/`
- Common interfaces (ConsoleController, Request, Response) in `todolist/adapter/`
- Framework setup in `shared/io/springboot/` or `shared/io/standard/`

## Error Recovery

If an AI agent makes a mistake:

1. **Identify Violation** - Which rule/pattern was broken?
2. **Understand Fix** - How should it be implemented correctly?
3. **Reference Documents** - Which section in architecture/methodologies docs applies?
4. **Implement Correction** - Apply fix following all rules
5. **Verify Change** - Run through checklist again
6. **Test Thoroughly** - Ensure fix doesn't break anything else

## Verification Checklist

Before submitting code, verify:

### Architecture

- [ ] Dependencies point inward
- [ ] No layer violations
- [ ] All communication through interfaces
- [ ] No circular dependencies
- [ ] No direct feature-to-feature dependencies

### Patterns

- [ ] Correct pattern applied for use case
- [ ] Proper interface segregation
- [ ] Dependency inversion followed
- [ ] Single responsibility principle applied

### Naming

- [ ] All names follow conventions
- [ ] Descriptive and clear
- [ ] No abbreviations (except common ones)
- [ ] Consistent with existing code

### Code Quality

- [ ] No code duplication
- [ ] Short, focused methods
- [ ] Clear, readable code
- [ ] Proper encapsulation
- [ ] No dead code
- [ ] Self-documenting code

### Testing

- [ ] Tests written for new code
- [ ] Test names describe behavior
- [ ] High coverage achieved
- [ ] Tests isolated and independent

---

**Non-compliance with these guidelines will result in code that does not follow project's architecture and must be corrected.**
