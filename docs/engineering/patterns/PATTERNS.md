# Nexus Design Patterns

## Overview

Nexus implements 23+ proven design patterns consistently across the codebase to ensure maintainable, scalable software.

## Architecture Patterns

### **1. Hexagonal Architecture (Ports and Adapters)**

**Purpose**: Clear separation between business logic and external systems

**Components**:
- **Inbound Adapters**: Controllers (REST, CLI) handle external inputs
- **Outbound Adapters**: Presenters (format output), Repositories (data access)
- **Core Domain**: Business entities and value objects (no dependencies)

**Dependency Rule**: Adapters depend on Ports (interfaces), NOT implementations

---

### **2. Clean Architecture (4 Layers)**

**Purpose**: Strict layering with dependency inversion

**Layers**:
1. **Domain Layer**: Core business logic
2. **Application Layer**: Use cases orchestrate domain logic
3. **Adapter Layer**: External interfaces (controllers, presenters, gateways)
4. **Frameworks Layer**: Concrete implementations (Express, Database)

**Dependency Rule**: Dependencies point **inward only** (from outer to inner)

**Benefits**:
- Testable in isolation (mock ports)
- Swappable implementations
- Technology-agnostic domain logic

---

### **3. Repository Pattern**

**Purpose**: Abstract data access behind interface

**Implementation**:
```typescript
interface IUserRepository {
  findById(id: string): Promise<User>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

class InMemoryUserRepository implements IUserRepository {
  private users: Map<string, User> = new Map();

  async findById(id: string): Promise<User | null> {
    return this.users.get(id);
  }

  async save(user: User): Promise<void> {
    this.users.set(user.id, user);
  }

  async delete(id: string): Promise<void> {
    this.users.delete(id);
  }
}
```

**Benefits**:
- Easy to switch between data stores (InMemory, SQLite, PostgreSQL)
- Domain logic protected from data access changes

---

### **4. Use Case (Application Service) Pattern**

**Purpose**: Encapsulate business operations in single-responsibility classes

**Implementation**:
```typescript
// Interface
interface CreateUserUseCase extends Command<CreateUserInput, CqrsOutput> {
  execute(input: CreateUserInput): Promise<CqrsOutput>;
}

// Service
class CreateUserService implements CreateUserUseCase {
  constructor(
    private readonly userRepository: IUserRepository
  ) {
    this.userRepository = userRepository;
  }

  async execute(input: CreateUserInput): Promise<CqrsOutput> {
    // Validate input
    const user = User.create(input.email, input.password);

    // Persist
    await this.userRepository.save(user);

    // Return success
    return CqrsOutput.create()
      .succeed()
      .setData(user.id)
      .setMessage('User created successfully');
  }
}
```

**Benefits**:
- Single responsibility per use case
- Easy to test (mock dependencies)
- Clear business intent

---

### **5. DTO (Data Transfer Object) Pattern**

**Purpose**: Transfer data between layers without exposing domain entities

**Implementation**:
```typescript
// Plain object for data transfer
export class UserDto {
  constructor(
    public readonly id: string,
    public readonly email: string,
    public readonly roles: string[],
  ) {}
}

// Used for API responses, not for business logic
```

**Benefits**:
- Decouples API structure from domain model
- Easy to version API without breaking domain
- Prevents exposing business rules in API layer

---

### **6. PO (Persistent Object) Pattern**

**Purpose**: Database table representation, separate from domain entities

**Implementation**:
```typescript
// Persistent object for database
import { Entity, Column, PrimaryKey } from '@prisma/client';

@Entity()
export class UserPo {
  @Field()
  id!: string;

  @Field()
  email!: string;

  @Field()
  passwordHash!: string;
}
```

**Benefits**:
- Separate persistence concerns from domain logic
- Framework-optimized (Prisma annotations)
- Database schema version control

---

### **7. Mapper Pattern**

**Purpose**: Convert between Domain, DTO, and PO

**Implementation**:
```typescript
class UserMapper {
  static toDto(entity: User): UserDto {
    return new UserDto(
      entity.id,
      entity.email,
      entity.roles,
    );
  }

  static toDomain(po: UserPo): User {
    return new User(
      UserId.of(po.id),
      EmailAddress.of(po.email),
      PasswordHash.of(po.passwordHash),
    );
  }

  static toPo(entity: User): UserPo {
    return new UserPo(
      po.id,
      po.email,
      po.passwordHash,
    );
  }
}
```

**Benefits**:
- Clean separation of concerns
- Easy to add conversion logic
- Centralized transformation logic

---

## Creational Patterns

### **8. Factory Method Pattern**

**Purpose**: Centralized object creation with validation

**Implementation**:
```typescript
class UserFactory {
  static create(dto: CreateUserDTO): User {
    const email = EmailAddress.of(dto.email);
    const passwordHash = PasswordHash.hash(dto.password);
    const roles = dto.roles || [Role.USER];

    return new User({
      id: UserId.generate(),
      email,
      passwordHash,
      roles,
    });
  }
}
```

**Benefits**:
- Single place for object creation logic
- Validation centralized
- Easy to extend for additional variants

---

### **9. Builder Pattern (Fluent API)**

**Purpose**: Construct complex objects step-by-step

**Implementation**:
```typescript
export class CqrsOutputBuilder {
  private data: any = {};
  private exitCode: number = ExitCode.SUCCESS;
  private message: string = '';
  private attachedData: any = {};

  success(): this {
    this.exitCode = ExitCode.SUCCESS;
    return this;
  }

  fail(): this {
    this.exitCode = ExitCode.FAILURE;
    return this;
  }

  setMessage(message: string): this {
    this.message = message;
    return this;
  }

  setData(key: string, value: any): this {
    this.data[key] = value;
    return this;
  }

  attachData(key: string, value: any): this {
    this.attachedData[key] = value;
    return this;
  }

  build(): CqrsOutput {
    return CqrsOutput.from(this.data);
  }
}
```

**Benefits**:
- Clear, readable API
- Flexible for optional parameters
- Self-returning for fluency

---

### **10. Null Object Pattern**

**Purpose**: Eliminate null checks throughout codebase

**Implementation**:
```typescript
// Input.NullInput for empty scenarios
export class AddProjectInput {
  static readonly NullInput: AddProjectInput = new AddProjectInput();
}

// Usage
if (input === AddProjectInput.NullInput) {
  // Handle empty input scenario
  return;
}
```

**Benefits**:
- No null checks needed
- Clear intent (explicit null checks)
- Consistent null handling

---

## Structural Patterns

### **11. Interface Segregation Principle**

**Purpose**: Small, focused interfaces

**Implementation**:
```typescript
// Reader interface - read-only operations
interface IUserReader {
  findById(id: string): Promise<User | null>;
}

// Writer interface - write operations
interface IUserWriter {
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

// Single combined interface with separate methods
interface IUserRepository extends IUserReader, IUserWriter {}
```

**Benefits**:
- Clients depend only on methods they use
- Smaller, focused interfaces
- Easier to test (mock only what's needed)
- Prevents "interface pollution"

---

### **12. Dependency Inversion Principle**

**Purpose**: High-level modules don't depend on low-level modules

**Implementation**:
```typescript
// Correct - High-level depends on abstraction
class UserService {
  constructor(
    private readonly userRepository: IUserRepository // ✅ Interface
  ) {
    this.userRepository = userRepository;
  }
}

// Wrong - High-level depends on implementation
class UserService {
  constructor(
    private readonly repository: InMemoryUserRepository // ✗ Implementation
  ) {
    this.repository = repository;
  }
}
```

**Benefits**:
- Easier to swap implementations
- Testable high-level code in isolation
- Decoupled from implementation changes

---

### **13. Strategy Pattern**

**Purpose**: Multiple implementations of same interface, interchangeable at runtime

**Implementation**:
```typescript
// Interface
interface IUserRepository {
  findById(id: string): Promise<User | null>;
}

// Strategy 1: In-memory (development)
class InMemoryUserRepository implements IUserRepository {
  private users: Map<string, User> = new Map();

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) || null;
  }
}

// Strategy 2: SQLite (production)
class SQLiteUserRepository implements IUserRepository {
  async findById(id: string): Promise<User | null> {
    const result = await this.db.query(
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
    return result ? UserMapper.toDomain(result[0]) : null;
  }
}

// Strategy 3: PostgreSQL (production)
class PostgreSQLUserRepository implements IUserRepository {
  async findById(id: string): Promise<User | null> {
    const result = await this.pool.query(
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
    return result ? UserMapper.toDomain(result[0]) : null;
  }
}
```

**Benefits**:
- Runtime strategy selection based on environment
- Easy to add new strategies
- Testing with mock implementations
- A/B testing without real dependencies

---

### **14. Command Pattern**

**Purpose**: Encapsulate requests as objects

**Implementation**:
```typescript
// Command interface
interface IConsoleCommand {
  execute(): Promise<Response>;
}

// Specific command implementations
class AddProjectCommand implements IConsoleCommand {
  constructor(
    private readonly addProjectUseCase: AddProjectUseCase
  ) {
    this.addProjectUseCase = addProjectUseCase;
  }

  async execute(): Promise<Response> {
    const input = AddProjectInput.from(process.argv.slice(2));
    const result = await this.addProjectUseCase.execute(input);
    return Response.success(result);
  }
}
```

**Benefits**:
- Encapsulates request logic
- Easy to add new commands
- Consistent command interface

---

### **15. Observer Pattern**

**Purpose**: Multiple objects interested in state changes

**Implementation**:
```typescript
// Subject
class User {
  private observers: Observer[] = [];

  register(observer: Observer): void {
    this.observers.push(observer);
  }

  unregister(observer: Observer): void {
    this.observers = this.observers.filter(o => o !== observer);
  }

  notify(event: DomainEvent): void {
    this.observers.forEach(o => o.onEvent(event));
  }
}

// Observer
class EmailService implements Observer {
  onEvent(event: DomainEvent): void {
    if (event.type === 'UserCreated') {
      this.sendWelcomeEmail(event.userEmail);
    }
  }
}
```

**Benefits**:
- Loose coupling between subjects and observers
- Dynamic subscription management
- Event-driven architecture support

---

## Behavioral Patterns

### **16. Strategy Pattern**

**Purpose**: Pluggable algorithms

**Implementation**:
```typescript
// Strategy interface
interface IEmailValidator {
  validate(email: string): boolean;
}

// Concrete strategies
class SimpleEmailValidator implements IEmailValidator {
  validate(email: string): boolean {
    const parts = email.split('@');
    return parts.length === 2 &&
           parts[1].includes('.') &&
           parts[1].includes('.');
  }
}

class RegexEmailValidator implements IEmailValidator {
  private readonly EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

  validate(email: string): boolean {
    return this.EMAIL_REGEX.test(email.trim());
  }
}
```

**Benefits**:
- Easy to switch validation strategies
- Single-responsibility principle
- Testable in isolation

---

### **17. Command Pattern**

**Purpose**: Encapsulate requests as objects

**Implementation**:
```typescript
// Command
interface ICommand<TInput, TOutput> {
  execute(input: TInput): Promise<TOutput>;
}

// Command handler
class CommandExecutor {
  async execute<TInput, TOutput>(
    command: ICommand<TInput, TOutput>
  ): Promise<TOutput> {
    return await command.execute(input);
  }
}

// Usage
await commandExecutor.execute(new AddProjectCommand());
```

**Benefits**:
- Encapsulated request logic
- Easy to add new commands
- Undo/redo/queue support
- Transaction support

---

### **18. Template Method Pattern**

**Purpose**: Define execution contract for use cases

**Implementation**:
```typescript
// Interface defines execution contract
interface IUseCase<TInput, TOutput> {
  execute(input: TInput): Promise<TOutput>;
}

// Implementation provides specific behavior
class AddUserService implements IUseCase<AddUserInput, UserDto> {
  async execute(input: AddUserInput): Promise<UserDto> {
    // Validate
    const user = User.create(input.email, input.password);

    // Persist
    await this.userRepository.save(user);

    // Return
    return UserMapper.toDto(user);
  }
}
```

**Benefits**:
- Contract-first design
- Easy to test (mock interface)
- Decouples definition from implementation
- Self-documenting contracts

---

## Functional Patterns

### **19. Optional Pattern**

**Purpose**: Return potentially missing values explicitly

**Implementation**:
```typescript
import { Optional } from 'typescript';

interface IUserRepository {
  findById(id: string): Promise<Optional<User>>;
}

class InMemoryUserRepository implements IUserRepository {
  private users: Map<string, User> = new Map();

  findById(id: string): Promise<Optional<User>> {
    return Optional.of(this.users.get(id));
  }
}
```

**Benefits**:
- Explicit about potential null returns
- No silent null failures
- Type-safe null handling
- Forces developer to handle null cases

---

### **20. Stream API**

**Purpose**: Declarative data transformations

**Implementation**:
```typescript
// Traditional loop
const activeUsers: User[] = [];
users.forEach(user => {
  if (user.isActive) {
    activeUsers.push(user);
  }
});

// Stream API
const activeUsers = users.stream()
  .filter(user => user.isActive())
  .collect(Collectors.toList());
```

**Benefits**:
- Declarative and readable
- Easy to chain transformations
- Functional programming benefits
- Efficient (can be parallel with parallelStream())

---

## Pattern Language

Nexus follows consistent idiomatic patterns throughout the codebase:

### **Entity Pattern**
- Singular nouns: `Project`, `Task`, `ToDoList`
- Business domain terms
- Noun phrase naming: Clear business meaning

### **Value Object Pattern**
- Represent identity: `ProjectName`, `TaskId`, `ToDoListId`
- Use TypeScript `record` keyword
- Immutable fields

### **Use Case Pattern**
- Verb + noun: `AddTaskUseCase`, `ShowUseCase`
- Service implementation: `AddTaskService`, `ShowService`

### **Service Pattern**
- Noun + `Service`: `AddTaskService`, `ShowService`
- Implements corresponding `UseCase` interface

### **Controller Pattern**
- Verb + `ConsoleController`, `WebController`
- Prefix indicates medium

### **DTO Pattern**
- Noun + `Dto`: `TaskDto`, `ProjectDto`, `ToDoListDto`
- Public fields only (no getters/setters)

### **Persistent Object Pattern**
- Noun + `Po`: `TaskPo`, `ProjectPo`, `ToDoListPo`
- Framework annotations for persistence

### **Mapper Pattern**
- Class name: `XxxMapper`
- Methods: `toDto()`, `toDomain()`, `toPo()`
- Support single and collection: `toDto(List<T>)`

### **Input/Output Object Pattern**
- Suffix `Input`, `Output`: `AddTaskInput`, `ShowOutput`
- Public fields (no behavior)

### **Presenter Pattern**
- Interface: `XxxPresenter` (in usecase/port/out/)
- Implementation: `XxxConsolePresenter` or `XxxWebPresenter` (in adapters)
- Method: `present(Dto data)`

## See Also

- [Naming Conventions](./engineering/conventions/SOLID_PRINCIPLES.md)
- [Testing Conventions](./engineering/conventions/TESTING_CONVENTIONS.md)
- [Development Guidelines](./engineering/conventions/DEVELOPMENT_GUIDELINES.md)
- [Security Guidelines](./engineering/conventions/SECURITY_GUIDELINES.md)
- [Code Quality Standards](./engineering/quality/CODE_QUALITY_STANDARDS.md)