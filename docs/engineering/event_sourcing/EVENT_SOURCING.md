# Event Sourcing

## Overview

Event Sourcing is a pattern where state changes are stored as a sequence of events, rather than storing the current state. Nexus supports Event Sourcing as an **optional** pattern for domains requiring audit trails.

## Core Philosophy

### Store Events, Not State

**Traditional Approach (CRUD)**:
```sql
UPDATE users SET email = 'new@example.com' WHERE id = '123'
-- Current state stored
-- History lost
```

**Event Sourcing**:
```sql
INSERT INTO events (aggregate_id, event_type, payload, created_at)
VALUES ('123', 'EmailUpdated', '{"email": "new@example.com"}', NOW())
-- State change stored as event
-- Complete history maintained
```

### State Reconstruction

Current state is reconstructed by replaying events:

```typescript
// Reconstruct user state from events
const events = await eventStore.getEvents('user-123');

const user = User.reconstruct(events);
// Now contains:
// - Email: new@example.com (from EmailUpdated event)
// - All previous email addresses (from history)
// - When each change occurred (from event timestamps)
```

## Key Concepts

### Events

Immutable records of something that happened.

**Properties**:
- **Immutable**: Cannot be changed
- **Timestamped**: When it occurred
- **Typed**: Specific event class
- **Payload**: Event data
- **Idempotent**: Safe to replay multiple times

**Example**:
```typescript
class UserCreated {
  constructor(
    public readonly aggregateId: string,
    public readonly email: string,
    public readonly name: string,
    public readonly createdAt: Date
  ) {}
}

class EmailUpdated {
  constructor(
    public readonly aggregateId: string,
    public readonly oldEmail: string,
    public readonly newEmail: string,
    public readonly updatedAt: Date
  ) {}
}

class PasswordChanged {
  constructor(
    public readonly aggregateId: string,
    public readonly newPasswordHash: string,
    public readonly changedAt: Date
  ) {}
}
```

### Event Store

Repository for storing and retrieving events.

**Operations**:
- **Append**: Add new event
- **Get**: Retrieve all events for aggregate
- **Subscribe**: Listen to new events
- **Replay**: Reconstruct state from events

**Example**:
```typescript
interface IEventStore {
  append(aggregateId: string, event: IDomainEvent): Promise<void>;
  getEvents(aggregateId: string): Promise<IDomainEvent[]>;
  getEventsAfter(aggregateId: string, version: number): Promise<IDomainEvent[]>;
  subscribe(aggregateId: string, callback: (event: IDomainEvent) => void): Subscription;
}

class SqlEventStore implements IEventStore {
  async append(aggregateId: string, event: IDomainEvent): Promise<void> {
    await this.database.query(
      'INSERT INTO events (aggregate_id, event_type, payload, created_at, version) VALUES (?, ?, ?, ?, ?)',
      [
        aggregateId,
        event.constructor.name,
        JSON.stringify(event),
        new Date(),
        event.version
      ]
    );
  }

  async getEvents(aggregateId: string): Promise<IDomainEvent[]> {
    const results = await this.database.query(
      'SELECT * FROM events WHERE aggregate_id = ? ORDER BY created_at ASC',
      [aggregateId]
    );

    return results.map(row => this.deserializeEvent(row));
  }
}
```

### Aggregate Root

Manages state transitions using events.

**Example**:
```typescript
class User implements Entity<UserId> {
  private events: IDomainEvent[] = [];

  constructor(
    private readonly id: UserId,
    private email: Email,
    private name: string,
    private passwordHash: string,
    private createdAt: Date
  ) {}

  static create(email: Email, name: string, password: string): User {
    const id = UserId.generate();
    const passwordHash = PasswordHash.hash(password);

    // Create initial event
    const event = new UserCreated(
      id.value,
      email.value,
      name,
      passwordHash,
      new Date()
    );

    // Create user with event applied
    const user = new User(id, email, name, passwordHash, event.createdAt);
    user.events.push(event);

    return user;
  }

  updateEmail(newEmail: Email): void {
    const oldEmail = this.email;

    // Change state
    this.email = newEmail;

    // Create event
    const event = new EmailUpdated(
      this.id.value,
      oldEmail.value,
      newEmail.value,
      new Date()
    );

    this.events.push(event);
  }

  changePassword(newPassword: string): void {
    const newPasswordHash = PasswordHash.hash(newPassword);

    // Change state
    this.passwordHash = newPasswordHash;

    // Create event
    const event = new PasswordChanged(
      this.id.value,
      newPasswordHash,
      new Date()
    );

    this.events.push(event);
  }

  getUncommittedEvents(): IDomainEvent[] {
    return [...this.events];
  }

  markEventsAsCommitted(): void {
    this.events = [];
  }

  static reconstruct(events: IDomainEvent[]): User {
    // Start from initial event
    const firstEvent = events[0] as UserCreated;

    const user = new User(
      new UserId(firstEvent.aggregateId),
      Email.of(firstEvent.email),
      firstEvent.name,
      firstEvent.newPasswordHash,
      firstEvent.createdAt
    );

    // Replay all subsequent events
    for (let i = 1; i < events.length; i++) {
      const event = events[i];

      if (event instanceof EmailUpdated) {
        user.email = Email.of(event.newEmail);
      } else if (event instanceof PasswordChanged) {
        user.passwordHash = event.newPasswordHash;
      }
    }

    return user;
  }
}
```

## Event Sourcing Benefits

### Complete Audit Trail

```typescript
// Get complete history of user
const events = await eventStore.getEvents('user-123');

events.forEach(event => {
  console.log(`${event.constructor.name} at ${event.createdAt}:`, event.payload);
});

// Output:
// UserCreated at 2024-01-01 10:00:00: { email: 'old@example.com', name: 'John' }
// EmailUpdated at 2024-01-15 14:30:00: { oldEmail: 'old@example.com', newEmail: 'new@example.com' }
// PasswordChanged at 2024-02-01 09:15:00: { newPasswordHash: '...' }
// EmailUpdated at 2024-03-10 11:00:00: { oldEmail: 'new@example.com', newEmail: 'final@example.com' }
```

### Temporal Queries

Query state at any point in time:

```typescript
// Get user state as of Feb 1st, 2024
const allEvents = await eventStore.getEvents('user-123');
const eventsBeforeDate = allEvents.filter(e => e.createdAt < new Date('2024-02-01'));
const userAtDate = User.reconstruct(eventsBeforeDate);

console.log('Email on Feb 1st:', userAtDate.email.value);
// Output: Email on Feb 1st: new@example.com
```

### Event Replay

Replay events to rebuild state:

```typescript
// Replay all events to reconstruct current state
const events = await eventStore.getEvents('user-123');
const currentUser = User.reconstruct(events);

console.log('Current email:', currentUser.email.value);
console.log('Current name:', currentUser.name);
```

### Debugging

Debug issues by examining event sequence:

```typescript
// User reports issue: "Email wasn't updated"
// Debug by checking event sequence
const events = await eventStore.getEvents('user-123');

const emailEvents = events.filter(e => e instanceof EmailUpdated);
console.log('Email change history:');
emailEvents.forEach(e => {
  console.log(`From: ${e.oldEmail} → To: ${e.newEmail} at ${e.updatedAt}`);
});

// Find the issue in event history
```

## Event Sourcing Implementation

### Repository Pattern

```typescript
class UserRepository implements IUserRepository {
  constructor(private readonly eventStore: IEventStore) {}

  async findById(id: UserId): Promise<User | null> {
    const events = await this.eventStore.getEvents(id.value);

    if (events.length === 0) {
      return null;
    }

    return User.reconstruct(events);
  }

  async save(user: User): Promise<void> {
    const events = user.getUncommittedEvents();

    for (const event of events) {
      await this.eventStore.append(user.getId().value, event);
    }

    user.markEventsAsCommitted();
  }

  async delete(id: UserId): Promise<void> {
    // In Event Sourcing, we don't delete
    // Instead, we mark aggregate as deleted
    const deletedEvent = new UserDeleted(id.value, new Date());
    await this.eventStore.append(id.value, deletedEvent);
  }
}
```

### Use Case Integration

```typescript
class UpdateEmailService implements UpdateEmailUseCase {
  constructor(
    private readonly userRepository: IUserRepository,
    private readonly eventStore: IEventStore
  ) {}

  async execute(input: UpdateEmailInput): Promise<UpdateEmailOutput> {
    // Get current state by replaying events
    const user = await this.userRepository.findById(new UserId(input.userId));

    if (!user) {
      return UpdateEmailOutput.failure('User not found');
    }

    // Apply business logic
    const newEmail = Email.of(input.newEmail);
    user.updateEmail(newEmail);

    // Save new events
    await this.userRepository.save(user);

    return UpdateEmailOutput.success(user.getId().value);
  }
}
```

## Event Projections

### Read Models

Create optimized read models from events:

```typescript
// User email projection (for quick email lookups)
class UserEmailProjection {
  constructor(private readonly database: DatabaseConnection) {}

  async apply(event: IDomainEvent): Promise<void> {
    if (event instanceof UserCreated) {
      await this.database.query(
        'INSERT INTO user_emails (user_id, email) VALUES (?, ?)',
        [event.aggregateId, event.email]
      );
    } else if (event instanceof EmailUpdated) {
      await this.database.query(
        'UPDATE user_emails SET email = ? WHERE user_id = ?',
        [event.newEmail, event.aggregateId]
      );
    } else if (event instanceof UserDeleted) {
      await this.database.query(
        'DELETE FROM user_emails WHERE user_id = ?',
        [event.aggregateId]
      );
    }
  }
}
```

### Event Handlers

Listen to events and update projections:

```typescript
class EventHandler {
  constructor(
    private readonly emailProjection: UserEmailProjection,
    private readonly searchIndex: SearchIndex
  ) {}

  async handle(event: IDomainEvent): Promise<void> {
    // Update email projection
    await this.emailProjection.apply(event);

    // Update search index
    await this.searchIndex.update(event);
  }
}
```

## When to Use Event Sourcing

### Ideal Use Cases

✅ **Use Event Sourcing for**:

- Audit requirements (complete history needed)
- Temporal queries (state at any time)
- Event-driven architectures (react to events)
- Complex business rules (domain events)
- Multi-system integration (event sharing)

### Avoid Event Sourcing for:

❌ **Avoid Event Sourcing for**:

- Simple CRUD (no audit needed)
- High write volume (event overhead)
- Schema changes frequently (migration complexity)
- Simple state (no business events)
- Performance-critical reads (need caching)

## Challenges

### Schema Evolution

```typescript
// Handle event schema changes
class UserCreated {
  // V1: email and name only
  // V2: email, name, and phone
  // V3: email, name, phone, and address

  static fromV1(v1Event: any): UserCreated {
    return new UserCreated(
      v1Event.aggregateId,
      v1Event.email,
      v1Event.name,
      null, // phone (V2)
      null, // address (V3)
      v1Event.createdAt
    );
  }

  static fromV2(v2Event: any): UserCreated {
    return new UserCreated(
      v2Event.aggregateId,
      v2Event.email,
      v2Event.name,
      v2Event.phone,
      null, // address (V3)
      v2Event.createdAt
    );
  }
}
```

### Snapshotting

Optimize by creating snapshots:

```typescript
class UserSnapshotStore {
  async saveSnapshot(user: User): Promise<void> {
    await this.database.query(
      'INSERT INTO user_snapshots (aggregate_id, state, version, created_at) VALUES (?, ?, ?, ?)',
      [user.getId().value, JSON.stringify(user), user.getVersion(), new Date()]
    );
  }

  async getSnapshot(id: string): Promise<User | null> {
    const result = await this.database.query(
      'SELECT * FROM user_snapshots WHERE aggregate_id = ? ORDER BY version DESC LIMIT 1',
      [id]
    );

    return result ? JSON.parse(result.state) : null;
  }
}

// Load from snapshot, then replay events after snapshot
const snapshot = await snapshotStore.getSnapshot('user-123');
let user = snapshot;
const events = await eventStore.getEventsAfter('user-123', snapshot.version);
events.forEach(event => user.apply(event));
```

### Event Order

Ensure events are processed in order:

```typescript
class OrderedEventProcessor {
  private queue: Map<string, IDomainEvent[]> = new Map();

  async process(event: IDomainEvent): Promise<void> {
    const aggregateEvents = this.queue.get(event.aggregateId) || [];
    aggregateEvents.push(event);
    this.queue.set(event.aggregateId, aggregateEvents);

    // Process in order
    aggregateEvents.sort((a, b) => a.version - b.version);
    await this.applyEvents(aggregateEvents);
  }
}
```

## Best Practices

### ✅ DO

- Use immutable events
- Version events (optimistic concurrency)
- Create projections for queries
- Use snapshots for performance
- Handle schema evolution
- Document event schemas
- Test event replay
- Handle event failures gracefully

### ❌ DON'T

- Modify events after creation
- Store current state only
- Use Event Sourcing for simple CRUD
- Forget to version events
- Ignore concurrency issues
- Create circular event dependencies
- Store sensitive data in events
- Replay events without testing

## Event Sourcing in Nexus

### Optional Pattern

Event Sourcing is **optional** in Nexus. Use it when:

- Domain requires audit trail
- Temporal queries needed
- Event-driven architecture
- Complex business rules

### Integration with CQRS

Combine Event Sourcing with CQRS:

```typescript
// Write model: Event-sourced aggregate
class User {
  // Events-based state
}

// Read model: Optimized projection
class UserView {
  // Denormalized for queries
}
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/cqrs/CQRS.md](../cqrs/CQRS.md) - CQRS pattern
- [docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md](../domain_driven/DOMAIN_DRIVEN_DESIGN.md) - Domain-Driven Design
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
