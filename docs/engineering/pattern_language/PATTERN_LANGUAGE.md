# Pattern Language

## Overview

A Pattern Language is a collection of design patterns and conventions that work together coherently in a specific context. Nexus establishes a consistent pattern language across the codebase for maintainability and predictability.

## What Is a Pattern Language?

### Definition

A pattern language is a structured method of describing good design practices within a field of expertise. It consists of:

- **Patterns**: Reusable solutions to common problems
- **Context**: When and where to apply each pattern
- **Consequences**: Results and trade-offs of using the pattern
- **Related Patterns**: How patterns connect to each other

### Purpose

- **Consistency**: Same patterns across codebase
- **Predictability**: Know what to expect
- **Efficiency**: Reuse proven solutions
- **Communication**: Shared vocabulary for the team

## Nexus Pattern Language

### Entity Pattern

**Context**: Need to represent objects with identity.

**Problem**: Objects need unique identity and lifecycle.

**Solution**: Create Entity class with identity field.

**Example**:
```typescript
class Post implements Entity<PostId> {
  constructor(
    private readonly id: PostId,
    private title: PostTitle,
    private content: PostContent
  ) {}

  equals(other: Post): boolean {
    return this.id.equals(other.id);
  }
}
```

**When to Use**: Objects with lifecycle and identity.

**Related Patterns**: Value Object, Aggregate Root.

---

### Value Object Pattern

**Context**: Need to represent concept by attributes, not identity.

**Problem**: Primitive types don't capture domain meaning.

**Solution**: Create immutable value object with validation.

**Example**:
```typescript
class Email {
  private constructor(public readonly value: string) {
    if (!this.isValid(value)) {
      throw new Error('Invalid email format');
    }
  }

  static of(value: string): Email {
    return new Email(value);
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }
}
```

**When to Use**: Concepts without identity, defined by attributes.

**Related Patterns**: Entity, Factory.

---

### Factory Pattern

**Context**: Need to create complex objects with validation.

**Problem**: Direct creation is error-prone and complex.

**Solution**: Use static factory method for object creation.

**Example**:
```typescript
class User {
  private constructor(
    private readonly id: UserId,
    private readonly email: Email,
    private readonly name: string
  ) {}

  static create(email: string, name: string, password: string): User {
    const user = new User(
      UserId.generate(),
      Email.of(email),
      name
    );

    user.setPassword(password);

    return user;
  }
}
```

**When to Use**: Complex object creation with validation.

**Related Patterns**: Builder, Value Object.

---

### Repository Pattern

**Context**: Need to abstract data access.

**Problem**: Business logic depends on database implementation.

**Solution**: Create repository interface and implementations.

**Example**:
```typescript
interface IPostRepository {
  findById(id: PostId): Promise<Post | null>;
  save(post: Post): Promise<void>;
  delete(id: PostId): Promise<void>;
}

class InMemoryPostRepository implements IPostRepository {
  private posts: Map<string, Post> = new Map();

  async findById(id: PostId): Promise<Post | null> {
    return this.posts.get(id.value) || null;
  }

  async save(post: Post): Promise<void> {
    this.posts.set(post.id.value, post);
  }
}
```

**When to Use**: Need data access abstraction.

**Related Patterns**: Dependency Injection, Interface Segregation.

---

### Use Case Pattern

**Context**: Need to implement business operation.

**Problem**: Business logic scattered across multiple classes.

**Solution**: Create use case class for single business operation.

**Example**:
```typescript
interface PublishPostUseCase extends Command<PublishPostInput, PublishPostOutput> {}

class PublishPostService implements PublishPostUseCase {
  constructor(
    private readonly postRepository: IPostRepository,
    private readonly eventBus: IEventBus
  ) {}

  async execute(input: PublishPostInput): Promise<PublishPostOutput> {
    const post = await this.postRepository.findById(new PostId(input.postId));

    if (!post) {
      return PublishPostOutput.failure('Post not found');
    }

    post.publish();

    await this.postRepository.save(post);
    await this.eventBus.publish(new PostPublished(post.id.value, new Date()));

    return PublishPostOutput.success(post.id.value);
  }
}
```

**When to Use**: Implementing business operations.

**Related Patterns**: Repository, Dependency Injection.

---

### Mapper Pattern

**Context**: Need to convert between different models.

**Problem**: Manual conversion is error-prone and repetitive.

**Solution**: Create mapper with conversion methods.

**Example**:
```typescript
class UserMapper {
  static toDto(user: User): UserDto {
    return {
      id: user.id.value,
      email: user.email.value,
      name: user.name
    };
  }

  static toDomain(po: UserPo): User {
    return new User(
      UserId.of(po.id),
      Email.of(po.email),
      po.name
    );
  }

  static toPo(user: User): UserPo {
    const po = new UserPo();
    po.id = user.id.value;
    po.email = user.email.value;
    po.name = user.name;
    return po;
  }

  static toDtoList(users: User[]): UserDto[] {
    return users.map(UserMapper.toDto);
  }
}
```

**When to Use**: Converting between layers (domain ↔ DTO ↔ PO).

**Related Patterns**: Factory, Value Object.

---

### Strategy Pattern

**Context**: Need pluggable algorithms.

**Problem**: Multiple implementations of same algorithm.

**Solution**: Create interface and multiple implementations.

**Example**:
```typescript
interface IPasswordHasher {
  hash(password: string): Promise<string>;
  verify(password: string, hash: string): Promise<boolean>;
}

class BcryptPasswordHasher implements IPasswordHasher {
  async hash(password: string): Promise<string> {
    return await bcrypt.hash(password, 10);
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return await bcrypt.compare(password, hash);
  }
}

class Argon2PasswordHasher implements IPasswordHasher {
  async hash(password: string): Promise<string> {
    return await argon2.hash(password);
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return await argon2.verify(hash, password);
  }
}

// Usage
class UserService {
  constructor(private readonly passwordHasher: IPasswordHasher) {}

  async createUser(password: string): Promise<User> {
    const hash = await this.passwordHasher.hash(password);
    // ...
  }
}
```

**When to Use**: Pluggable algorithms, different implementations.

**Related Patterns**: Dependency Injection, Factory.

---

### Observer Pattern

**Context**: Need to react to state changes.

**Problem**: Objects need to be notified of changes.

**Solution**: Create event bus or observable pattern.

**Example**:
```typescript
interface IEventBus {
  publish<T extends IDomainEvent>(event: T): Promise<void>;
  subscribe<T extends IDomainEvent>(
    eventType: string,
    handler: (event: T) => void
  ): Subscription;
}

class InMemoryEventBus implements IEventBus {
  private subscribers: Map<string, Array<(event: any) => void>> = new Map();

  async publish<T extends IDomainEvent>(event: T): Promise<void> {
    const handlers = this.subscribers.get(event.constructor.name) || [];

    for (const handler of handlers) {
      await handler(event);
    }
  }

  subscribe<T extends IDomainEvent>(
    eventType: string,
    handler: (event: T) => void
  ): Subscription {
    if (!this.subscribers.has(eventType)) {
      this.subscribers.set(eventType, []);
    }

    this.subscribers.get(eventType)!.push(handler);

    return {
      unsubscribe: () => {
        const handlers = this.subscribers.get(eventType)!;
        const index = handlers.indexOf(handler);
        if (index > -1) {
          handlers.splice(index, 1);
        }
      }
    };
  }
}
```

**When to Use**: Event-driven architecture, decoupled components.

**Related Patterns**: Event Sourcing, CQRS.

---

### Builder Pattern

**Context**: Need to construct complex objects.

**Problem**: Constructor has many parameters.

**Solution**: Use builder with fluent API.

**Example**:
```typescript
class UserBuilder {
  private id?: UserId;
  private email?: Email;
  private name?: string;
  private role?: string;

  withId(id: UserId): UserBuilder {
    this.id = id;
    return this;
  }

  withEmail(email: string): UserBuilder {
    this.email = Email.of(email);
    return this;
  }

  withName(name: string): UserBuilder {
    this.name = name;
    return this;
  }

  withRole(role: string): UserBuilder {
    this.role = role;
    return this;
  }

  build(): User {
    if (!this.id || !this.email || !this.name) {
      throw new Error('Required fields missing');
    }

    return new User(
      this.id,
      this.email,
      this.name,
      this.role || 'USER'
    );
  }
}

// Usage
const user = new UserBuilder()
  .withId(UserId.generate())
  .withEmail('user@example.com')
  .withName('John Doe')
  .withRole('ADMIN')
  .build();
```

**When to Use**: Complex object construction with optional parameters.

**Related Patterns**: Factory, Fluent Interface.

---

### Null Object Pattern

**Context**: Need to handle null values gracefully.

**Problem**: Null checks throughout code.

**Solution**: Return null object instead of null.

**Example**:
```typescript
class User {
  static readonly NULL = new User(UserId.null(), Email.null(), 'Null User');

  constructor(
    private readonly id: UserId,
    private readonly email: Email,
    private readonly name: string
  ) {}

  isNull(): boolean {
    return this === User.NULL;
  }
}

class UserRepository {
  async findById(id: string): Promise<User> {
    const user = await this.database.query('SELECT * FROM users WHERE id = ?', [id]);

    if (!user) {
      return User.NULL; // Return null object
    }

    return new User(UserId.of(user.id), Email.of(user.email), user.name);
  }
}

// Usage
const user = await repository.findById('user-123');

if (user.isNull()) {
  console.log('User not found');
} else {
  console.log(`Found: ${user.name}`);
}

// No null checks needed
console.log(`Email: ${user.email.value}`);
```

**When to Use**: Avoid null checks, provide default behavior.

**Related Patterns**: Optional, Strategy.

---

### Optional Pattern

**Context**: Need to handle potentially missing values.

**Problem**: Null checks, undefined values.

**Solution**: Use Optional<T> wrapper.

**Example**:
```typescript
class Optional<T> {
  private constructor(private readonly value: T | null) {}

  static of<T>(value: T | null): Optional<T> {
    return new Optional(value);
  }

  static empty<T>(): Optional<T> {
    return new Optional(null);
  }

  isPresent(): boolean {
    return this.value !== null;
  }

  get(): T {
    if (this.value === null) {
      throw new Error('No value present');
    }
    return this.value;
  }

  orElse(defaultValue: T): T {
    return this.value !== null ? this.value : defaultValue;
  }

  map<U>(mapper: (value: T) => U): Optional<U> {
    if (this.value === null) {
      return Optional.empty();
    }

    return Optional.of(mapper(this.value));
  }

  filter(predicate: (value: T) => boolean): Optional<T> {
    if (this.value === null || !predicate(this.value)) {
      return Optional.empty();
    }

    return this;
  }
}

// Usage
const user = Optional.of(await repository.findById('user-123'));

const email = user.map(u => u.email).orElse(Email.null());
```

**When to Use**: Handle null/undefined gracefully.

**Related Patterns**: Null Object, Mapper.

---

### Singleton Pattern

**Context**: Need exactly one instance of a class.

**Problem**: Multiple instances cause problems.

**Solution**: Create singleton with private constructor.

**Example**:
```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection;

  private constructor() {
    // Connect to database
  }

  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }

    return DatabaseConnection.instance;
  }

  async query(sql: string, params: any[]): Promise<any> {
    // Execute query
  }
}

// Usage
const db = DatabaseConnection.getInstance();
await db.query('SELECT * FROM users', []);
```

**When to Use**: Shared resources (database, logger, config).

**Related Patterns**: Dependency Injection, Factory.

---

## Pattern Relationships

### Connection Patterns

Patterns work together in codebase:

```
Entity ↔ Value Object
  ↓
Repository ← Use Case ← Factory
  ↓
Mapper ↔ DTO ↔ PO
  ↓
Presenter ← Use Case → Builder
  ↓
Observer (Event Bus)
```

### Example Flow

1. **Factory** creates **Entity** with **Value Objects**
2. **Use Case** uses **Repository** to persist **Entity**
3. **Mapper** converts **Entity** to **PO**
4. **Builder** constructs complex objects
5. **Observer** notifies of changes

## Consistency Conventions

### Naming Conventions

All patterns follow Nexus naming conventions:

- **Factory**: `static of()`, `static create()`
- **Mapper**: `static toDto()`, `static toDomain()`, `static toPo()`
- **Repository**: `findById()`, `save()`, `delete()`
- **Use Case**: `execute()`
- **Builder**: `withXxx()` methods

### Structure Conventions

All patterns follow Nexus structure:

```typescript
// Value Object
class Xxx {
  private constructor(public readonly value: T) {
    // Validation
  }

  static of(value: T): Xxx {
    return new Xxx(value);
  }

  equals(other: Xxx): boolean {
    // Equality based on value
  }
}

// Entity
class Xxx implements Entity<XxxId> {
  constructor(
    private readonly id: XxxId,
    // ... other fields
  ) {}

  equals(other: Xxx): boolean {
    // Equality based on id
  }
}

// Repository
interface IXxxRepository {
  findById(id: XxxId): Promise<Xxx | null>;
  save(xxx: Xxx): Promise<void>;
  delete(id: XxxId): Promise<void>;
}
```

## Benefits

### Consistency

- Same patterns across codebase
- Predictable code structure
- Easy to navigate

### Maintainability

- Proven solutions
- Less code duplication
- Clear intent

### Communication

- Shared vocabulary
- Less explanation needed
- Team alignment

### Onboarding

- New developers learn patterns once
- Apply patterns consistently
- Reduce learning curve

## Best Practices

### ✅ DO

- Use consistent patterns across codebase
- Follow naming conventions
- Document pattern usage
- Choose appropriate pattern for context
- Test pattern implementations
- Review pattern usage

### ❌ DON'T

- Mix patterns inconsistently
- Use patterns in wrong context
- Create anti-patterns
- Ignore pattern relationships
- Over-engineer with unnecessary patterns
- Forget to test

## Pattern Language in Nexus

### Documented Patterns

All patterns documented in:
- [docs/engineering/patterns/PATTERNS.md](../patterns/PATTERNS.md) - All design patterns
- [docs/engineering/conventions/NAMING_CONVENTIONS.md](../conventions/NAMING_CONVENTIONS.md) - Naming conventions

### Enforced Patterns

Patterns enforced by:
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent guidelines
- Code review checklist
- Architecture rules
- Testing requirements

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/patterns/PATTERNS.md](../patterns/PATTERNS.md) - All design patterns
- [docs/engineering/conventions/NAMING_CONVENTIONS.md](../conventions/NAMING_CONVENTIONS.md) - Naming conventions
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
