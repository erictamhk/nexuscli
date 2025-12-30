# SOLID Principles

## Overview

SOLID principles are fundamental design principles that make software more maintainable, scalable, and robust. Nexus strictly enforces these principles across all code.

## Single Responsibility Principle (SRP)

### Class Responsibility

- **One reason to change per class**
- Clear, focused purpose
- Delegating to collaborators

**Example:**

```typescript
// BAD: Violates SRP - multiple responsibilities
class UserService {
  createUser(data: UserData): User {
    // Validates input
    // Hashes password
    // Sends email
    // Saves to database
    // Returns user
  }
}

// GOOD: Follows SRP - single responsibility
class UserCreator {
  constructor(
    private validator: UserValidator,
    private passwordHasher: PasswordHasher,
    private repository: UserRepository,
    private emailService: EmailService
  ) {}

  createUser(data: UserData): User {
    this.validator.validate(data);
    const passwordHash = this.passwordHasher.hash(data.password);
    const user = User.create(data.email, passwordHash);
    this.repository.save(user);
    this.emailService.sendWelcomeEmail(user);
    return user;
  }
}
```

### Method Responsibility

- **Do one thing well**
- Short, focused methods
- Extract private methods for complex logic

**Example:**

```typescript
// BAD: Violates SRP - method does too many things
class ReportGenerator {
  generateReport(): string {
    // 1. Fetch data from database
    // 2. Transform data
    // 3. Format as HTML
    // 4. Save to file
    // 5. Send email notification
  }
}

// GOOD: Follows SRP - each method has single responsibility
class ReportGenerator {
  private fetchData(): ReportData {
    return this.repository.fetchReportData();
  }

  private transformData(data: ReportData): TransformedData {
    return this.dataTransformer.transform(data);
  }

  private formatReport(data: TransformedData): string {
    return this.reportFormatter.formatAsHtml(data);
  }

  private saveReport(report: string): void {
    this.fileStorage.save('report.html', report);
  }

  private notifyUser(): void {
    this.emailService.sendNotification('Report generated');
  }

  generateReport(): string {
    const data = this.fetchData();
    const transformed = this.transformData(data);
    const report = this.formatReport(transformed);
    this.saveReport(report);
    this.notifyUser();
    return report;
  }
}
```

## Interface Segregation Principle (ISP)

### Small Interfaces

- Focused, single-purpose interfaces
- One method per interface where appropriate
- Clients depend only on methods they use

**Example:**

```typescript
// BAD: Violates ISP - clients forced to implement methods they don't need
interface UserOperations {
  createUser(user: User): void;
  deleteUser(id: string): void;
  updateUser(user: User): void;
  findById(id: string): User | null;
  findByEmail(email: string): User | null;
  findAll(): User[];
}

class UserReader implements UserOperations {
  // Reader shouldn't need to implement create/update/delete
  createUser(user: User): void {
    throw new Error('Not implemented');
  }
  deleteUser(id: string): void {
    throw new Error('Not implemented');
  }
  updateUser(user: User): void {
    throw new Error('Not implemented');
  }
  // ...
}

// GOOD: Follows ISP - small, focused interfaces
interface IUserReader {
  findById(id: string): User | null;
  findByEmail(email: string): User | null;
  findAll(): User[];
}

interface IUserWriter {
  create(user: User): void;
  update(user: User): void;
  delete(id: string): void;
}

class UserReader implements IUserReader {
  // Only implements methods needed for reading
  findById(id: string): User | null {
    return this.repository.findById(id);
  }
  findByEmail(email: string): User | null {
    return this.repository.findByEmail(email);
  }
  findAll(): User[] {
    return this.repository.findAll();
  }
}
```

### Client-Specific Interfaces

- Interfaces designed for specific client needs
- No "god interfaces" with many methods

**Example:**

```typescript
// BAD: One interface for all clients
interface UserRepository {
  findById(id: string): User | null;
  findByEmail(email: string): User | null;
  findByUsername(username: string): User | null;
  findAll(): User[];
  findByRole(role: string): User[];
  findByStatus(status: string): User[];
  create(user: User): void;
  update(user: User): void;
  delete(id: string): void;
  changePassword(id: string, newHash: string): void;
  updateLastLogin(id: string): void;
}

// GOOD: Client-specific interfaces
interface IUserIdFinder {
  findById(id: string): User | null;
}

interface IUserEmailFinder {
  findByEmail(email: string): User | null;
}

interface IUserRepository extends IUserIdFinder, IUserEmailFinder {
  create(user: User): void;
  update(user: User): void;
  delete(id: string): void;
}

class AuthenticationService {
  constructor(private emailFinder: IUserEmailFinder) {
    // Only depends on email finding, not full repository
  }

  async authenticate(email: string, password: string): Promise<User | null> {
    const user = await this.emailFinder.findByEmail(email);
    if (user && this.passwordVerifier.verify(password, user.passwordHash)) {
      return user;
    }
    return null;
  }
}
```

## Dependency Inversion Principle (DIP)

### High-Level Modules Don't Depend on Low-Level Modules

- Both depend on abstractions (interfaces)
- Never depend on concrete implementations

**Example:**

```typescript
// BAD: High-level module depends on low-level module
class UserService {
  // Direct dependency on concrete implementation
  private repository = new SqlUserRepository();

  createUser(email: string, password: string): void {
    this.repository.save(email, password);
  }
}

// GOOD: High-level module depends on abstraction
interface IUserRepository {
  save(email: string, password: string): void;
}

class UserService {
  constructor(private repository: IUserRepository) {
    // Dependency on interface, not implementation
  }

  createUser(email: string, password: string): void {
    this.repository.save(email, password);
  }
}
```

### Both Depend on Abstractions

- High-level modules define abstractions
- Low-level modules implement abstractions
- Abstractions don't depend on details

**Example:**

```typescript
// Abstraction defined by high-level module
interface IEmailSender {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
}

// Low-level module implements abstraction
class SmtpEmailSender implements IEmailSender {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // SMTP-specific implementation
    await this.smtpClient.send({
      to,
      subject,
      body
    });
  }
}

class SesModuleEmailSender implements IEmailSender {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // AWS SES-specific implementation
    await this.sesClient.sendEmail({
      Destination: { ToAddresses: [to] },
      Message: { Subject: { Data: subject }, Body: { Data: body } }
    });
  }
}

// High-level module uses abstraction
class UserRegistrationService {
  constructor(
    private userRepository: IUserRepository,
    private emailSender: IEmailSender
  ) {
    // Depends on abstractions only
  }

  async registerUser(email: string, password: string): Promise<void> {
    const user = await this.userRepository.create(email, password);
    await this.emailSender.sendEmail(
      email,
      'Welcome!',
      'Thank you for registering.'
    );
  }
}
```

### Abstraction Independence

- Interfaces don't depend on implementation details
- Details change without affecting abstractions

**Example:**

```typescript
// GOOD: Abstraction independent of details
interface ICacheProvider {
  get<T>(key: string): T | null;
  set<T>(key: string, value: T, ttl: number): void;
  delete(key: string): void;
}

class InMemoryCacheProvider implements ICacheProvider {
  private cache = new Map<string, any>();

  get<T>(key: string): T | null {
    return this.cache.get(key) || null;
  }

  set<T>(key: string, value: T, ttl: number): void {
    this.cache.set(key, value);
    setTimeout(() => this.cache.delete(key), ttl * 1000);
  }

  delete(key: string): void {
    this.cache.delete(key);
  }
}

class RedisCacheProvider implements ICacheProvider {
  async get<T>(key: string): Promise<T | null> {
    const value = await this.redisClient.get(key);
    return value ? JSON.parse(value) : null;
  }

  async set<T>(key: string, value: T, ttl: number): Promise<void> {
    await this.redisClient.setex(key, ttl, JSON.stringify(value));
  }

  async delete(key: string): Promise<void> {
    await this.redisClient.del(key);
  }
}

// Usage - abstraction independent of implementation
class ProductService {
  constructor(
    private repository: IProductRepository,
    private cache: ICacheProvider
  ) {}

  async getProduct(id: string): Promise<Product | null> {
    const cached = this.cache.get<Product>(`product:${id}`);
    if (cached) return cached;

    const product = await this.repository.findById(id);
    if (product) {
      this.cache.set(`product:${id}`, product, 3600); // 1 hour TTL
    }
    return product;
  }
}
```

## Enables Testability and Flexibility

### Testability

- Dependencies can be mocked easily
- Unit tests isolate behavior
- No external system dependencies in tests

**Example:**

```typescript
class UserServiceTest {
  @Test
  async createUser_saves_to_repository(): Promise<void> {
    // Arrange - create mock repository
    const mockRepository = {
      save: jest.fn().mockResolvedValue({ id: '123', email: 'test@example.com' })
    } as jest.Mocked<IUserRepository>;

    const mockEmailSender = {
      sendEmail: jest.fn().mockResolvedValue(undefined)
    } as jest.Mocked<IEmailSender>;

    const service = new UserService(mockRepository, mockEmailSender);

    // Act
    await service.registerUser('test@example.com', 'password123');

    // Assert - verify dependencies were called correctly
    expect(mockRepository.save).toHaveBeenCalledWith(
      'test@example.com',
      expect.any(String) // password hash
    );
    expect(mockEmailSender.sendEmail).toHaveBeenCalledWith(
      'test@example.com',
      'Welcome!',
      expect.any(String)
    );
  }
}
```

### Flexibility

- Easy to swap implementations
- Configuration changes without code changes
- Multiple implementations for different environments

**Example:**

```typescript
// Development environment
const developmentConfig = {
  repository: new InMemoryUserRepository(),
  emailSender: new ConsoleEmailSender() // Log emails to console
};

// Production environment
const productionConfig = {
  repository: new PostgreSqlUserRepository(process.env.DATABASE_URL),
  emailSender: new SesModuleEmailSender(process.env.AWS_REGION)
};

// Same service works with both
const userService = new UserService(
  config.repository,
  config.emailSender
);
```

## SOLID in Practice

### Example: Shopping Cart

```typescript
// SRP: Each class has single responsibility
class Cart {
  // Manages cart state and business rules
}

class CartValidator {
  // Validates cart operations
}

class CartCalculator {
  // Calculates totals and discounts
}

// ISP: Small, focused interfaces
interface ICartReader {
  getItems(): CartItem[];
  getTotal(): Money;
}

interface ICartWriter {
  addItem(item: CartItem): void;
  removeItem(itemId: string): void;
  clear(): void;
}

interface ICartValidator {
  canAddItem(item: CartItem): boolean;
}

interface ICartCalculator {
  calculateTotal(items: CartItem[]): Money;
  calculateDiscount(items: CartItem[], coupon: Coupon): Money;
}

// DIP: All depend on abstractions
class CartService {
  constructor(
    private reader: ICartReader,
    private writer: ICartWriter,
    private validator: ICartValidator,
    private calculator: ICartCalculator
  ) {}

  async addItem(item: CartItem): Promise<AddItemResult> {
    if (!this.validator.canAddItem(item)) {
      return AddItemResult.failure('Item cannot be added');
    }

    this.writer.addItem(item);
    const total = this.reader.getTotal();

    return AddItemResult.success(total);
  }
}
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - Architecture overview
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/patterns/PATTERNS.md](patterns/PATTERNS.md) - Design patterns
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
