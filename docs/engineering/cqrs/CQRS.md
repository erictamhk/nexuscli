# CQRS (Command Query Responsibility Segregation)

## Overview

CQRS is a pattern that separates read and write operations for better scalability and performance. Nexus supports CQRS as an **optional** pattern for complex domains.

## Core Philosophy

### Separate Commands and Queries

**Traditional Approach**:
```typescript
// Same model for reads and writes
class UserRepository {
  async findById(id: string): Promise<User | null> { }
  async save(user: User): Promise<void> { }
  async findAll(): Promise<User[]> { }  // Read
  async findByEmail(email: string): Promise<User | null> { }  // Read
}
```

**CQRS Approach**:
```typescript
// Write Model (Commands)
interface IUserCommandRepository {
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

// Read Model (Queries)
interface IUserQueryRepository {
  findById(id: string): Promise<UserView | null>;
  findByEmail(email: string): Promise<UserView | null>;
  findAll(): Promise<UserView[]>;
  searchByName(name: string): Promise<UserView[]>;
}
```

### Independent Models

Read and write models are optimized for their purposes:

- **Write Model**: Optimized for consistency, business rules
- **Read Model**: Optimized for queries, denormalized data

## Write Model (Commands)

### Commands

Represent operations that modify state:

```typescript
// Command: Represents an action
interface Command<INPUT, OUTPUT> {
  execute(input: INPUT): Promise<OUTPUT>;
}

class CreateUserCommand implements Command<CreateUserInput, CreateUserOutput> {
  constructor(
    private readonly commandRepository: IUserCommandRepository,
    private readonly eventBus: IEventBus
  ) {}

  async execute(input: CreateUserInput): Promise<CreateUserOutput> {
    // Business logic
    const user = User.create(input.email, input.name);

    // Persist to write model
    await this.commandRepository.save(user);

    // Publish event
    await this.eventBus.publish(new UserCreated(user.id.value, user.email.value, user.name));

    return CreateUserOutput.success(user.id.value);
  }
}
```

### Write Repository

Operations for modifying state:

```typescript
interface IUserCommandRepository {
  save(user: User): Promise<void>;
  update(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

class SqlUserCommandRepository implements IUserCommandRepository {
  async save(user: User): Promise<void> {
    await this.database.query(
      'INSERT INTO users (id, email, name, password_hash) VALUES (?, ?, ?, ?)',
      [user.id.value, user.email.value, user.name, user.passwordHash]
    );
  }
}
```

### Write Model Properties

- **Consistency**: Ensures business invariants
- **Validation**: Enforces business rules
- **Transactions**: Maintains data integrity
- **Single Source of Truth**: Only model for writes

## Read Model (Queries)

### Queries

Represent operations that read data:

```typescript
// Query: Represents a data request
interface Query<INPUT, OUTPUT> {
  execute(input: INPUT): Promise<OUTPUT>;
}

class GetUserByIdQuery implements Query<GetUserByIdInput, GetUserByIdOutput> {
  constructor(
    private readonly queryRepository: IUserQueryRepository
  ) {}

  async execute(input: GetUserByIdInput): Promise<GetUserByIdOutput> {
    return await this.queryRepository.findById(input.userId);
  }
}

class SearchUsersQuery implements Query<SearchUsersInput, SearchUsersOutput> {
  constructor(
    private readonly queryRepository: IUserQueryRepository
  ) {}

  async execute(input: SearchUsersInput): Promise<SearchUsersOutput> {
    return await this.queryRepository.searchByName(input.name);
  }
}
```

### Read Repository

Operations optimized for queries:

```typescript
interface IUserQueryRepository {
  findById(id: string): Promise<UserView | null>;
  findByEmail(email: string): Promise<UserView | null>;
  findAll(): Promise<UserView[]>;
  searchByName(name: string): Promise<UserView[]>;
  findByRole(role: string): Promise<UserView[]>;
}

class SqlUserQueryRepository implements IUserQueryRepository {
  async findById(id: string): Promise<UserView | null> {
    const result = await this.database.query(
      `SELECT
        u.id,
        u.email,
        u.name,
        r.name as role_name,
        COUNT(p.id) as post_count
      FROM users u
      LEFT JOIN roles r ON u.role_id = r.id
      LEFT JOIN posts p ON p.user_id = u.id
      WHERE u.id = ?
      GROUP BY u.id`,
      [id]
    );

    return result ? UserViewMapper.fromRow(result) : null;
  }

  async searchByName(name: string): Promise<UserView[]> {
    const results = await this.database.query(
      `SELECT
        u.id,
        u.email,
        u.name,
        r.name as role_name
      FROM users u
      LEFT JOIN roles r ON u.role_id = r.id
      WHERE u.name LIKE ?
      ORDER BY u.name ASC`,
      [`%${name}%`]
    );

    return results.map(UserViewMapper.fromRow);
  }
}
```

### Read Model (View)

Denormalized data optimized for queries:

```typescript
// Read model: Denormalized for queries
class UserView {
  constructor(
    public readonly id: string,
    public readonly email: string,
    public readonly name: string,
    public readonly roleName: string,
    public readonly postCount: number
  ) {}
}
```

### Read Model Properties

- **Performance**: Optimized for queries
- **Denormalization**: Reduces joins
- **Caching**: Easy to cache
- **Projections**: Can be from multiple sources

## Event Bus Integration

### Publish Events

Write model publishes events on state changes:

```typescript
class CreateUserCommand implements Command<CreateUserInput, CreateUserOutput> {
  async execute(input: CreateUserInput): Promise<CreateUserOutput> {
    const user = User.create(input.email, input.name);
    await this.commandRepository.save(user);

    // Publish event for read model to update
    await this.eventBus.publish(new UserCreated(
      user.id.value,
      user.email.value,
      user.name,
      user.roleId.value
    ));

    return CreateUserOutput.success(user.id.value);
  }
}
```

### Subscribe to Events

Read model subscribes to events and updates:

```typescript
class UserViewUpdater {
  constructor(
    private readonly queryRepository: IUserQueryRepository,
    private readonly eventBus: IEventBus
  ) {
    this.eventBus.subscribe('UserCreated', this.handleUserCreated);
    this.eventBus.subscribe('UserUpdated', this.handleUserUpdated);
    this.eventBus.subscribe('UserDeleted', this.handleUserDeleted);
  }

  private async handleUserCreated(event: UserCreated): Promise<void> {
    await this.queryRepository.insert(UserView.fromEvent(event));
  }

  private async handleUserUpdated(event: UserUpdated): Promise<void> {
    await this.queryRepository.update(UserView.fromEvent(event));
  }

  private async handleUserDeleted(event: UserDeleted): Promise<void> {
    await this.queryRepository.delete(event.userId);
  }
}
```

## CQRS Benefits

### Independent Scaling

```typescript
// Scale reads independently from writes
const readDatabase = new ReadDatabaseConnection(readReplica1, readReplica2, readReplica3);
const writeDatabase = new WriteDatabaseConnection(primaryDatabase);

// Read model can use read replicas
const queryRepository = new SqlUserQueryRepository(readDatabase);

// Write model uses primary
const commandRepository = new SqlUserCommandRepository(writeDatabase);
```

### Optimized Read Models

```typescript
// Read model: Denormalized, no business logic
const userView = {
  id: 'user-123',
  email: 'user@example.com',
  name: 'John Doe',
  roleName: 'Admin',
  postCount: 42,
  lastLoginAt: '2024-01-15T10:30:00Z'
};

// Query: Fast, no business logic needed
const user = await queryRepository.findById('user-123');
```

### Optimized Write Models

```typescript
// Write model: Only data needed for operations
const user = {
  id: 'user-123',
  email: 'user@example.com',
  name: 'John Doe',
  roleId: 'role-1',
  passwordHash: '$2b$10$...'
};

// Command: Business logic, invariants
user.changeEmail('new@example.com');
await commandRepository.save(user);
```

### Better Separation of Concerns

```typescript
// Write model: Business rules
class User {
  changeEmail(newEmail: Email): void {
    if (!this.canChangeEmail()) {
      throw new Error('Email cannot be changed now');
    }
    this.email = newEmail;
  }
}

// Read model: No business logic
class UserView {
  constructor(
    public readonly id: string,
    public readonly email: string,
    // ... other fields
  ) {}
}
```

## When to Use CQRS

### Ideal Use Cases

✅ **Use CQRS for**:

- **High read volume**: Lots of queries, fewer writes
- **Different read/write needs**: Queries need different data than writes
- **Complex queries**: Joins, aggregations, filtering
- **Multiple data sources**: Read from different databases/cache
- **Event-driven architecture**: Read model updated via events
- **Independent scaling**: Read and write scale differently

### Avoid CQRS for

❌ **Avoid CQRS for**:

- **Simple CRUD**: Basic operations don't benefit
- **Strong consistency**: Immediate consistency required (eventual not OK)
- **Low complexity**: Simple domain doesn't need separation
- **Small team**: Overhead not justified
- **Same model for reads and writes**: No optimization benefit

## CQRS + Event Sourcing

### Combination Benefits

```typescript
// Write model: Event-sourced
class User {
  // Events-based state
}

// Read model: Projections from events
class UserView {
  // Denormalized from events
}

// Flow:
// Command → Write Model → Events → Read Model
```

### Example

```typescript
// 1. Command: Create user
const createUser = new CreateUserCommand(commandRepository, eventBus);
await createUser.execute({ email: 'user@example.com', name: 'John' });

// 2. Write Model: Persists events
await eventStore.append('user-123', new UserCreated('user-123', 'user@example.com', 'John'));

// 3. Event Bus: Publishes event
await eventBus.publish(new UserCreated('user-123', 'user@example.com', 'John'));

// 4. Read Model: Updates projection
await readRepository.insert(UserView.fromEvent(event));

// 5. Query: Read from optimized projection
const user = await queryRepository.findById('user-123');
```

## Eventual Consistency

### Handling Updates

Read model updates asynchronously:

```typescript
class CreateUserCommand implements Command<CreateUserInput, CreateUserOutput> {
  async execute(input: CreateUserInput): Promise<CreateUserOutput> {
    const user = User.create(input.email, input.name);
    await this.commandRepository.save(user);

    // Publish event (async)
    await this.eventBus.publish(new UserCreated(...));

    // Read model will update asynchronously
    // Eventual consistency, not immediate

    return CreateUserOutput.success(user.id.value);
  }
}
```

### Stale Data Handling

```typescript
// Read model may be slightly stale
const user = await queryRepository.findById('user-123');

// Check staleness if needed
const age = Date.now() - user.updatedAt.getTime();
if (age > 60000) { // > 1 minute old
  // Trigger refresh
  await this.eventBus.publish(new RefreshUserView('user-123'));
}
```

## Implementation Example

### Write Model

```typescript
// Domain entity (write model)
class User implements Entity<UserId> {
  constructor(
    private readonly id: UserId,
    private email: Email,
    private name: string,
    private roleId: string,
    private passwordHash: string
  ) {}

  changeEmail(newEmail: Email): void {
    this.email = newEmail;
  }

  changeRole(newRoleId: string): void {
    this.roleId = newRoleId;
  }
}

// Write repository
interface IUserCommandRepository {
  save(user: User): Promise<void>;
  update(user: User): Promise<void>;
  delete(id: UserId): Promise<void>;
}

class SqlUserCommandRepository implements IUserCommandRepository {
  async save(user: User): Promise<void> {
    await this.database.query(
      'INSERT INTO users (id, email, name, role_id, password_hash) VALUES (?, ?, ?, ?, ?)',
      [user.id.value, user.email.value, user.name, user.roleId, user.passwordHash]
    );
  }
}
```

### Read Model

```typescript
// Read model (projection)
class UserView {
  constructor(
    public readonly id: string,
    public readonly email: string,
    public readonly name: string,
    public readonly roleName: string,
    public readonly postCount: number,
    public readonly lastLoginAt: Date
  ) {}

  static fromRow(row: any): UserView {
    return new UserView(
      row.id,
      row.email,
      row.name,
      row.role_name,
      row.post_count,
      row.last_login_at
    );
  }
}

// Read repository
interface IUserQueryRepository {
  findById(id: string): Promise<UserView | null>;
  findByEmail(email: string): Promise<UserView | null>;
  findAll(): Promise<UserView[]>;
  searchByName(name: string): Promise<UserView[]>;
}

class SqlUserQueryRepository implements IUserQueryRepository {
  async findById(id: string): Promise<UserView | null> {
    const result = await this.database.query(
      `SELECT
        u.id,
        u.email,
        u.name,
        r.name as role_name,
        COUNT(p.id) as post_count,
        MAX(l.last_login_at) as last_login_at
      FROM users u
      LEFT JOIN roles r ON u.role_id = r.id
      LEFT JOIN posts p ON p.user_id = u.id
      LEFT JOIN logins l ON l.user_id = u.id
      WHERE u.id = ?
      GROUP BY u.id`,
      [id]
    );

    return result ? UserView.fromRow(result) : null;
  }
}
```

## Best Practices

### ✅ DO

- Separate read and write models
- Optimize read models for queries
- Use event bus for synchronization
- Handle eventual consistency
- Use projections for read models
- Scale reads independently from writes
- Document consistency requirements

### ❌ DON'T

- Mix read and write operations
- Use business logic in read models
- Assume immediate consistency
- Ignore event ordering
- Overuse CQRS for simple cases
- Forget to update read models
- Use same model for reads and writes

## CQRS in Nexus

### Optional Pattern

CQRS is **optional** in Nexus. Use it when:

- High read/write volume
- Complex query requirements
- Need for independent scaling
- Event-driven architecture

### Integration

CQRS integrates with:
- **Event Sourcing**: Write model event-sourced
- **Event Bus**: Synchronizes read and write models
- **Projections**: Read models built from events
- **Caching**: Optimizes read models

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/event_sourcing/EVENT_SOURCING.md](../event_sourcing/EVENT_SOURCING.md) - Event Sourcing
- [docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md](../clean_architecture/CLEAN_ARCHITECTURE.md) - Clean Architecture
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
