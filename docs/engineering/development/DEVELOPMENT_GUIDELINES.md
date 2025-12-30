# Development Guidelines

## Overview

Nexus follows comprehensive development guidelines to ensure code quality, maintainability, and team productivity. These guidelines are mandatory for all developers working on the project.

## Design by Contract

### Precondition Checking

Validate inputs at use case boundaries:

```typescript
interface Input {
  validate(): void;
}

interface Command<INPUT, OUTPUT> {
  execute(input: INPUT): Promise<OUTPUT>;
}

class AddTaskService implements Command<AddTaskInput, AddTaskOutput> {
  async execute(input: AddTaskInput): Promise<AddTaskOutput> {
    // Precondition - validate input
    input.validate();

    // Business logic
    const task = Task.create(input.description);
    await this.repository.save(task);

    return AddTaskOutput.success(task.id.value);
  }
}

class AddTaskInput implements Input {
  constructor(
    public readonly description: string,
    public readonly projectId?: string
  ) {}

  validate(): void {
    if (!this.description || this.description.trim().length === 0) {
      throw new Error('Description cannot be empty');
    }
    if (this.description.length > 500) {
      throw new Error('Description cannot exceed 500 characters');
    }
  }
}
```

### Use Contract Libraries

Use `require()` and `reject()` for preconditions:

```typescript
// Precondition library
export function require(condition: boolean, message: string): void {
  if (!condition) {
    throw new Error(message);
  }
}

export function reject(condition: boolean, message: string): void {
  if (condition) {
    throw new Error(message);
  }
}

// Usage
class Task {
  setDescription(description: string): void {
    reject(!description, 'Description cannot be empty');
    reject(description.length > 500, 'Description too long');
    this.description = description;
  }
}
```

### Postcondition Validation

Ensure invariants hold after operations:

```typescript
class ShoppingCart {
  private items: CartItem[] = [];
  private readonly totalLimit: Money;

  addItem(item: CartItem): void {
    // Precondition
    if (!item) {
      throw new Error('Item cannot be null');
    }

    // Action
    this.items.push(item);

    // Postcondition - verify invariant
    const total = this.calculateTotal();
    if (total.greaterThan(this.totalLimit)) {
      // Rollback
      this.items.pop();
      throw new Error('Total would exceed limit');
    }
  }
}
```

### Domain Enforces Consistency Rules

Business rules enforced in domain layer:

```typescript
class Order {
  setStatus(status: OrderStatus): void {
    const currentStatus = this.status;

    // Domain invariant: Status transition rules
    if (currentStatus === OrderStatus.COMPLETED && status !== OrderStatus.COMPLETED) {
      throw new Error('Cannot change status of completed order');
    }

    if (currentStatus === OrderStatus.CANCELLED && status !== OrderStatus.CANCELLED) {
      throw new Error('Cannot change status of cancelled order');
    }

    // Valid transitions
    const validTransitions = {
      [OrderStatus.PENDING]: [OrderStatus.PROCESSING, OrderStatus.CANCELLED],
      [OrderStatus.PROCESSING]: [OrderStatus.SHIPPED, OrderStatus.CANCELLED],
      [OrderStatus.SHIPPED]: [OrderStatus.DELIVERED, OrderStatus.RETURNED]
    };

    const allowedTransitions = validTransitions[currentStatus] || [];
    if (!allowedTransitions.includes(status)) {
      throw new Error(`Invalid status transition: ${currentStatus} → ${status}`);
    }

    this.status = status;
    this.updatedAt = new Date();
  }
}
```

## KISS (Keep It Simple, Stupid)

### Avoid Over-Engineering

Solve actual problems, not hypothetical ones:

```typescript
// BAD - Over-engineered solution
class TaskManager {
  private tasks: Task[];
  private cache: Map<string, Task>;
  private index: Map<string, number>;
  private observers: Observer[];
  private eventBus: EventBus;
  private metrics: MetricsCollector;

  async addTask(task: Task): Promise<void> {
    // Complex logic with cache, index, observers, event bus, metrics
    // ... 100+ lines of code
  }
}

// GOOD - Simple solution
class TaskManager {
  private tasks: Task[] = [];

  addTask(task: Task): void {
    this.tasks.push(task);
  }

  getTasks(): Task[] {
    return [...this.tasks];
  }
}
```

### Straightforward Solutions

Use simple approaches:

```typescript
// BAD - Overly complex
class StringValidator {
  validateEmail(email: string): boolean {
    // Complex regex with multiple conditions
    const pattern = /^(?:(?:[\w`!#$%&'*+\-/=?^{|}~]+\.)*[\w`!#$%&'*+\-/=?^{|}~]+@)(?:\[)(?:IPv6:(?:(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){7})|(?:(?!.*::[a-f0-9])(?::?[a-f0-9]{0,4}){0,7}?[a-f0-9]{1,4}))\]|(?:(?:IPv4:(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)))\]|(?:(?:[a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}))$/;
    return pattern.test(email);
  }
}

// GOOD - Simple and clear
class StringValidator {
  private readonly EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

  validateEmail(email: string): boolean {
    if (!email) return false;
    return this.EMAIL_REGEX.test(email);
  }
}
```

### Clear, Readable Code

Avoid clever, obscure code:

```typescript
// BAD - Clever but hard to understand
function isPalindrome(str: string): boolean {
  return str === str.split('').reverse().join('');
}

// GOOD - Clear and easy to understand
function isPalindrome(str: string): boolean {
  const chars = str.split('');
  const reversed = [...chars].reverse();
  return chars.join('') === reversed.join('');
}
```

## DRY (Don't Repeat Yourself)

### Extract Common Patterns

Remove code duplication:

```typescript
// BAD - Repeated validation logic
class UserValidator {
  validateEmail(email: string): void {
    if (!email) {
      throw new Error('Email cannot be empty');
    }
    if (email.length > 255) {
      throw new Error('Email too long');
    }
  }

  validateName(name: string): void {
    if (!name) {
      throw new Error('Name cannot be empty');
    }
    if (name.length > 100) {
      throw new Error('Name too long');
    }
  }
}

// GOOD - Extracted common pattern
class Validator {
  validateRequired(value: string, fieldName: string): void {
    if (!value || value.trim().length === 0) {
      throw new Error(`${fieldName} cannot be empty`);
    }
  }

  validateMaxLength(value: string, maxLength: number, fieldName: string): void {
    if (value.length > maxLength) {
      throw new Error(`${fieldName} cannot exceed ${maxLength} characters`);
    }
  }
}

class UserValidator {
  private readonly validator = new Validator();

  validateEmail(email: string): void {
    this.validator.validateRequired(email, 'Email');
    this.validator.validateMaxLength(email, 255, 'Email');
  }

  validateName(name: string): void {
    this.validator.validateRequired(name, 'Name');
    this.validator.validateMaxLength(name, 100, 'Name');
  }
}
```

### Use Base Classes and Interfaces

Share functionality:

```typescript
// BAD - Duplicate code in services
class UserService {
  async create(data: UserData): Promise<User> {
    const existing = await this.repository.findByEmail(data.email);
    if (existing) {
      throw new Error('Email already exists');
    }
    const user = new User(data);
    return await this.repository.save(user);
  }
}

class ProductService {
  async create(data: ProductData): Promise<Product> {
    const existing = await this.repository.findBySku(data.sku);
    if (existing) {
      throw new Error('SKU already exists');
    }
    const product = new Product(data);
    return await this.repository.save(product);
  }
}

// GOOD - Shared base class
abstract class BaseCreateService<T> {
  constructor(protected readonly repository: IRepository<T>) {}

  async create(data: any): Promise<T> {
    await this.validateUnique(data);
    const entity = this.createEntity(data);
    return await this.repository.save(entity);
  }

  protected abstract async validateUnique(data: any): Promise<void>;
  protected abstract createEntity(data: any): T;
}

class UserService extends BaseCreateService<User> {
  protected async validateUnique(data: UserData): Promise<void> {
    const existing = await this.repository.findByEmail(data.email);
    if (existing) {
      throw new Error('Email already exists');
    }
  }

  protected createEntity(data: UserData): User {
    return new User(data);
  }
}
```

### Shared Utility Methods

Reusable helper functions:

```typescript
// GOOD - Utility functions
class StringUtils {
  static truncate(str: string, maxLength: number): string {
    return str.length > maxLength ? str.substring(0, maxLength) + '...' : str;
  }

  static capitalize(str: string): string {
    return str.charAt(0).toUpperCase() + str.slice(1);
  }

  static slugify(str: string): string {
    return str
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/^-|-$/g, '');
  }
}

// Usage
const title = StringUtils.capitalize('hello world'); // "Hello world"
const slug = StringUtils.slugify('Hello World'); // "hello-world"
```

## YAGNI (You Aren't Gonna Need It)

### Implement Only What's Needed Now

Don't build for future requirements:

```typescript
// BAD - Implements features that might be needed later
class UserService {
  async createUser(data: UserData): Promise<User> {
    // Features we think we might need
    this.logAuditTrail('user created', data);
    this.sendWelcomeEmail(data.email);
    this.updateUserStatistics(data);
    this.cacheUserInRedis(user);
    this.notifyAnalytics(data);
    // ... actual user creation logic
  }
}

// GOOD - Implements only what's needed now
class UserService {
  async createUser(data: UserData): Promise<User> {
    const user = User.create(data);
    return await this.repository.save(user);
  }
}
```

### Refactor When Requirements Change

Adapt to changing needs:

```typescript
// Start simple
class TaskService {
  async createTask(description: string): Promise<Task> {
    const task = Task.create(description);
    return await this.repository.save(task);
  }
}

// Later, when requirement changes to add due dates
class TaskService {
  async createTask(description: string, dueDate?: Date): Promise<Task> {
    const task = Task.create(description, dueDate);
    return await this.repository.save(task);
  }
}

// Later, when requirement changes to add priorities
class TaskService {
  async createTask(
    description: string,
    dueDate?: Date,
    priority?: Priority
  ): Promise<Task> {
    const task = Task.create(description, dueDate, priority);
    return await this.repository.save(task);
  }
}
```

### Avoid Speculative Features

Don't anticipate unnecessary complexity:

```typescript
// BAD - Speculative feature that might never be used
class TaskService {
  async createTask(description: string): Promise<Task> {
    const task = Task.create(description);

    // Speculative: caching for scalability that might never be needed
    await this.cache.set(task.id.value, task);

    // Speculative: indexing for search that might never be needed
    await this.searchIndex.index(task);

    return await this.repository.save(task);
  }
}

// GOOD - Simple, add complexity when needed
class TaskService {
  async createTask(description: string): Promise<Task> {
    const task = Task.create(description);
    return await this.repository.save(task);
  }
}
```

## Tell, Don't Ask

### Tell Objects What to Do

Objects manage their own behavior:

```typescript
// BAD - Ask object for data, then manipulate externally
class Order {
  getItems(): OrderItem[] {
    return this.items;
  }

  getTotal(): Money {
    return this.total;
  }
}

class OrderService {
  processOrder(order: Order): void {
    // Ask for data, then manipulate externally
    const items = order.getItems();
    items.forEach(item => {
      if (item.getQuantity() > 10) {
        item.setDiscount(0.1); // Manipulate item externally
      }
    });
  }
}

// GOOD - Tell object what to do
class OrderItem {
  applyBulkDiscount(): void {
    if (this.quantity > 10) {
      this.discount = 0.1;
    }
  }
}

class Order {
  applyBulkDiscounts(): void {
    this.items.forEach(item => item.applyBulkDiscount());
  }
}

class OrderService {
  processOrder(order: Order): void {
    order.applyBulkDiscounts();
  }
}
```

### Minimize Getters Exposing Internal State

Encapsulate object state:

```typescript
// BAD - Exposes internal state
class ShoppingCart {
  getItems(): CartItem[] {
    return this.items; // Direct access to internal state
  }

  addItem(item: CartItem): void {
    this.items.push(item);
    this.recalculateTotal();
  }
}

// External code manipulates internal state
const cart = new ShoppingCart();
cart.addItem(item1);
const items = cart.getItems();
items.pop(); // Directly modifies internal state!

// GOOD - Encapsulated state
class ShoppingCart {
  addItem(item: CartItem): void {
    this.items.push(item);
    this.recalculateTotal();
  }

  removeItem(itemId: string): void {
    this.items = this.items.filter(item => item.id !== itemId);
    this.recalculateTotal();
  }

  getItemCount(): number {
    return this.items.length;
  }

  getTotal(): Money {
    return this.total;
  }
}
```

### Command-Oriented

Send commands, don't query state:

```typescript
// BAD - Query state, then make decisions
class OrderService {
  processOrder(order: Order): void {
    if (order.getStatus() === OrderStatus.PENDING) {
      order.setStatus(OrderStatus.PROCESSING);
    }
    if (order.getItems().length > 0) {
      this.paymentService.charge(order.getTotal());
    }
  }
}

// GOOD - Send commands
class OrderService {
  processOrder(order: Order): void {
    order.startProcessing();
    order.chargePayment(this.paymentService);
  }
}
```

## Configuration Over Code

### Configuration in Separate Files

Externalize settings:

```typescript
// config/database.ts
export const databaseConfig = {
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT || '5432'),
  username: process.env.DATABASE_USERNAME || 'user',
  password: process.env.DATABASE_PASSWORD || 'password',
  database: process.env.DATABASE_NAME || 'myapp',
  ssl: process.env.NODE_ENV === 'production'
};

// config/app.ts
export const appConfig = {
  port: parseInt(process.env.PORT || '3000'),
  environment: process.env.NODE_ENV || 'development',
  logLevel: process.env.LOG_LEVEL || 'info'
};

// Usage
class DatabaseConnection {
  constructor() {
    this.connect(databaseConfig);
  }
}
```

### Environment-Specific Settings

Separate dev/staging/prod configs:

```typescript
// config/environments/development.ts
export const developmentConfig = {
  database: {
    host: 'localhost',
    port: 5432,
    ssl: false
  },
  email: {
    provider: 'console', // Log emails to console
    from: 'noreply@localhost'
  },
  cache: {
    provider: 'memory' // Use in-memory cache
  }
};

// config/environments/production.ts
export const productionConfig = {
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT),
    ssl: true
  },
  email: {
    provider: 'ses', // Use AWS SES
    from: process.env.EMAIL_FROM
  },
  cache: {
    provider: 'redis', // Use Redis
    url: process.env.REDIS_URL
  }
};

// config/index.ts
const env = process.env.NODE_ENV || 'development';

export const config = env === 'production'
  ? productionConfig
  : developmentConfig;
```

### No Hardcoded Values

Avoid hardcoding configuration:

```typescript
// BAD - Hardcoded values
class EmailService {
  constructor() {
    this.smtpHost = 'smtp.example.com'; // Hardcoded
    this.smtpPort = 587; // Hardcoded
    this.fromEmail = 'noreply@example.com'; // Hardcoded
  }
}

// GOOD - Externalized configuration
class EmailService {
  constructor(private config: EmailConfig) {
    this.smtpHost = config.smtpHost;
    this.smtpPort = config.smtpPort;
    this.fromEmail = config.fromEmail;
  }
}

// Usage
const emailConfig = {
  smtpHost: process.env.SMTP_HOST,
  smtpPort: parseInt(process.env.SMTP_PORT || '587'),
  fromEmail: process.env.EMAIL_FROM
};

const emailService = new EmailService(emailConfig);
```

## Additional Best Practices

### Fail Fast

Validate early and fail explicitly:

```typescript
// GOOD - Fail fast
class UserService {
  async createUser(data: UserData): Promise<User> {
    // Validate immediately
    if (!data.email) {
      throw new ValidationError('Email is required');
    }
    if (!this.isValidEmail(data.email)) {
      throw new ValidationError('Invalid email format');
    }
    if (!data.password) {
      throw new ValidationError('Password is required');
    }
    if (data.password.length < 8) {
      throw new ValidationError('Password must be at least 8 characters');
    }

    // Continue with business logic
    const user = User.create(data);
    return await this.repository.save(user);
  }
}
```

### Use Meaningful Names

Names should reveal intent:

```typescript
// BAD - Unclear names
const d = new Date();
const u = await repo.get(id);
const p = u.p;

// GOOD - Clear names
const currentDate = new Date();
const user = await userRepository.findById(userId);
const profile = user.profile;
```

### Keep Functions Small

Functions should do one thing:

```typescript
// BAD - Function does too much
function processOrder(order: Order) {
  // Validate
  if (!order.items || order.items.length === 0) {
    throw new Error('Order has no items');
  }
  // Calculate total
  let total = 0;
  order.items.forEach(item => total += item.price * item.quantity);
  // Save
  order.total = total;
  order.status = 'PROCESSED';
  order.processedAt = new Date();
  // Send email
  sendEmail(order.userEmail, 'Order processed', `Your order total: ${total}`);
}

// GOOD - Small, focused functions
function validateOrder(order: Order): void {
  if (!order.items || order.items.length === 0) {
    throw new Error('Order has no items');
  }
}

function calculateOrderTotal(order: Order): Money {
  return order.items.reduce((total, item) =>
    total.add(item.price.multiply(item.quantity)),
    Money.zero()
  );
}

function saveOrder(order: Order): void {
  order.status = 'PROCESSED';
  order.processedAt = new Date();
}

function sendOrderProcessedEmail(order: Order): void {
  sendEmail(
    order.userEmail,
    'Order processed',
    `Your order total: ${order.total}`
  );
}

function processOrder(order: Order): void {
  validateOrder(order);
  const total = calculateOrderTotal(order);
  order.total = total;
  saveOrder(order);
  sendOrderProcessedEmail(order);
}
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - Architecture overview
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/conventions/SOLID_PRINCIPLES.md](SOLID_PRINCIPLES.md) - SOLID principles
- [docs/engineering/conventions/NAMING_CONVENTIONS.md](NAMING_CONVENTIONS.md) - Naming conventions
- [docs/engineering/conventions/TESTING_CONVENTIONS.md](TESTING_CONVENTIONS.md) - Testing conventions
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
