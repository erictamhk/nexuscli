# Project Constitution

## Design Principles and Patterns

### Architectural Principles

#### 1. Hexagonal Architecture (Ports and Adapters)

The project follows strict hexagonal architecture principles:

- **Inbound Adapters**: Handle external inputs (console controllers, web controllers)
- **Outbound Adapters**: Handle external outputs (presenters, repositories)
- **Core Domain**: Contains business logic (entities, value objects)
- **Dependency Rule**: Dependencies always point inward toward the domain

#### 2. Separation of Concerns

Each layer has a single, well-defined responsibility:

- **Entity Layer**: Business domain modeling and behavior
- **Use Case Layer**: Application business logic orchestration
- **Adapter Layer**: External system integration

### Domain-Driven Design Patterns

#### 3. Entity Pattern

- Entities have identity and lifecycle
- Equality based on identity, not attributes
- Encapsulate business behavior
- Example: `Project`, `Task`, `ToDoList`

#### 4. Value Object Pattern

- Immutable objects identified by their attributes
- No lifecycle or identity
- Use records for immutability
- Static factory methods for creation
- Self-validation in constructors
- Example: `ProjectName`, `TaskId`, `ToDoListId`

#### 5. Aggregate Root Pattern

- One entity serves as entry point to aggregate
- Controls consistency boundaries
- Example: `ToDoList` as aggregate root managing `Project` and `Task`

#### 6. Read-Only Interface Pattern

- Return read-only views from domain methods
- Prevent unauthorized modifications
- Throw `UnsupportedOperationException` on write attempts
- Example: `ReadOnlyProject`, `ReadOnlyTask`

### Application Layer Patterns

#### 7. Use Case (Application Service) Pattern

- Each use case implements a specific business operation
- One class per use case (Single Responsibility Principle)
- Constructor injection for dependencies
- Input/Output interfaces for ports
- Naming: `XxxService` implements `XxxUseCase`

#### 8. CQRS (Command Query Responsibility Segregation)

- Separate interfaces for Commands and Queries
- Commands: Modify state (extend `Command<INPUT, OUTPUT>`)
- Queries: Read state (extend `Query<INPUT, OUTPUT>`)
- Different use cases for different operations

#### 9. Repository Pattern

- Abstract data access behind interface
- Multiple implementations (InMemory, CRUD, etc.)
- Methods: `save()`, `delete()`, `findById()`
- Domain entities as parameters, not DTOs

#### 10. Presenter Pattern

- Separate output formatting from business logic
- Interface defined in output ports
- Implementation in adapters
- Takes DTOs as input, formats for specific output medium

### Data Transfer Patterns

#### 11. DTO (Data Transfer Object) Pattern

- Plain objects for data transfer between layers
- No behavior, only data
- Public fields (simple POJOs)
- Naming: `XxxDto`

#### 12. PO (Persistent Object) Pattern

- Database table representation
- Separate from domain entities
- Framework annotations for persistence
- Used in repository implementations

#### 13. Mapper Pattern

- Static methods for conversions
- Three-way transformation:
  - `toDto()`: Domain → DTO
  - `toDomain()`: PO → Domain
  - `toPo()`: Domain → PO
- Support single and collection conversions

### Creational Patterns

#### 14. Factory Method Pattern

- Static factory methods for object creation
- Naming: `of()` as standard
- Centralized creation logic
- Validation in factory methods

#### 15. Builder Pattern (Fluent API)

- Method chaining for object construction
- Example: `CqrsOutput.create().fail().setMessage(...).succeed()`
- Self-returning methods for fluency

#### 16. Null Object Pattern

- `Input.NullInput` for empty input scenarios
- Avoids null checks throughout codebase

### Structural Patterns

#### 17. Interface Segregation Principle

- Small, focused interfaces
- One method per interface where appropriate
- Clients depend only on methods they use

#### 18. Dependency Inversion Principle

- High-level modules don't depend on low-level modules
- Both depend on abstractions (interfaces)
- Abstractions don't depend on details

### Behavioral Patterns

#### 19. Strategy Pattern

- Multiple implementations of same interface
- Interchangeable at runtime
- Example: Repository implementations

#### 20. Command Pattern

- Encapsulate requests as objects
- Each console command as separate controller
- Executor pattern for command dispatching

#### 21. Template Method Pattern

- Use case interfaces define execution contract
- Implementations provide specific behavior

### Functional Programming Patterns

#### 22. Optional Pattern

- Return `Optional<T>` for potentially missing values
- Use `map()`, `filter()`, `findFirst()` for operations
- Avoid `get()` without checking presence

#### 23. Stream API

- Declarative data transformations
- Method chaining for clarity
- Immutable operations

## Coding Style and Conventions

### Naming Conventions

#### Entities

- Singular nouns: `Project`, `Task`, `ToDoList`
- Business domain terms
- Noun phrase naming

#### Value Objects

- Represent identity: `ProjectName`, `TaskId`, `ToDoListId`
- Use `record` keyword
- Immutable fields

#### Use Cases (Input Ports)

- Suffix `UseCase`: `AddTaskUseCase`, `ShowUseCase`
- Verb + noun naming
- Describe intent, not implementation

#### Services (Use Case Implementations)

- Suffix `Service`: `AddTaskService`, `ShowService`
- Implement corresponding `UseCase` interface

#### Controllers

- Suffix `Controller`: `AddConsoleController`, `ShowConsoleController`
- Prefix indicates medium: `ConsoleController`, `WebController`

#### DTOs

- Suffix `Dto`: `TaskDto`, `ProjectDto`, `ToDoListDto`
- Public fields (no getters/setters)

#### Persistent Objects

- Suffix `Po`: `TaskPo`, `ProjectPo`, `ToDoListPo`
- Framework annotations for persistence

#### Mappers

- Class name: `XxxMapper`
- Methods: `toDto()`, `toDomain()`, `toPo()`
- Support both single and collection: `toDto(List<T>)`

#### Input/Output Objects

- Suffix `Input`, `Output`: `AddTaskInput`, `ShowOutput`
- Public fields
- Implement `Input` interface for Input objects

#### Presenters

- Interface: `XxxPresenter` (in output ports)
- Implementation: `XxxConsolePresenter`, `XxxWebPresenter` (in adapters)
- Method: `present(Dto data)`

### Code Organization

#### Package Structure

```
domain.entity
usecase.port.in
usecase.port.out
usecase.service
adapter.in.controller
adapter.out.presenter
adapter.out.repository
io.framework
```

#### Layer Communication

- Controllers → Use Cases → Repository
- Use Cases → Presenter (output)
- No direct adapter-to-adapter communication
- All communication through ports (interfaces)

### Field and Method Conventions

#### Fields

- Private accessibility by default
- Final for immutable fields
- Descriptive names

#### Methods

- **Getters**: `getField()` or `isField()` for booleans
- **Setters**: `setField()` (only on entities, not value objects)
- **Actions**: Verb + noun: `addProject()`, `setDone()`, `deleteTask()`
- **Queries**: Noun or verb: `getProjects()`, `containTask()`, `findXxx()`

### Validation and Error Handling

#### Validation Strategy

- Validate in domain constructors (value objects)
- Throw exceptions for invalid state
- Validate in use cases before business operations
- Use contract libraries for pre/postconditions

#### Error Handling

- Domain layer throws domain-specific exceptions
- Use case layer catches and converts to output
- Controller layer handles user-facing error messages
- Use specific exception types, not generic

#### Output Pattern

- Use `CqrsOutput` for consistent response structure
- Indicate success/failure with exit codes
- Include error messages
- Support data attachment (ID, DTO, etc.)

### Testing Conventions

#### Test Organization

- Separate test package structure mirroring main
- Unit tests per class
- Integration tests for use cases
- Naming: `XxxTest`

#### Test Structure

- Arrange-Act-Assert pattern
- Descriptive test method names
- Test one thing per test
- Use in-memory repositories for testing

#### Test Naming

- Method names describe behavior: `add_a_project_with_duplicated_name_has_no_effect()`
- Use underscores for readability
- Test case description in name

### Immutability and Encapsulation

#### Immutability Rules

- Value objects always immutable
- Use `record` keyword where possible
- Final fields on entities where appropriate
- Return defensive copies or unmodifiable collections

#### Encapsulation Rules

- Private fields
- Public interface through methods
- No direct field access
- Domain logic encapsulated in entities

### Dependency Management

#### Dependency Injection

- Constructor injection preferred
- Field injection only for framework components
- Inject interfaces, not implementations
- Use configuration classes for wiring

#### Dependency Direction

- Never depend on concrete classes outside your layer
- Depend only on abstractions (interfaces)
- Dependency rule: inward toward domain

### Stream API Usage

#### Stream Conventions

- Use method references where possible: `TaskMapper::toDto`
- Prefer `filter()`, `map()`, `findFirst()` over loops
- Use `stream()` for transformations, `parallelStream()` carefully
- Collect with `toList()` (immutable) or `collect(Collectors.toList())`

### Null Safety

#### Null Handling

- Use `Optional<T>` for potentially missing values
- Avoid returning null
- Use `orElse()`, `orElseThrow()`, `orElseGet()` for default values
- Null checks only at system boundaries

### Static Methods and Classes

#### Static Method Usage

- Factory methods: `ProjectName.of()`, `TaskId.of()`
- Utility methods: mappers, validators
- Pure functions without side effects

## Development Guidelines

### Design by Contract

#### Precondition Checking

- Use contract libraries (e.g., `require()`, `reject()`)
- Validate inputs at use case boundaries
- Describe conditions with messages

#### Postcondition Validation

- Ensure invariants hold after operations
- Domain enforces consistency rules
- Repository validates before persistence

### Single Responsibility Principle

#### Class Responsibility

- One reason to change per class
- Clear, focused purpose
- Delegating to collaborators

#### Method Responsibility

- Do one thing well
- Short, focused methods
- Extract private methods for complex logic

### DRY (Don't Repeat Yourself)

#### Code Reuse

- Extract common patterns
- Use base classes and interfaces
- Shared utility methods
- Mapper pattern for conversions

### KISS (Keep It Simple, Stupid)

#### Simplicity Rules

- Avoid over-engineering
- Solve actual problems, not hypothetical ones
- Use straightforward solutions
- Clear, readable code over clever code

### YAGNI (You Aren't Gonna Need It)

#### Development Approach

- Implement only what's needed now
- Don't build for future requirements
- Refactor when requirements change

### Tell, Don't Ask

#### Object-Oriented Design

- Tell objects what to do, don't ask about state
- Objects manage their own behavior
- Minimize getters exposing internal state

### Encapsulation of Collections

#### Collection Handling

- Return unmodifiable views or copies
- Validate collections on input
- Use stream operations over direct collection access
- Prevent external modification

### Configuration Over Code

#### Externalization

- Configuration in separate files
- Environment-specific settings
- No hardcoded values

### Error Message Guidelines

#### Message Format

- Clear and user-friendly
- Provide context for failure
- Suggest corrective actions where applicable
- Use format strings: `format("Could not find project '%s'", name)`

## Code Quality Standards

### Code Organization

- Logical grouping of related code
- Consistent package structure
- Clear separation between layers
- No circular dependencies

### Code Readability

- Self-documenting code
- Descriptive variable and method names
- Minimal comments (code should explain itself)
- Consistent formatting

### Maintainability

- Easy to locate code
- Clear dependencies
- Testable design
- Low coupling, high cohesion

### Extensibility

- Open for extension, closed for modification
- Interface-based design
- Plugin architecture
- Strategy pattern for variations

## Security Guidelines

### Input Validation

- Validate all inputs
- Sanitize user data
- Validate domain constraints
- Type-safe where possible

### Data Protection

- No logging of sensitive data
- Secure storage of credentials
- Proper error messages (don't leak info)
- Use framework security features

## Performance Considerations

### Efficient Operations

- Lazy evaluation where appropriate
- Use indexes for lookups
- Batch operations when possible
- Cache frequently accessed data

### Memory Management

- Avoid memory leaks
- Proper resource cleanup
- Use appropriate data structures
- Stream for large collections

## Documentation Standards

### Self-Documenting Code

- Meaningful names
- Clear structure
- Obvious intent
- Minimal need for comments

### API Documentation

- Interface contracts
- Input/output specifications
- Error conditions
- Usage examples

## Testing Philosophy

### Test Coverage

- Unit tests for domain logic
- Integration tests for use cases
- End-to-end tests for user scenarios
- Test boundaries and edge cases

### Test Isolation

- Independent tests
- No shared state
- Mock external dependencies
- Fast execution

### Test Quality

- Test behavior, not implementation
- Descriptive test names
- Clear test intent
- Maintainable test code
