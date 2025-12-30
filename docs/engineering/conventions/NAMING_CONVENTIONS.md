# Naming Conventions

## Overview

Consistent naming conventions are critical for code readability and maintainability. Nexus enforces strict naming rules across all layers.

## Entities

### Rules

- **Singular nouns**: `Project`, `Task`, `ToDoList`
- **Business domain terms**: Use ubiquitous language from domain model
- **Noun phrase naming**: Clear business meaning
- **No abbreviations**: Except common ones (ID, URL, API, HTTP, JSON, JWT, SQL)

### Examples

```typescript
// GOOD - Clear, descriptive names
class User { }
class Product { }
class Order { }
class ShoppingCart { }
class PaymentMethod { }

// BAD - Abbreviations, unclear names
class Usr { }
class Prod { }
class Ord { }
class ShopCart { }
class PayMeth { }
```

## Value Objects

### Rules

- **Represent identity**: `ProjectName`, `TaskId`, `ToDoListId`
- **Use `record` keyword**: TypeScript records for immutability
- **Immutable fields**: All fields are `readonly`
- **Noun or noun phrase**: Describe what they represent

### Examples

```typescript
// GOOD - Identity value objects
type ProjectName = {
  readonly value: string;
};

type TaskId = {
  readonly value: string;
};

type UserId = {
  readonly value: string;
};

// GOOD - Value values
type Email = {
  readonly value: string;
};

type Money = {
  readonly amount: number;
  readonly currency: string;
};

type Percentage = {
  readonly value: number;
};

// GOOD - Complex value objects
type Address = {
  readonly street: string;
  readonly city: string;
  readonly state: string;
  readonly zipCode: string;
  readonly country: string;
};
```

## Read-Only Interfaces

### Rules

- **Read-only views**: `ReadOnlyProject`, `ReadOnlyTask`, `ReadOnlyUser`
- **Immutable**: All fields are `readonly`
- **Prefix**: Always start with `ReadOnly`
- **Purpose**: Prevent unauthorized modifications

### Examples

```typescript
// GOOD - Read-only interfaces
type ReadOnlyUser = {
  readonly id: UserId;
  readonly email: Email;
  readonly name: string;
  readonly createdAt: Date;
};

type ReadOnlyProduct = {
  readonly id: ProductId;
  readonly name: string;
  readonly description: string;
  readonly price: Money;
  readonly createdAt: Date;
};
```

## Use Cases (Input Ports)

### Rules

- **Suffix `UseCase`**: `AddTaskUseCase`, `ShowUseCase`, `SetDoneUseCase`
- **Verb + noun naming**: Describe intent, not implementation
- **Clear action**: What business operation is performed?

### Examples

```typescript
// GOOD - Clear use case interfaces
interface AddTaskUseCase extends Command<AddTaskInput, AddTaskOutput> { }
interface ShowUseCase extends Query<ShowInput, ShowOutput> { }
interface SetDoneUseCase extends Command<SetDoneInput, SetDoneOutput> { }
interface DeleteTaskUseCase extends Command<DeleteTaskInput, DeleteTaskOutput> { }

// GOOD - More complex use cases
interface RegisterUserUseCase extends Command<RegisterUserInput, RegisterUserOutput> { }
interface AuthenticateUserUseCase extends Command<AuthenticateUserInput, AuthenticateUserOutput> { }
interface UpdateUserProfileUseCase extends Command<UpdateProfileInput, UpdateProfileOutput> { }
interface ChangePasswordUseCase extends Command<ChangePasswordInput, ChangePasswordOutput> { }

// BAD - Describes implementation, not intent
interface UserCreator { } // Should be CreateUserUseCase
interface TaskAdder { } // Should be AddTaskUseCase
interface ProductUpdater { } // Should be UpdateProductUseCase
```

## Services (Use Case Implementations)

### Rules

- **Suffix `Service`**: `AddTaskService`, `ShowService`, `SetDoneService`
- **Implement interface**: Implements corresponding `UseCase` interface
- **Matches use case name**: `AddTaskService` implements `AddTaskUseCase`

### Examples

```typescript
// GOOD - Service implementations
class AddTaskService implements AddTaskUseCase {
  constructor(
    private taskRepository: ITaskRepository,
    private validator: TaskValidator
  ) {}

  async execute(input: AddTaskInput): Promise<AddTaskOutput> {
    // Implementation
  }
}

class ShowService implements ShowUseCase {
  constructor(
    private taskRepository: ITaskRepository,
    private presenter: ShowPresenter
  ) {}

  async execute(input: ShowInput): Promise<ShowOutput> {
    // Implementation
  }
}
```

## Controllers

### Rules

- **Suffix `Controller`**: `AddConsoleController`, `ShowController`, `WebController`
- **Prefix indicates medium**: `ConsoleController` vs `WebController`
- **Verb + noun**: Describe action
- **Matches use case**: `AddTaskController` uses `AddTaskService`

### Examples

```typescript
// GOOD - Console controllers
class AddConsoleController implements IConsoleController {
  constructor(private service: AddTaskService) { }

  async execute(command: string): Promise<void> {
    // Parse command and call service
  }
}

class ShowConsoleController implements IConsoleController {
  constructor(private service: ShowService) { }

  async execute(command: string): Promise<void> {
    // Parse command and call service
  }
}

// GOOD - Web controllers
class AddWebController {
  constructor(private service: AddTaskService) { }

  async handleRequest(req: Request, res: Response): Promise<void> {
    // Parse HTTP request and call service
  }
}

class ShowWebController {
  constructor(private service: ShowService) { }

  async handleRequest(req: Request, res: Response): Promise<void> {
    // Parse HTTP request and call service
  }
}
```

## DTOs (Data Transfer Objects)

### Rules

- **Suffix `Dto`**: `TaskDto`, `ProjectDto`, `ToDoListDto`
- **Public fields**: No getters/setters (simple objects)
- **Plain data structures**: No behavior, only data
- **Singular or plural**: Depends on context (single item vs collection)

### Examples

```typescript
// GOOD - Simple DTOs
type UserDto = {
  id: string;
  email: string;
  name: string;
  createdAt: string;
};

type TaskDto = {
  id: string;
  description: string;
  done: boolean;
  projectId: string | null;
  createdAt: string;
  updatedAt: string;
};

// GOOD - DTO with nested objects
type OrderDto = {
  id: string;
  userId: string;
  items: OrderItemDto[];
  total: {
    amount: number;
    currency: string;
  };
  status: string;
  createdAt: string;
};

type OrderItemDto = {
  productId: string;
  productName: string;
  quantity: number;
  price: {
    amount: number;
    currency: string;
  };
};
```

## Persistent Objects (POs)

### Rules

- **Suffix `Po`**: `TaskPo`, `ProjectPo`, `ToDoListPo`
- **Framework annotations**: For database persistence
- **Database field names**: Often snake_case in database, camelCase in code
- **No business logic**: Only getters/setters

### Examples

```typescript
// GOOD - Persistent objects with database annotations
@Entity('users')
class UserPo {
  @PrimaryColumn()
  id: string;

  @Column()
  email: string;

  @Column()
  name: string;

  @Column()
  passwordHash: string;

  @CreateDateColumn()
  createdAt: Date;

  // Getters and setters only - no business logic
  getId(): string {
    return this.id;
  }

  setId(id: string): void {
    this.id = id;
  }

  getEmail(): string {
    return this.email;
  }

  setEmail(email: string): void {
    this.email = email;
  }
}

@Entity('tasks')
class TaskPo {
  @PrimaryColumn()
  id: string;

  @Column()
  description: string;

  @Column()
  done: boolean;

  @Column({ nullable: true })
  projectId: string | null;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

## Mappers

### Rules

- **Class name**: `XxxMapper`
- **Methods**: `toDto()`, `toDomain()`, `toPo()`
- **Support collections**: `toDto(List<T>)`
- **Static methods only**: No instance needed

### Examples

```typescript
// GOOD - Mapper with all conversion methods
class UserMapper {
  static toDto(user: User): UserDto {
    return {
      id: user.id.value,
      email: user.email.value,
      name: user.name,
      createdAt: user.createdAt.toISOString()
    };
  }

  static toDomain(po: UserPo): User {
    return new User({
      id: UserId.of(po.id),
      email: Email.of(po.email),
      name: po.name,
      passwordHash: po.passwordHash,
      createdAt: po.createdAt
    });
  }

  static toPo(user: User): UserPo {
    const po = new UserPo();
    po.id = user.id.value;
    po.email = user.email.value;
    po.name = user.name;
    po.passwordHash = user.passwordHash;
    po.createdAt = user.createdAt;
    return po;
  }

  // Collection methods
  static toDtoList(users: User[]): UserDto[] {
    return users.map(UserMapper.toDto);
  }

  static toDomainList(pos: UserPo[]): User[] {
    return pos.map(UserMapper.toDomain);
  }

  static toPoList(users: User[]): UserPo[] {
    return users.map(UserMapper.toPo);
  }
}
```

## Input/Output Objects

### Rules

- **Suffix `Input`, `Output`**: `AddTaskInput`, `ShowOutput`
- **Public fields**: Simple data structures
- **Input interfaces**: Implement `Input` interface for validation
- **Output classes**: Simple data containers

### Examples

```typescript
// GOOD - Input objects
interface Input { } // Marker interface for validation

interface AddTaskInput extends Input {
  description: string;
  projectId?: string;
}

interface ShowInput extends Input {
  taskId?: string;
  projectName?: string;
}

interface RegisterUserInput extends Input {
  email: string;
  password: string;
  name: string;
}

// GOOD - Output objects
interface AddTaskOutput {
  taskId: string;
  success: boolean;
  message?: string;
}

interface ShowOutput {
  tasks: TaskDto[];
  total: number;
}

interface RegisterUserOutput {
  userId: string;
  email: string;
  success: boolean;
  message?: string;
}
```

## Presenters

### Rules

- **Interface**: `XxxPresenter` (in output ports)
- **Implementation**: `XxxConsolePresenter`, `XxxWebPresenter` (in adapters)
- **Method**: `present(Dto data)`
- **Output formatting**: Formats data for specific medium

### Examples

```typescript
// GOOD - Presenter interface (in output ports)
interface ShowPresenter {
  present(data: ShowOutput): void;
}

interface HelpPresenter {
  present(data: HelpOutput): void;
}

// GOOD - Presenter implementations (in adapters)
class ShowConsolePresenter implements ShowPresenter {
  present(data: ShowOutput): void {
    console.log('Tasks:');
    console.log(`Total: ${data.total}`);
    data.tasks.forEach(task => {
      console.log(`  - ${task.description} [${task.done ? '✓' : ' '}]`);
    });
  }
}

class ShowWebPresenter implements ShowPresenter {
  present(data: ShowOutput): void {
    this.response.json({
      success: true,
      tasks: data.tasks,
      total: data.total
    });
  }

  constructor(private response: Response) { }
}
```

## Fields

### Rules

- **Private accessibility** by default
- **Final for immutable fields** (TypeScript `readonly`)
- **Descriptive names**: Clear purpose
- **No abbreviations** (except: ID, URL, API, HTTP, JSON, JWT, SQL)

### Examples

```typescript
// GOOD - Descriptive field names
class User {
  private readonly id: UserId;
  private readonly email: Email;
  private name: string;
  private readonly createdAt: Date;
}

class Task {
  private readonly id: TaskId;
  private description: string;
  private done: boolean;
  private readonly createdAt: Date;
  private updatedAt: Date;
}

class ShoppingCart {
  private readonly items: CartItem[];
  private readonly userId: UserId;
  private coupon: Coupon | null;
}

// BAD - Unclear names
class User {
  private u: UserId; // What is 'u'?
  private em: Email; // What is 'em'?
  private n: string; // What is 'n'?
  private d: Date; // What is 'd'?
}
```

## Methods

### Getters

- **Name**: `getField()` or `isField()` for booleans
- **Purpose**: Get field value
- **Read-only**: Don't allow modification through getters

### Examples

```typescript
// GOOD - Getter methods
class User {
  getName(): string {
    return this.name;
  }

  getEmail(): Email {
    return this.email;
  }

  isActive(): boolean {
    return this.status === 'active';
  }

  isVerified(): boolean {
    return this.verifiedAt !== null;
  }
}
```

### Setters

- **Name**: `setField()` (only on entities, not value objects)
- **Purpose**: Set field value
- **Validate**: Validate before setting

### Examples

```typescript
// GOOD - Setter methods (entities only)
class Task {
  setDescription(description: string): void {
    if (!description || description.trim().length === 0) {
      throw new Error('Description cannot be empty');
    }
    this.description = description;
    this.updatedAt = new Date();
  }

  setDone(done: boolean): void {
    this.done = done;
    this.updatedAt = new Date();
  }
}

// BAD - Setters on value objects (value objects are immutable)
class Email {
  setValue(value: string): void { // WRONG - value objects must be immutable
    this.value = value;
  }
}
```

### Actions

- **Verb + noun**: `addProject()`, `setDone()`, `deleteTask()`
- **Clear intent**: What does this action do?
- **Side effects**: Clear about what changes

### Examples

```typescript
// GOOD - Action methods
class ShoppingCart {
  addItem(item: CartItem): void {
    this.items.push(item);
  }

  removeItem(itemId: string): void {
    this.items = this.items.filter(item => item.id !== itemId);
  }

  clear(): void {
    this.items = [];
  }

  applyCoupon(coupon: Coupon): void {
    if (this.isValidCoupon(coupon)) {
      this.coupon = coupon;
    }
  }

  calculateTotal(): Money {
    let total = new Money(0, 'USD');
    this.items.forEach(item => {
      total = total.add(item.price.multiply(item.quantity));
    });
    if (this.coupon) {
      total = total.applyDiscount(this.coupon.discountPercentage);
    }
    return total;
  }
}
```

### Queries

- **Noun or verb**: `getProjects()`, `containsTask()`, `findXxx()`
- **Boolean checks**: `hasXxx()`, `containsXxx()`
- **Search methods**: `findByXxx()`, `searchXxx()`

### Examples

```typescript
// GOOD - Query methods
class Project {
  getTasks(): Task[] {
    return this.tasks;
  }

  containsTask(taskId: TaskId): boolean {
    return this.tasks.some(task => task.id.equals(taskId));
  }

  findTaskById(taskId: TaskId): Task | null {
    return this.tasks.find(task => task.id.equals(taskId)) || null;
  }

  hasCompletedTasks(): boolean {
    return this.tasks.some(task => task.isDone());
  }

  getCompletedTaskCount(): number {
    return this.tasks.filter(task => task.isDone()).length;
  }
}

class UserRepository {
  findById(id: string): Promise<User | null> {
    return this.database.queryOne('SELECT * FROM users WHERE id = ?', [id]);
  }

  findByEmail(email: string): Promise<User | null> {
    return this.database.queryOne('SELECT * FROM users WHERE email = ?', [email]);
  }

  findAll(): Promise<User[]> {
    return this.database.query('SELECT * FROM users');
  }

  searchByName(name: string): Promise<User[]> {
    return this.database.query('SELECT * FROM users WHERE name LIKE ?', [`%${name}%`]);
  }
}
```

## Factory Methods

- **Name**: `of()` as standard for static factory
- **Purpose**: Create objects with validation
- **Centralized creation**: One place for object creation logic

### Examples

```typescript
// GOOD - Factory methods
class UserId {
  private constructor(public readonly value: string) { }

  static of(value: string): UserId {
    if (!value || value.trim().length === 0) {
      throw new Error('User ID cannot be empty');
    }
    return new UserId(value);
  }
}

class Email {
  private constructor(public readonly value: string) { }

  static of(value: string): Email {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) {
      throw new Error('Invalid email format');
    }
    return new Email(value);
  }
}

class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: string
  ) { }

  static of(amount: number, currency: string): Money {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
    if (!currency || currency.length !== 3) {
      throw new Error('Currency must be 3-letter ISO code');
    }
    return new Money(amount, currency);
  }

  static zero(currency: string): Money {
    return new Money(0, currency);
  }
}
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - Architecture overview
- [docs/engineering/conventions/SOLID_PRINCIPLES.md](SOLID_PRINCIPLES.md) - SOLID principles
- [docs/engineering/patterns/PATTERNS.md](../patterns/PATTERNS.md) - Design patterns
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
